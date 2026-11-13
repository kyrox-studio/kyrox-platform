# KYROX Fair CRM — Project Status

Living status for FAIR CRM. This file records **current implementation truth only**. Detailed delivery history belongs in [CHANGELOG.md](CHANGELOG.md); the previous long-form status snapshot is preserved at [../../archive/fair-crm/status/PROJECT_STATUS_2026-07-24.md](../../archive/fair-crm/status/PROJECT_STATUS_2026-07-24.md).

| Field | Value |
|-------|-------|
| Last verified | **2026-11-13** (`main` `e613f6b8`, migration head `0099_background_job_scope_and_cancel`); last recorded deploy `e613f6b8` (2026-11-13, production head `0099_background_job_scope_and_cancel`) |
| Active ecosystem milestone | **M4 — FAIR CRM v1** |
| Implementation repository | `kyrox-studio/kyrox-fair-crm` |
| Migration head in `main` | `0099_background_job_scope_and_cancel` (`job_scope` on every job table, cancel fields for Data Operations runs, backups and import jobs; [#392](https://github.com/kyrox-studio/kyrox-fair-crm/pull/392)) |
| Current work queue | [ROADMAP.md](ROADMAP.md) |
| Shared standards | [../../standards/README.md](../../standards/README.md) |

## Current capability state

ADR-038 (one System Fair across editions) and ADR-040 (TOBB controlled sync/review) are merged to `main` with PR #342; ADR-039 user-directed merge and Constitution §5 organization boundaries remain in force.

| Area | Status |
|------|--------|
| Tenant isolation / SaaS P0.1 | **Certified DONE (2026-08-26)** — TI-01 through TI-09 complete across API, repository, worker, export/download and Platform Super Admin boundaries |
| Identity / SaaS onboarding P0.2 | **Approved onboarding/credential slice DONE (2026-08-29)** — Core identity runtime, thin FAIR CRM bridge, public signup/activation/recovery, login integration, authenticated password change, Super Admin compatibility and production-shaped cross-repository lifecycle certification are complete |
| Organization suspension runtime / OL-07 | **Certified DONE (2026-09-07)** — lifecycle authority, queued/running cancellation, pre-handoff blocking, durable in-flight provider semantics and deterministic reactivation/resumption behavior are implemented/certified CRM Organizasyonlar ekranı (2026-11-11, [#391](https://github.com/kyrox-studio/kyrox-fair-crm/pull/391)): Durum sütunu, Askıya Al / Aktif Et (Core suspend / reactivate); kendi organizasyonda Askıya Al yok; silinen organizasyon geri gelmez (ADR-0006). |
| Organization closure orchestration / OL08-01 | **Certified DONE (2026-09-07)** — durable FAIR CRM-owned non-destructive closure execution, SYSTEM start/status/retry, live Core `SUSPENDED` precondition, idempotency/race controls, blocked/retry evidence and append-only local audit evidence are merged and cross-repository certified |
| Closure quiescence / OL08-02 | **Certified DONE (2026-09-08)** — closure execution is proven not to bypass OL-07 queued/running/provider/lifecycle guards; closure start parent/event FK ordering was fixed transactionally |
| Closure export planner / OL08-03A | **Certified DONE (2026-09-08)** — durable versioned organization/execution-scoped completeness plan, explicit v1 data-class registry, organization-scoped counts, record-ID fingerprints, hard secret-source exclusions, SYSTEM metadata API, idempotency/audit/tenant-isolation evidence |
| Closure credential disposition / OL08-C + C2 | **Current-model runtime covered (2026-09-09)** — OL08-04A durable disposition/evidence foundation plus exact-secret MailerSend invalidity verification; provider/operator evidence remains fail-closed until definitive proof and local reusable secret zeroization follows accepted evidence only |
| Closure package / artifact lifecycle / OL08-E1 + E2 | **Runtime accepted (2026-09-09)** — canonical package/inventory/integrity, strict managed-file cleanup/non-existence evidence, external-reference non-action and independent package expiry/purge are implemented |
| Product-data disposition / OL08-D | **Runtime accepted (2026-09-10)** — dependency-aware 19-class tenant product-data hard delete is implemented behind lifecycle/grace/export/package/credential/artifact gates |
| Terminal closure / OL08-07 | **Runtime accepted (2026-09-11)** — final same-suspension-episode Core tombstone is implemented; authoritative Core `deleted_at` is mirrored exactly as FAIR `closed_at`, with restart-safe reconciliation |
| Retained evidence expiry / OL08-F | **Runtime accepted (2026-09-11)** — Core audit evidence and FAIR closure evidence expire only after exact `T_terminal + 12 calendar months`, with Core-first fail-closed ordering, FK-safe FAIR purge and idempotent retry semantics |
| Backup ageing / restore interaction / OL10 | **Policy + runtime accepted** — FAIR full-DB backup ageing/pruning and fail-closed restore reconciliation remain independently governed from OL08-F retained evidence |
| Customers / fairs / participations | Implemented |
| Contacts / activities / todos | Implemented |
| Data integration / import engine | Implemented and actively hardened |
| Matching / preview / decision flows | Implemented; later migrations continue stabilization |
| Scraper / adapter workflows | Implemented |
| Operations / automation engine | Implemented; pause-vs-cancel behavior change remains planned |
| Bulk email / delivery flows | Implemented: MailerSend webhook (unsubscribe/spam), Contact/Customer e-posta izni, merkezi `EmailConsentPolicy`, ayrı mail worker servisi |
| Admin backup / restore | Implemented |
| Responsive shared UI / table system | Implemented |
| Permission-aware UI | Shared rule defined; full product-wide consistency audit remains active work |
| Quotation-related capabilities | Present in current implementation; documentation reconciliation required |
| Cost catalog | Current implementation includes cost-catalog schema and Core permission support; tenant isolation is certified, while broader documentation/API/UI completeness is still being reconciled |

## Current quality / documentation focus

Açık işler ve sıraları yalnız [ROADMAP.md](ROADMAP.md) içindedir. Delivered truth stays here; detailed history stays in [CHANGELOG.md](CHANGELOG.md).

## SaaS and lifecycle records

`organization` is the tenant/account boundary; no parallel Tenant model exists and CRM business semantics stay product-owned. Delivery records (PRs, heads, CI runs, migrations) live in the canonical trackers, not here:

- P0.1 tenant isolation: [P0_1_TENANT_ISOLATION_CERTIFICATION](../../archive/fair-crm/backlog/P0_1_TENANT_ISOLATION_CERTIFICATION.md)
- P0.2 identity/onboarding: [P0_2_IDENTITY_ONBOARDING_IMPLEMENTATION.md](../../ecosystem/P0_2_IDENTITY_ONBOARDING_IMPLEMENTATION.md)
- OL-07 suspension runtime: [P0_2_OL_07_IMPLEMENTATION.md](../../ecosystem/P0_2_OL_07_IMPLEMENTATION.md)
- OL-08 closure (migrations `0077`–`0083`) and OL08-F expiry: [P0_2_OL_08_IMPLEMENTATION.md](../../ecosystem/P0_2_OL_08_IMPLEMENTATION.md), [P0_2_OL_08_F_RUNTIME_ACCEPTANCE.md](../../ecosystem/P0_2_OL_08_F_RUNTIME_ACCEPTANCE.md)

The required-export closure path is runtime-covered through terminal tombstone and retained-evidence expiry. The optional `not_required` export disposition has no accepted policy authority and fails closed. No further OL08 runtime is sequenced; a new case needs a separate policy/product decision ([ROADMAP.md](ROADMAP.md)).

## Backlog / planned behavior

Tek iş listesi: [ROADMAP.md](ROADMAP.md). Ayrıntı belgeleri sıra belirlemez.

## Update protocol

When a meaningful FAIR CRM capability changes:

1. Update this file only if current capability truth changed.
2. Update [CHANGELOG.md](CHANGELOG.md) with delivered history.
3. Update [ROADMAP.md](ROADMAP.md) when active/future work changes.
4. Refresh the FAIR CRM summary in [../../ecosystem/STATUS.md](../../ecosystem/STATUS.md).
5. Keep exact test counts, transient CI details and commit SHAs out of permanent standards and roadmap documents.
