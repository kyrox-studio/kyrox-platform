# Data Integration — Import Architecture

**Module:** Data Integration (`data_integration`)  
**Frontend label:** Veri Entegrasyonu  
**Route:** `/data-integration`  
**Status:** Implemented — backend modules `imports` and `data_integration`; source adapters: [SOURCE_ADAPTER_FRAMEWORK.md](SOURCE_ADAPTER_FRAMEWORK.md).

This document defines the **Universal Import Standard** pipeline for KYROX Fair CRM. It extends the existing Import Engine ([IMPORT_ENGINE.md](../../../archive/fair-crm/import/IMPORT_ENGINE.md)) and Smart Import Wizard (Sprint 07) into a source-agnostic, preview-first integration platform.

---

## Design principles

1. **Import is not direct insert.** Every batch passes through analysis, mapping, matching, preview, and explicit user decisions before persistence.
2. **Preview-first.** Upload and file analysis never write CRM domain data.
3. **Fair context required.** Batch-level `fair_id` is mandatory (ADR-012). The fair may be an organization fair or a system fair. Hall/stand belong on `CustomerFairParticipation`. The batch `organization_id` stays the tenant boundary (Constitution §5).
4. **Background execution.** Apply runs as a background job with progress and a final report.
5. **English backend, Turkish frontend.** API/module/table names in English; user-facing labels in Turkish.

See also: [MERGE_RULES.md](MERGE_RULES.md), [MATCHING_RULES.md](MATCHING_RULES.md), [decisions/DECISIONS.md](../decisions/DECISIONS.md) ADR-016.

---

## End-to-end pipeline

```text
Batch Setup
    ↓
File Analysis
    ↓
Header Mode (Excel)
    ↓
Column Mapping
    ↓
Normalization
    ↓
Smart Matching
    ↓
Preview
    ↓
Decision (per row / bulk)
    ↓
Background Import (Apply)
    ↓
Final Report
```

### Stage summary

| Stage | Purpose | CRM writes? |
|-------|---------|-------------|
| **Batch Setup** | User selects source type, fair, file(s), and batch metadata | No |
| **File Analysis** | Parse file structure, detect sheets, infer row count, sample values | No |
| **Header Mode** | Excel-only: declare how column headers are determined | No |
| **Column Mapping** | Map source columns → canonical CRM fields | No |
| **Normalization** | Canonical field values, Turkish/legal suffix cleanup, email/phone normalization | No |
| **Smart Matching** | Match rows to existing customers and fair participations; confidence scores | No |
| **Preview** | Row-level status, merge diff, validation errors, suggested actions | No |
| **Decision** | User confirms create / update / skip / manual review per row or bulk | No (decisions stored on batch rows) |
| **Background Import** | Apply approved rows in a job queue; idempotent, resumable | **Yes** |
| **Final Report** | Summary counts, failures, skipped, links to batch detail | Read-only |

---

## Batch Setup

User creates an **Import Batch** with:

| Field | Required | Notes |
|-------|----------|-------|
| `fair_id` | Yes | Selected fair for entire batch (ADR-012) |
| `source_type` | Yes | `ImportSourceType`: `excel`, `csv`, `api`, `pdf`, `scraper`, `database`, `manual`, `other` (ADR-016 priority) |
| `file_name` + `stored_file_content` | Yes (file sources) | Stored for replay and audit |
| `selected_sheet_name`, `header_mode`, `header_row_index` | Excel | Sheet and header choice |
| `column_mapping_json` | After mapping | Source column → CRM field mapping of this batch; there is no saved/reusable mapping template |
| `notes` | No | Operator context |

**Entry points:**

- **Veri Entegrasyonu → Import İşleri → Yeni Import**
- **Fuar Detayı → Katılımcıları İçe Aktar** (fair pre-filled; legacy `/imports` path migrates to `/data-integration` in implementation sprint)

---

## File Analysis

Source adapters extract **raw rows** without CRM semantics.

| Source | Adapter responsibility | In code |
|--------|------------------------|---------|
| Excel | Sheet selection, row/column bounds, typed cell reads | `ExcelSourceAdapter` |
| Web Scraper | Normalized scraper output → tabular rows | Scraper handoff into the batch |
| CSV, API, PDF, database | Source-specific extract | Only `ImportSourceType` values; no adapter |

Output: **raw row matrix** + metadata (sheet name, row numbers, sample values per column).

---

## Header Mode (Excel — mandatory)

On batch setup, user must choose **Excel Header Mode**:

| Mode | Turkish UI label | Behavior |
|------|------------------|----------|
| `first_row_header` | İlk satır başlık | Row 1 = column names; data from row 2 |
| `no_header` | Başlık yok | Columns labeled **A, B, C, D, …**; mapping uses sample values |
| `manual_header_row` | Başlık satırını ben seçeceğim | User picks header row index; rows above ignored |

### No-header mapping UX

When **Başlık yok** is selected:

- Mapping screen shows columns as **A, B, C, D, …**
- The mapping grid shows up to 50 data rows under the column headers ([IMPORT_MAPPING_STANDARD.md](IMPORT_MAPPING_STANDARD.md))
- User manually maps A/B/C… → canonical fields (`company_name`, `email`, `phone`, …)
- `company_name` mapping is **required** before analysis can proceed

When **Başlık satırını ben seçeceğim** is selected:

- User selects row number (1-based UI)
- That row becomes header labels for mapping (same as first-row mode after selection)

---

## Column Mapping

Maps source columns → **canonical import fields**.

