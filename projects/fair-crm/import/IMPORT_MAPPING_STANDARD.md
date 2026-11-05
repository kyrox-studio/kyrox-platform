# Universal Import Mapping Preview Standard

**Status:** Accepted; the UI part is superseded by the ADR-025 mapping grid  
**ADR:** [ADR-024](../decisions/DECISIONS.md), [ADR-025](../decisions/DECISIONS.md)  
**Related:** [IMPORT_ARCHITECTURE.md](IMPORT_ARCHITECTURE.md), [IMPORT_WIZARD_UX_FLOW.md](../../../archive/fair-crm/import/IMPORT_WIZARD_UX_FLOW.md)

---

## Purpose

Before column mapping is saved, users must answer: **"What does this column actually contain?"**

Header names alone are insufficient for:

- Headerless files (columns A/B/C)
- Incorrect or misleading headers
- Duplicate or ambiguous column names

This standard defines how **sample values** are produced server-side and displayed in the mapping UI.

---

## UI (ADR-025 grid)

The wizard (`ImportWizardPage.tsx`) shows the Excel rows as a table; every column header has a mapping dropdown (Kullanma / Firma Adı / …). `company_name` is required and a field cannot be assigned twice. The grid comes from `build_excel_grid_preview` and shows at most **50** data rows (`MAX_GRID_PREVIEW_ROWS`). Empty cells render as `—`.

The API still returns per-column `mapping_columns` samples (below); the wizard does not render them.

---

## Header Modes

| Mode | Column label | Sample source |
|------|--------------|---------------|
| `first_row_header` | Header text + letter | Rows after row 0 |
| `no_header` | Column A/B/C… | First data rows (including row 0) |
| `manual_header_row` | Header from selected row | Rows after selected header row |

---

## API Contract

### Upload response (`POST …/upload`)

Includes `mapping_columns` using the default suggested header mode:

```json
{
  "mapping_columns": [
    {
      "key": "A",
      "index": 0,
      "letter": "A",
      "header": "Firma",
      "samples": ["ABC Mobilya", "XYZ Plastik", null],
      "stats": {
        "total": 3,
        "empty": 1,
        "filled": 2,
        "first_value": "ABC Mobilya"
      }
    }
  ]
}
```

### Mapping preview (`GET …/{batch_id}/mapping-preview`)

Query parameters:

- `header_mode` — optional; `first_row_header` | `no_header` | `manual_header_row`
- `header_row_index` — required for `manual_header_row` (0-based)

Returns `columns` (same shape) and `grid`. The wizard calls it when the user changes header mode or the manual header row and reads `grid`.

---

## Performance Rules

1. `mapping_columns` samples use at most the first **10** data rows (`MAX_MAPPING_SAMPLE_ROWS`); the grid uses at most **50** (`MAX_GRID_PREVIEW_ROWS`).
2. The full Excel sheet is **not** sent to the UI.
3. Raw preview remains stored server-side in `batch.raw_preview_json` for analyze/apply.

---

## Source Adapter Compatibility

Excel is the only adapter with a mapping preview (`build_mapping_preview_columns`, `build_excel_grid_preview` in `column_mapper.py`). A new adapter must return the same shapes.

---

## Implementation References

| Layer | Location |
|-------|----------|
| Sample builder | `backend/app/modules/imports/application/column_mapper.py` |
| Preview API | `GET /api/v1/data-integration/imports/{batch_id}/mapping-preview` |
| Wizard UI | `frontend/src/pages/ImportWizardPage.tsx` |
