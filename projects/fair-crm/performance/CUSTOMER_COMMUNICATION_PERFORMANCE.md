# Customer Communication — Phase 2 Performance Review (TODO)

**Status:** Deferred (ROADMAP §4)  
**Date:** 2026-07-03  
**Related:** Migration `0020_customer_communications`, [decisions/DECISIONS.md](../decisions/DECISIONS.md) (ADR-031)

---

## Context

Customer phone, email, and website data now lives in child tables:

| Table | Purpose |
|-------|---------|
| `crm_customer_phones` | One row per phone number |
| `crm_customer_emails` | One row per email address |
| `crm_customer_websites` | One row per website |

The old scalar columns on `crm_customers` (`phone`, `email`, `website`) were dropped in migration `0021_drop_customer_communication_scalars`.

**Current indexes:** each child table has `customer_id` and `organization_id` indexes and `crm_customer_emails.email` is indexed (migration `0020`); `crm_customer_phones.phone` and `crm_customer_websites.website` are indexed by migration `0023`. **Decision (ADR-031):** no further communication-value indexes until a trigger below is met; index strategy follows real query patterns.

**Explicit non-goals for now:**

- No migration
- No new indexes
- No database schema changes
- No application code changes for performance

---

## Phase 2 Performance Review

Revisit when **any** trigger below is met:

| Trigger | Threshold |
|---------|-----------|
| Customer count | > 100,000 |
| Import batch size | > 50,000 rows |
| Duplicate analysis | Becomes slow in production or staging |
| Merge operation | Becomes slow in production or staging |

---

## Future review items

When triggered, run a structured performance review:

### 1. Measure before indexing

Run `EXPLAIN ANALYZE` on the heaviest paths:

- **Import lookup queries** — import matching is company-name only today ([MATCHING_RULES.md](../import/MATCHING_RULES.md)); this path matters only if phone/email/website matching is added
- **Duplicate grouping queries** — admin data-operation duplicate analysis (group by email, phone, domain, etc.)
- **Merge lookup queries** — loading and reconciling communication rows when merging winner/loser customers

Capture baseline latency, row counts, and sequential scan hotspots.

### 2. Index candidates (decide after EXPLAIN)

Consider indexes on **normalized** communication values (exact columns TBD after normalization rules stabilize):

| Candidate | Columns (illustrative) |
|-----------|-------------------------|
| Email lookup | `organization_id` + normalized email |
| Phone lookup | `organization_id` + normalized phone |
| Domain lookup | `organization_id` + normalized domain |

Additional options if profiling warrants:

- **Partial indexes** — e.g. `is_primary = true` only, or active-customer subsets
- **Composite indexes** aligned to actual `WHERE` / `GROUP BY` clauses from duplicate and import jobs

Do not add indexes speculatively; each index must map to a measured slow query.

### 3. Bulk operations

Review bulk import and upsert paths:

- Batch insert/update for communication child rows during large imports
- Avoid per-row round trips where merge or import applies thousands of communication changes
- Consider `COPY` / bulk upsert patterns for maintenance-scale backfills

---

## Checklist (when Phase 2 starts)

- [ ] Confirm trigger threshold met (document which one)
- [ ] `EXPLAIN ANALYZE` — import lookup queries
- [ ] `EXPLAIN ANALYZE` — duplicate grouping queries
- [ ] `EXPLAIN ANALYZE` — merge lookup queries
- [ ] Propose index DDL with measured benefit (new migration only after approval)
- [ ] Re-run duplicate/import/merge benchmarks on staging with production-like volume
- [ ] Update this document with findings and accepted index strategy

---

## References

- Child table migration: `backend/alembic/versions/0020_customer_communications.py`
- List API batch loading: `SqlAlchemyCustomerCommunicationRepository.load_list_summaries()`
- Duplicate analysis and merge: Data Operations `duplicate_customer_analysis` (`customers/application/duplicate_group_merge*.py`)
- Import matching: `import/MATCHING_RULES.md`
