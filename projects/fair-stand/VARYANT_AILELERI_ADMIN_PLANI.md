# Varyant Aileleri — Admin'den Yönetim Planı

> Tarih: 2026-10-25 · Durum: **plan, kod yok**. Sırası: [ROADMAP.md](ROADMAP.md).
> Önceki iş: `0086_variant_family_cards` (Vitrin, Raf, TV, Video Wall, Uzun Saksı tek kart; varyant Özellikler panelinde).

## Amaç

Varyant ailelerini (panel başlığı, katalog kartı adı, düğme etiketi, sıra, eş aile) koddan çıkarıp DB'de tutmak ve CRM admin ekranından yönetmek. Yeni bir aile ya da yeni üye eklemek için kod değişikliği gerekmesin.

## Envanter (2026-10-25)

### DB

- `fair_stand_items.variant` (`String(64)`, boş olabilir): Item'ın ailesi. Değerler: `wall-panel`, `wall-separator`, `wall-separator-sarmasik`, `wall-short-1`, `wall-short-2`, `desk-banko`, `desk-banko-l`, `base`, `wall-showcase`, `shelf-row`, `tv`, `video-wall`, `long-planter`.
- `fair_stand_item_width_variants` (0072): raf kartı → genişlik ürünleri (`shelf_row_2` → `_100 / _150 / _200`). Ayrı mekanizma; raf genişliği takıldığı duvardan seçilir.
- Aile tanımı (başlık, kart adı, etiket, sıra) için tablo **yok**.

### Fair Stand kodu

- `src/catalogWidthFamily.js` `VARIANT_FAMILIES`: her aile için
  - `title` — panel başlığı ("Ekran boyu")
  - `cardLabel` — katalog kartı adı ("TV")
  - `label` — düğme metni (`50 cm`, `2 Gözlü`, `Tekli Raf`, `42"`, `2×2`)
  - `order` — sıralama (genişlik, göz sayısı, sıra sayısı, hücre sayısı)
  - `option` — eş aile (Separatör → Sarmaşıklı)
- Aile adına sabit bağlı kod: `autoFascia.js` ve `items.js` `isWallShortFamilyDescriptor` (`wall-short-1/2`), `designState.js` eski kısa panel kaydı dönüşümü.
- Ailenin varsayılanı = katalogda görünen üyesi (kart). Bırakınca kart konur; varyant panelde seçilir (`describeModuleVariants`, `main.js` `changeModuleVariant`).

### Backend / bootstrap

- `/catalog/bootstrap`: `categories`, `items` (içinde `variant`), `previewKinds`, `itemTypes`, `rules`, `settings`, `units`. Aile tanımı yok.
- Admin uçları: `/admin/categories`, `/admin/units`, `/admin/previews` (ekle / düzenle / arşivle / geri al), `/admin/items`, `/admin/item-records`.
- `admin_items.py`: `variant` alanını okur/yazar, mevcut değerleri öneri listesi verir. `width_variants` admin'den düzenlenmez (yalnız klonlamada kopyalanır).

### CRM admin

- `FairStandItemsAdminPage.tsx`: Item formunda "Varyant" alanı (öneri listeli metin kutusu).
- `FairStandUnitsAdminPage.tsx` (~320 satır): küçük ekle / düzenle / arşivle ekranı — yeni ekran için şablon.

## Plan

### 1. DB — `fair_stand_variant_families` (migration 0087)

- Alanlar: `family_key` (PK), `title`, `card_label`, `label_rule` (`width_cm` / `eye_count` / `shelf_rows` / `video_wall_grid` / `name_suffix`), `option_family_key` + `option_label` (eş aile), `sort_index`, `is_active`.
- Migration bugünkü 13 aileyi koddaki değerlerle doldurur (`wall-separator-sarmasik` dahil).
- `fair_stand_items.variant` → `family_key` FK (boş serbest).
- Item / reçete / BOM / kayıtlı proje değişmez; tablo yalnız katalog ve arayüz bilgisidir.

### 2. Backend

- Model, repository, `/admin/variant-families` (listele / ekle / düzenle / arşivle / geri al) — kategori ve birim uçlarıyla aynı kalıp.
- **Yetki:** ayrı CRUD kuralı yazılmaz; uçlar Item admin izinlerini kullanır (kullanıcı kararı, 2026-10-25).
- `/catalog/bootstrap` cevabına `variantFamilies`.
- Item kaydında `variant` tanımlı ve etkin bir aile olmalı.

### 3. Fair Stand frontend

- `catalogBootstrap.js` aileleri yükler; `catalogWidthFamily.js` başlık, kart adı, etiket ve sırayı sabit tablo yerine kayıttan alır.
- Etiket ve sıra hazır kural türleriyle yapılır (yönetici seçer, kod yazmaz).
- Koda bağlı `wall-short-1/2` aileleri adıyla kalır; admin'de arşivlenemez olarak işaretlenir.

### 4. CRM admin

- Yeni "Varyant aileleri" ekranı: liste + ekle / düzenle / arşivle formu (başlık, kart adı, etiket kuralı, eş aile, sıra).
- Item formundaki "Varyant" alanı serbest metin yerine aile listesinden seçilir.
- Ailenin üyeleri ve hangisinin kart olduğu ekranda görünür; kartı değiştirmek mevcut "katalogda görünür" alanıyla yapılır.

### 5. Test ve belge

- Backend: migration (yükselt / yeniden çalıştır / geri al), API, bootstrap, doğrulama.
- Fair Stand birim testleri, CRM ekran testi.
- Hedefli E2E: admin'de tanımlanan aile bir Item'a atanınca sahnede varyant düğmeleri çıkar.
- README, PROJECT_STATUS (Alembic head 0087), `docs/refactor/CATALOG.md`.

### Dokunulmayacaklar

BOM ve reçeteler, Item anahtarları, kayıtlı projeler, `fair_stand_item_width_variants` (raf genişliği duvardan seçilmeye devam eder), sahnedeki itme / snap davranışı.

### Değişecek depolar

`kyrox-fair-stand` (backend + frontend), `kyrox-fair-crm` (admin ekranı).

## Açık kararlar (kodlamadan önce)

1. **Etiket kuralı:** öneri — hazır kural türlerinden seçim. Alternatif: her Item'a ayrı "varyant etiketi" alanı (daha esnek, her üyeye elle yazılır).
2. **Varsayılan kart:** öneri — ailenin katalogda görünen üyesi. Alternatif: tabloda `default_item_key`.
3. **Raf genişlik eşlemesi:** öneri — bu işe girmesin; ayrı iş.
