# Fair Stand `docs/items` arşivi

> **Arşiv — normatif değildir.** Bu klasör hiçbir canlı kuralı ezmez ([DOCUMENT_GOVERNANCE](../../../ecosystem/DOCUMENT_GOVERNANCE.md) "Archive policy").

## Kaynak

- Depo: `kyrox-fair-stand`, commit `1fabe73c` (2026-10-27).
- Kaynak yollar korunmuştur: buradaki `audit/x.md`, Stand'deki `docs/items/audit/x.md` dosyasıdır.
- [MANIFEST.tsv](MANIFEST.tsv): taşınan 423 dosyanın kaynak yolu ve sha256 değeri. Kopyalar kaynakla birebir aynıdır; içerik değiştirilmemiştir.
- Dosyaların içindeki göreli bağlantılar Stand deposuna göredir; burada çözülmeyebilir.

## İçerik ve taşınma nedeni

| Bölüm | Dosya | Neden arşivde |
|---|---|---|
| `audit/` | 313 md + 12 html/json/csv/tsv | 2026-09-16 tek seferlik Item audit dökümü (fa1b5be3, 67234e2c, fdc99d5e, e2383879). Kendi README'si "tarihsel snapshot; silinmez" diyordu: silinmedi, buraya taşındı. |
| `current-system/` | 94 md | Belgelerin hepsi kendini tarihî ilan ediyordu ("TARİHÎ ENVANTER. Güncel sistem değildir" / "Migration öncesi envanterdir"). |
| `ITEM_CONSOLIDATION_PLAN.md`, `ITEM_LIST.md` | 2 md | Plan/migration takip belgeleri; Fair Stand planları uygulama deposunda tutulmaz ([AGENTS](../../../AGENTS.md)). `ITEM_MIGRATION_RULES.md` taşınmadı: Stand `.cursor/rules/no-fabrication-prepare-gate.mdc` onu geçerli kural sayıyor. |
| `door_100_full_system_audit.md` | 1 md | `door_100` item'ı 2026-09-25 `e00544eb` ile `wall_door_100_350` oldu. |
| `inventory/ITEM_CAPABILITY_INVENTORY.md` | 1 md | 2026-09-15 Create Item formu taraması. Kodda artık olmayan map ve sabitleri sayıyordu (`*_ITEMS` kovaları, `WALL_BEHAVIOR`, `connectorType`, `normalizeCatalogDescriptor`). Ayrı taşındı: Stand `4d04f885` (2026-10-28); MANIFEST'in son satırı. |

Bu belgelerde geçen adların bir kısmı artık kodda yoktur (ör. `WALL_BEHAVIOR`, `src/rawBomDebug.js`, `COMPOSITE_ITEMS`). Güncel davranış için aşağıdaki canlı kaynaklar kullanılır.

## Güncel kaynaklar

- Item sözleşmesi: `kyrox-fair-stand/docs/items/contract/ITEM_CONTRACT.md`
- Item tanımları: `kyrox-fair-stand/docs/items/definitions/`
- Item/katalog mimarisi: [ITEM_CATALOG_ARCHITECTURE](../../../projects/fair-stand/ITEM_CATALOG_ARCHITECTURE.md)
