# Maintenance scripts

`kyrox-fair-crm/scripts/maintenance/` holds internal developer tooling. These are not product features; they are not exposed in the API or UI. Run from the repository root.

## CI governance tools

Run by `.github/workflows/development-standard.yml` (`refine_*` and `enrich_*` are imported by `compare_system_behavior_inventory.py`):

| Script | Purpose |
|---|---|
| `inventory_system_behavior.py` | Generate the static Fair CRM system-behavior inventory |
| `refine_system_behavior_inventory.py` | Refine the raw inventory with stronger static evidence |
| `enrich_backend_permission_evidence.py` | Enrich backend route permission evidence through dependency-provider imports |
| `compare_system_behavior_inventory.py` | Fail when refined system-behavior findings regress against the comparison base |
| `inventory_frontend_ui.py` | Scan `frontend/src` for shared vs local/non-standard UI usage |
| `compare_frontend_ui_inventory.py` | Fail when a frontend change introduces new UI governance violations |

Their tests are `test_*.py` in the same folder. Standard: [SYSTEM_BEHAVIOR_MAP_STANDARD.md](../../../standards/development/SYSTEM_BEHAVIOR_MAP_STANDARD.md).
