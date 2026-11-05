# Data Integration — Merge Rules

**Status:** Implemented — preview `imports/domain/services/merge_preview.py`, apply `imports/application/apply_import.py`, bulk `bulk_decision_actions.py`  
**Applies to:** Universal Import Standard (ADR-016)  
**Related:** [MATCHING_RULES.md](MATCHING_RULES.md), [IMPORT_ARCHITECTURE.md](IMPORT_ARCHITECTURE.md), history: [IMPORT_WIZARD_MERGE_RULES.md](../../../archive/fair-crm/import/IMPORT_WIZARD_MERGE_RULES.md)

These rules govern how import data is merged into existing CRM records during **Preview → Decision → Apply**. They apply to all source types.

---

## Core principle

**Import never blindly overwrites CRM data.** Merge is additive and conservative by default. The user sees field-level diffs and confirms decisions before apply.

---

## Customer field merge

| Scenario | Rule |
|----------|------|
| CRM field **populated**, import **empty** | **Keep CRM** — import does not clear |
| CRM field **empty**, import **has value** | **Fill from import** |
| CRM field **populated**, import **different value** | **Conflict** (`Çakışıyor`) — CRM value is kept; there is no field-level overwrite |
| CRM field **populated**, import **same value** (normalized) | **No change** — show as “Aynı” in merge preview |

### Never overwrite by default

- Existing customer `display_name`, address, country, city, website, notes, etc. are **not** replaced by conflicting import values without explicit user decision.
- Empty incoming values **never** null out populated CRM fields.

---

## Email merge (additive)

| Rule | Detail |
|------|--------|
| Additive | New emails from import are **added** to the customer email set |
| Deduplication | Same email (normalized: lowercase, trimmed) is **not** added twice |
| Multi-email | Import may contain `;` or `,` separated lists; each token normalized and merged |
| Cross-field | Email appearing in `phone` or `website` column is not auto-promoted without mapping |

Uses the shared `normalize_email_field` pipeline.

---

## Phone and website merge

| Rule | Detail |
|------|--------|
| Customer has no phone / website | Import values are written to the customer communication rows |
| Customer already has phone / website | Import value is **not** added; existing rows stay |
| Empty import value | No effect |

---

## Contact merge (additive)

| Rule | Detail |
|------|--------|
| New contact | Create `Contact` when import row includes contact fields and user decision is `create_new` or update with new contact |
| Existing contact match | Within the same customer: email first, then phone / mobile phone, then first + last name (`find_existing_contact_for_import`); empty contact fields are filled, emails are merged additively |
| Additive emails on contact | Same dedupe rules as customer email |
| No silent contact delete | Import never removes existing contacts |

---

## Fair participation merge

| Scenario | Rule |
|----------|------|
| Customer exists, **no** participation for batch fair | **Create** `CustomerFairParticipation` with hall/stand/status from import |
| Participation exists | Update hall/stand/notes only per merge rules (empty import does not clear) |
| Excel **only company name** | **Customer unchanged**; only participation created/updated for batch fair |
| No customer match | `create_new` decision creates customer **and** participation |

Hall and stand exist **only** on `CustomerFairParticipation` (ADR-010, ADR-012).

---

## Company-name-only rows

When the mapped import row contains **only** `company_name` (all other canonical fields empty after normalization):

1. If customer matched → **do not modify** customer scalar fields.
2. Ensure participation for batch `fair_id`:
   - Create if missing
   - Skip participation field updates if import has no hall/stand
3. Decision is `participation_only`

This supports exhibitor lists that are name-only.

---

## Conflicting information

When normalized import and CRM values differ on the same populated field, the preview outcome is `conflict` (**Çakışıyor**) and apply keeps the CRM value. There is no per-field "use import value" control.

---

## Skip and invalid rows

| Status | Apply behavior |
|--------|------------------|
| `skip` | Row not applied |
| `invalid` | Row not applied; validation errors shown in final report |
| `manual_review` | Row not applied until decision changed |

---

## Bulk decisions

| Bulk action | Rows | Decision |
|-------------|------|----------|
| `create_all_new` | Preview `new` | `create_new` |
| `link_all_existing` | Preview `will_update` | `update_existing` |
| `update_all_duplicates` | Preview `duplicate` | `update_existing` |
| `skip_invalid` | Preview `invalid` | `skip` |

A row that already has a different decision is not changed.

---

## Apply transaction policy

| Policy | Default |
|--------|---------|
| Row errors | **Continue** — collect failures, apply remaining rows |
| Batch rollback | Not automatic; operator may re-run after fix |
| Idempotency | Re-apply same batch with same decisions must not duplicate records |

---

## Merge preview display (Turkish labels)

| Outcome | Label |
|---------|-------|
| Same value | Aynı |
| New value on empty CRM field | Eklenecek |
| Update existing | Güncellenecek |
| Conflict | Çakışıyor |
| Preserved CRM value | Korunacak |
| New entity | Yeni |
| Both empty | Boş |
| Row skipped | Atlanacak |

---
