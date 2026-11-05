# GitHub Workflows

`kyrox-fair-crm/.github/workflows/`:

| Workflow | Trigger | Purpose |
|----------|---------|---------|
| `development-standard.yml` | PR, manual | Backend Quality Check, Feature Contract / Applicability, Frontend Tests / Build / UI Governance, System Behavior Map / Zero New Findings |
| `prod-path-e2e.yml` | PR touching `backend/**`, `scripts/**` or the workflow, manual | Prod-path auth/RBAC, lifecycle and identity gate ([CI_PROD_PATH_E2E.md](CI_PROD_PATH_E2E.md)) |
| `cleanup-non-main-branches.yml` | Manual, or a push to `main` that edits this file | Deletes every branch except `main` |

The repository is private on the free plan: no branch protection, so a green PR run is checked before a manual merge ([QUALITY_GATE_STANDARD.md](../QUALITY_GATE_STANDARD.md)).