| Rule | Detail |
|------|--------|
| Required field | `company_name` only (single required mapping) |
| Optional fields | email, phone, website, country, city, hall, stand, contact fields, … |
| No `fair_name` | Fair comes from batch `fair_id` only (ADR-012) |

Canonical field list aligns with `CANONICAL_FIELDS` in the Import Engine domain.

---

## Normalization

Applied after mapping, before matching:

- Trim whitespace; empty strings → `null`
- Company name normalization (see [MATCHING_RULES.md](MATCHING_RULES.md))
- Email: lowercase, multi-email split/merge rules
- Phone: digits only; Turkish numbers become `90` + 10 digits (`normalize_phone`); match keys only, display keeps the original
- Website: scheme normalization
- Turkish character folding for match keys (matching only; display preserves original where appropriate)

---

## Smart Matching

Two-level matching (ADR-012):

1. **Customer match** — existing `Customer` in organization
2. **Fair participation match** — existing `CustomerFairParticipation` for batch `fair_id`

Produces per row (`crm_import_rows`):

- `status` — `ImportRowStatus`: `pending`, `valid`, `invalid`, `possible_duplicate`, `ready_to_create`, `ready_to_update`, `applied`, `skipped`
- `match_customer_id`, `match_confidence`, `match_reason` (see [MATCHING_RULES.md](MATCHING_RULES.md))
- `participation_exists`, `match_participation_id`
- `suggested_action` — `ImportSuggestedAction`: `create_customer_and_participation`, `link_existing_customer_to_fair`, `update_participation`, `skip`
- `decision` — `ImportDecision`: `create_new`, `update_existing`, `participation_only`, `skip`, `manual_review`

---

## Preview

Preview screen shows:

- Row list with filters (all, new, update, duplicate, error, skip)
- **Merge diff viewer** (CRM vs import per field) — Sprint 07.1 baseline
- Bulk actions on filtered sets
- Validation errors inline
- No apply button without explicit user confirmation on decisions

---

## Decision

User sets row-level or bulk decisions:

| Decision | Meaning |
|----------|---------|
| `create_new` | New customer (+ participation if fair batch) |
| `update_existing` | Merge into matched customer/participation per merge rules |
| `participation_only` | Only the fair participation is written for the matched customer |
| `skip` | Row ignored on apply |
| `manual_review` | Held for operator; not applied until resolved |

Decisions are persisted on `ImportRow` before apply. Default: conservative (no overwrite of populated CRM fields).

---

## Background Import (Apply)

Apply runs as a **background job**:

- Job record (`ImportJobType`: `analyze`, `apply`, `bulk_decision`) with status `queued` → `running` → `completed` | `failed` | `partial` | `cancelled`
- Progress: processed / total rows
- Row-level apply results stored for audit
- Idempotent re-run rules (same batch + same decisions)
- Core audit events for applied changes (via KYROX Core integration)

User can navigate away; **Import Geçmişi** shows job status.

---

## Final Report

After job completion:

| Metric | Description |
|--------|-------------|
| Created customers | Count |
| Updated customers | Count |
| Participations created/updated | Count |
| Contacts added | Count |
| Skipped | Count |
| Failed | Count with error detail |
| Manual review remaining | Count |

Report is downloadable and linked from batch detail. Failures do not roll back successful rows unless batch policy specifies transactional apply (default: **continue on row error**, collect failures).

---

## Frontend navigation (Veri Entegrasyonu)

Sidebar (`components/layout/sidebarTreeItems.tsx`):

| Menu item | Route |
|-----------|-------|
| Import İşleri | `/data-integration/imports` (resume: `/data-integration/imports/continue/:batchId`) |
| Excel Import | `/data-integration/imports/new` (fair pre-filled: `/data-integration/imports/fair/:fairId`) |
| İşler | `/data-integration/jobs` |
| Raporlar | `/data-integration/reports` |
| Adapter Yönetimi | `/data-integration/adapters` |
| Run History | `/data-integration/run-history` (detail `/data-integration/runs/:runId`) |
| Scraper Test | `/data-integration/scraper-test` |

The legacy `/imports` and `/imports/fair/:fairId` frontend routes still open the same wizard.

## API

Import routes are mounted at `/api/v1/data-integration/imports`; the same router is also mounted at `/api/v1/imports` (hidden from OpenAPI) for existing consumers. Data-integration adapter/run routes live under `/api/v1/data-integration`.

---

## Background job standard

Import apply stays an organization job: `job_scope = organization` and a required batch `organization_id`. Status, progress, error aggregation, Import Geçmişi, and polling or SSE follow the [background job standard](../operations/BACKGROUND_JOB_STANDARD.md).

TOBB catalog sync and the system fair scraper are not import-apply jobs. They are `job_scope = system` with `organization_id` null. They do not write customers or participations.

## Shared participant dataset and tenant batches

A system scraper runs once. Its canonical handoff file is the shared dataset. Participant count is `scraper_run_history.total_rows`. There is no participant snapshot table in V1.

Each organization creates its own import batch from that handoff through the existing canonical import path. The Import Engine is not copied. A new scrape does not mutate older organization batches.

```text
SYSTEM DATASET = shared handoff
IMPORT BATCH = organization scoped
IMPORT ROW = organization scoped
```

The system run and its handoff must not be deleted while organization import flows still need that dataset. Querying the shared list in SQL without opening a batch is out of scope until a later decision.

---

## Permissions

Import routes use `fair_crm.imports.read`, `.create`, `.update`, `.delete`, `.execute`; scraper/adapter routes use `fair_crm.scraper.*` with the same actions. Scope rules: [PERMISSION_SCOPE_GOVERNANCE.md](../PERMISSION_SCOPE_GOVERNANCE.md).
