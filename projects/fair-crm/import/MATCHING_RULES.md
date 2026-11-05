# Data Integration — Matching Rules

**Status:** Implemented — `imports/domain/services/company_name_normalizer.py`, `company_name_matcher.py`, `duplicate_detector.py`, `imports/application/import_row_builder.py`  
**Applies to:** Smart Matching stage ([IMPORT_ARCHITECTURE.md](IMPORT_ARCHITECTURE.md))  
**Related:** [MERGE_RULES.md](MERGE_RULES.md), ADR-016

Matching determines whether an import row relates to an existing **Customer** and/or **CustomerFairParticipation** for the batch fair. Matching is **read-only** until the user confirms decisions.

---

## Matching levels

```text
Import Row (normalized)
    ↓
Level 1 — Customer Match
    ↓
Level 2 — Fair Participation Match (batch fair_id)
    ↓
Combined row status + confidence + suggested action
```

| Level | Question | Entity |
|-------|----------|--------|
| **Customer match** | Does this row refer to an existing customer in the org? | `Customer` |
| **Participation match** | Does this customer already participate in the batch fair? | `CustomerFairParticipation` |

A row may have:

- Customer match only (new participation for this fair)
- Customer + participation match (update candidate)
- No customer match (create new customer + participation)

Fair context is always batch-level `fair_id` (ADR-012). **`fair_name` is never used for matching.**

---

## Company name normalization

Applied to produce **match keys** (not necessarily display values).

| Step | Rule |
|------|------|
| Turkish folding | Table below, then Unicode NFKD with combining marks removed |
| Case | Lowercase |
| Punctuation | `.`, `-`, `/` become spaces (dotted abbreviations split into tokens); every other non-word character becomes a space |
| Collapse spaces | Multiple spaces → single space, trimmed |
| Legal suffix cleanup | Phrases in `LEGAL_SUFFIX_PHRASES` are removed repeatedly until nothing changes |

The comparison key uses the legal name when present, otherwise the display name (`company_name_comparison_key`, shared with admin duplicate detection).

**Display name** for new customers uses the original mapped value after trim; the stripped key is used only for matching.

---

## Turkish character normalization

For **match key** comparison only:

| Source | Folded |
|--------|--------|
| İ, I, ı, i | `i` |
| Ş, ş | `s` |
| Ğ, ğ | `g` |
| Ü, ü | `u` |
| Ö, ö | `o` |
| Ç, ç | `c` |

Original characters are preserved in stored display fields.

---

## Legal suffix cleanup

Remove or standardize common Turkish and international legal suffixes **for match keys**:

**Turkish examples:** A.Ş., A.S., LTD. ŞTİ., LTD. STI., SAN., TİC., TIC., VE TİC., ANONİM ŞİRKETİ, LİMİTED ŞİRKETİ

**International examples:** LLC, LTD, LIMITED, CO., COMPANY. Also stripped: sanayi/ticaret, dış ticaret, ithalat/ihracat, pazarlama, ve (`LEGAL_SUFFIX_PHRASES`).

---

## Customer match (code)

Customer matching uses **only the company name**. Email, phone and website are not matching signals today.

1. **Exact key:** the row's normalized company name equals a customer's comparison key (legal name, else display name) or its `normalized_name` → confidence `100`, reason `exact_normalized_match`.
2. **Token score:** otherwise candidates are customers whose key shares the first 3 characters and whose key length is close; each pair is scored 0–100 by token overlap (Jaccard / overlap ratio), sequence similarity and suffix/abbreviation rules. A brand/first-token mismatch and a short single "hub" token block the match. The best candidate wins.

| Confidence | Reason (`match_reason`) |
|---|---|
| `< 70` | no match |
| `70–84` | `weak_name_candidate` |
| `85–94` | `fuzzy_name_candidate` |
| `≥ 95` | `exact_normalized_match` |

## Row result (code)

| Case | `status` | `suggested_action` |
|---|---|---|
| Validation failed | `invalid` | `skip` |
| No customer match | `ready_to_create` | `create_customer_and_participation` |
| Weak or fuzzy match | `possible_duplicate` | `link_existing_customer_to_fair` |
| Exact match, participation exists for batch fair | `ready_to_update` | `update_participation` |
| Exact match, no participation for batch fair | `ready_to_update` | `link_existing_customer_to_fair` |
| Exact match, batch without fair | `possible_duplicate` | `link_existing_customer_to_fair` |

Participation match uses the same `customer_id` + batch `fair_id`; hall/stand are not identity signals.

## Duplicate rows within a batch

Before CRM matching, a row whose normalized company name already appeared earlier in the same batch is marked `invalid` with reason `batch_duplicate_company_name`; the first row keeps its status.

---

## Performance considerations

With 28k+ customers:

- Matching must use indexed fields (email, normalized name prefix, phone)
- Batch matching runs in background after mapping (analysis job)
- Results paginated in preview UI (server-side list per ADR-015)

---

## Tests

`backend/tests/modules/imports/test_company_name_matching.py` (legal-form variants, should-not-match pairs, Turkish `i`, confidence bands), `test_duplicate_detector.py` (exact index, prefix-bucket fuzzy, below-threshold) and `test_import_row_builder.py` (validation, batch duplicate).
