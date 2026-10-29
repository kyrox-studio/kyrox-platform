# Fair Stand — Yol Haritası

**Fair Stand'in tek iş listesi.** Açık, ertelenmiş veya karar bekleyen her iş burada bir satırdır. Ayrıntı belgeleri tasarımı ve kararı tutar; kendi açık iş listelerini tutmaz. Bugünkü durum: [PROJECT_STATUS.md](PROJECT_STATUS.md). Satırların durumu 2026-10-28'de koda göre doğrulandı (Fair Stand `main` `b3f8cadd`).

Kurallar:

- Yeni iş önce buraya satır olarak eklenir. Biten iş buradan silinir ve [PROJECT_STATUS.md](PROJECT_STATUS.md)'ye yazılır.
- Audit bulguları (F-*) yalnız Fair Stand deposundaki [`audit/FINDINGS.md`](https://github.com/kyrox-studio/kyrox-fair-stand/blob/main/audit/FINDINGS.md) içindedir; burada kopyalanmaz.
- FAZ 4 planı Fair Stand deposundaki [`ROADMAP.md`](https://github.com/kyrox-studio/kyrox-fair-stand/blob/main/ROADMAP.md) ve [`ROADMAP_PHASE_4.md`](https://github.com/kyrox-studio/kyrox-fair-stand/blob/main/ROADMAP_PHASE_4.md) içindedir; `test/roadmapProductionSourceOfTruth.test.js` onları okur.
- Item / zarf / davranış kararlarının gerekçesi ve yasakları: [`docs/refactor/PENDING_ITEM_DECISIONS.md`](https://github.com/kyrox-studio/kyrox-fair-stand/blob/main/docs/refactor/PENDING_ITEM_DECISIONS.md) (aşağıda "PENDING").

## 1. Kullanıcı kararı bekleyen

| İş | Bekleyen karar | Ayrıntı |
|---|---|---|
| T birleşiminde gövde köşe paneli | Bugün T'de gövde duvarı köşe paneli almaz (`relationshipBom.js` `payCorner: false`), arka yüz de böyle. Kullanıcının T görseli gövdenin kol tarafını da köşe gösteriyor; değişirse ön yüz BOM'u da değişir | PENDING B.13 |
| Stand `frame_width_cm` kaldırma | Kolon sahnede okunmuyor (kesit = reçete profil Item'ı) ama API ve CRM Temel Ayarlar alanı duruyor (`routes.py`, `admin_settings.py`). Kaldırmak migration + CRM işi; CRM'e dokunmadan önce sorulur | — |
| `mount_height_cm` ↔ `default_z_cm` | İki kolon; üretici/tüketici zinciri netleşmeden biri seçilmez | PENDING B.3 |
| Dikme sahne kesiti | `upright_*` sahnede 8 × 8, profiller 10 × 10; sahne dikmeyi profil kesitiyle çiziyor. Dikme kayıtları profille eşitlenecek mi | — |
| Görsel seçimi iki yol | Özellikler panelinde arşivden seçme ve yeni yükleme nasıl sunulacak | — |
| Hazır stand tasarımları | İlk adım `autoFurniture.js` (sabit mobilya takımı); birden çok hazır tasarım arasından seçim ve sihirbaz bağlantısı açık | — |
| Paket ve Ahşap stand sistemleri | Kullanıcı kararı 2026-10-29: Maxima / Paket / Ahşap üç stand sistemi; sihirbazın ilk adımı sistem seçimi. Paket stand = shell scheme sistemi (hazır tasarım değil): duvarlar 1 m, hepsi uzun tip panel. Hazır stand tasarımları ayrı iş, sonra yapılacak (yukarıdaki satır). Açık: Paket duvar yüksekliği, profil/bağlantı ve BOM; Ahşap duvar yapısı; proje içinde sistem değişebilir mi | — |
| Sihirbaz ilk adımı yalnız Maxima aktifken | Paket/Ahşap "Yakında" pasif kart mı, yoksa tek aktif sistemde adım atlanır mı | — |
| Proje–organizasyon kuralları | Proje `organization_id` ile çalışıyor; paylaşım ve varsayılan org ayrı karar | PENDING E.1 |

## 2. Editör ve kullanım

| İş | Durum | Ayrıntı |
|---|---|---|
| Varyant aileleri admin'den | Plan; kod yok. Aile tanımı hâlâ `src/catalogWidthFamily.js` içinde; backend tablosu, `/admin/variant-families` ve CRM ekranı yok | [VARYANT_AILELERI_ADMIN_PLANI.md](VARYANT_AILELERI_ADMIN_PLANI.md) |
| Görsel arşivinde çift tık seçili yüzeye uygular | Yok (`#asset-library` karolarında `dblclick` yok) | — |
| Grup aç / kapat (Faz 3b) | Karar 2026-10-18; kod yok | [GROUP_LOCK_ROADMAP.md](GROUP_LOCK_ROADMAP.md) |
| "Raf ev sahibi duvar" (grup Faz 6) | Plan | [GROUP_LOCK_ROADMAP.md](GROUP_LOCK_ROADMAP.md) |
| Grup dış sınır ölçüleri | Açık | [KULLANILABILIRLIK_YOL_HARITASI.md](KULLANILABILIRLIK_YOL_HARITASI.md) |
| Stand sistemi altyapısı (Maxima / Paket / Ahşap) | Plan 2026-10-29; kod yok. Sistem tablosu (aktif bilgisiyle; şimdilik yalnız Maxima), Item–sistem eşlemesi (eşleme yoksa Item bütün sistemlerde: zemin, ekstra ürünler), admin Item formunda sistem seçimi, proje payload'ında sistem (eski projeler Maxima), sihirbazın ilk adımı sistem seçimi, katalog seçili sisteme göre süzülür. Not: bugünkü `standType` yerleşimdir (Sırt Duvar / U / L / Ada), sistem değildir | — |
| Müşterisiz Yeni Proje | Plan 2026-10-29; kod yok. CRM Stand Projeleri → Yeni Proje müşteri sormadan sahneyi açar; proje "Geçici bağ" (`UNASSIGNED_PROJECT_CUSTOMER_ID`) ile kaydedilir, müşteri mevcut atama penceresinden sonra bağlanır. Bugün müşteri üç yerde zorunlu: CRM müşteri seçme penceresi (`StandProjectsPage.tsx`), backend `create_project` (`customerId is required`), editör içe aktarma | — |
| Zemine çizerek duvar | Plan 2026-10-29; kod yok. Zeminde çizgi çizilir (10 cm ve 0°/90° yapışma, uzunluk etiketi), çizgi `composeStraightWall` ile serbest (`wallId: 'free'`) panellere bölünür (depo duvarları gibi). Açık: çizilen duvarların köşe/T birleşimlerinin BOM'a yansıması | — |
| Kurulu standın duvar yüksekliğini değiştirmek | Yükseklik yalnız "Sahneyi Oluştur"da seçilir; duvar/kapı/vitrin/kısa duvarların `resolveWallItemKeyForHeight` ile yerinde çevrilmesi, bağlı aksesuar/renk/görselin korunması yok | PENDING B.16 |
| Kurulu standın genişlik/derinliğini değiştirmek | Bugün yeni sahne kurulur; yerinde yeniden boyutlandırma yok | [KULLANILABILIRLIK_YOL_HARITASI.md](KULLANILABILIRLIK_YOL_HARITASI.md) |
| Bölgesel zemini sahneye bırakarak ekleme ve sürükleyerek taşıma | Bugün tek dikdörtgen `stand.floorArea`, X/Y girişi | [KULLANILABILIRLIK_YOL_HARITASI.md](KULLANILABILIRLIK_YOL_HARITASI.md) |
| Bütün yapı düzenleme | Duvar / depo / alınlık üzerinden çalışma; genişlik değişince panel, dikme ve profili sistemin çözmesi (Item, birleşim ve BOM kurallarından); uzatma/kısaltma/silmenin bağlı öğelere etkisinin önceden gösterilmesi | [KULLANILABILIRLIK_YOL_HARITASI.md](KULLANILABILIRLIK_YOL_HARITASI.md) |
| Teknik kartların basit modda görünürlüğü | Dikme, profil, kısa panel, kutu blok; görünürlük BOM kaydını kaldırmaz | [KULLANILABILIRLIK_YOL_HARITASI.md](KULLANILABILIRLIK_YOL_HARITASI.md) |
| Basit modda gizlenen işlemler | Klavye kısayolları ve sağ tık dahil bütün giriş yolları birlikte; desteklenmeyen işler hazır gösterilmez | [KULLANILABILIRLIK_YOL_HARITASI.md](KULLANILABILIRLIK_YOL_HARITASI.md) |
| Panel yüzeyi ile bütün yapının seçimini ayırmak | Kamera, sürükleme ve dokunmatik ile birlikte | [KULLANILABILIRLIK_YOL_HARITASI.md](KULLANILABILIRLIK_YOL_HARITASI.md) |
| Renk ve görsel alanının ilk görünümü | Palet, görsel yükleme ve Doldur / Sığdır / Kaldır yeterli olsun; HEX dahil son hiyerarşi (RGB/CMYK panelde gizli) | [KULLANILABILIRLIK_YOL_HARITASI.md](KULLANILABILIRLIK_YOL_HARITASI.md) |
| Sol menüyü Stand / Ürün ekle / Tasarım etrafında sadeleştirmek | Kullanıcı erteledi (2026-10-23); önceki öneri dört sekmeydi (Stand / Duvar & Panel / Mobilya / Teklif) | [KULLANILABILIRLIK_YOL_HARITASI.md](KULLANILABILIRLIK_YOL_HARITASI.md) |
| "Render Al" → "Görsel oluştur" | Düğme hâlâ "Render Al" | [KULLANILABILIRLIK_YOL_HARITASI.md](KULLANILABILIRLIK_YOL_HARITASI.md) |
| İşe uygun kısa ipuçları | Tam rehber erişilebilir kalır | [KULLANILABILIRLIK_YOL_HARITASI.md](KULLANILABILIRLIK_YOL_HARITASI.md) |
| Editör ışığında beyaz yüzey parlaması | Açık; 2026-10-26 ışık denemeleri kullanıcı kararıyla geri alındı | [SECIM_OZELLIK_PANELI_YOL_HARITASI.md](SECIM_OZELLIK_PANELI_YOL_HARITASI.md) |
| Kenar / yüzey hizalaması ve hassas taşıma | Uygulanmadı | [SECIM_OZELLIK_PANELI_YOL_HARITASI.md](SECIM_OZELLIK_PANELI_YOL_HARITASI.md) |
| Straforun zorunlu destek yüzeyine bağlanması | Uygulanmadı | [SECIM_OZELLIK_PANELI_YOL_HARITASI.md](SECIM_OZELLIK_PANELI_YOL_HARITASI.md) |
| Döndürme kartının gerçek hesap ve fiziksel telefonda kabulü | 15° adım ve kart tarayıcı testleriyle doğrulandı; fiziksel cihaz kabulü yapılmadı | [SECIM_OZELLIK_PANELI_YOL_HARITASI.md](SECIM_OZELLIK_PANELI_YOL_HARITASI.md) |
| Katalog kartı açıklaması | DB'de Item açıklama alanı + admin ekranı + kart ipucu; kodda Item'a özel metin yasak | — |
| Logo ve yazı aracı | Duvara metin (yazı tipi, renk, boyut), yerleşen yazılar listesi; kodda yok | — |
| Sunum modu | Salon ortamı, dönen kamera, hazır açılar, müşteriye PDF; kodda yok | — |
| Müşteri onay akışı | Canlı paylaşım linkinde "Onayla / Not bırak"; kodda yok | — |
| Sahnesiz ürünler | Sarf, hizmet, montaj için listeye ekleme; kodda yok | — |
| "Projeyi kontrol et" | Çakışma, açık kenar, eksik dikme, stand dışı modül; tıklayınca sahnede gösterir; kodda yok | — |
| Rehbere kısa video / GIF | — | — |
| Özellikler paneli görsel düzeni | Konum/boyut (masaüstü sağ, mobil alt), başlık ve bölüm sırası, boş durum, kilit açıklaması, ortak bileşenler, Basit/Gelişmiş hiyerarşisi, örtüşme kontrolü (masaüstü / dar ekran / tablet) | [SECIM_OZELLIK_PANELI_YOL_HARITASI.md](SECIM_OZELLIK_PANELI_YOL_HARITASI.md) |
| Özellikler paneli erişilebilirliği | Klavye odağı, ekran okuyucu etiketleri, kaydırma, dokunma hedefleri, açık/kapalı davranış | [SECIM_OZELLIK_PANELI_YOL_HARITASI.md](SECIM_OZELLIK_PANELI_YOL_HARITASI.md) |
| Özellikler paneli doğrulaması | Büyük seçimde ve sürüklemede gereksiz yeniden kurulum olmaması; renk/görsel/ölçü/silme/grup işlemlerinin undo-redo, otomatik kayıt, revizyon ve kayıt-açma ile eşliği | [SECIM_OZELLIK_PANELI_YOL_HARITASI.md](SECIM_OZELLIK_PANELI_YOL_HARITASI.md) |

## 3. Item, BOM ve DB

| İş | Durum | Ayrıntı |
|---|---|---|
| Tavan / şerit iskeletini sökmek (6 adım) | Sıradaki adım 1: her Item'da eksiksiz `scene_dimensions` + yüzey/BOM. `fair_stand_item_strip_occupancy` ve `strip_count` hâlâ kodda. Adımlar: B.7 yükseklik SKU'ları → strip slot migration → raf host anchor → occupancy + CRM (B.4–B.5) → `strip_count`/`strip_height_m` (C.1) → tavan fallback (C.2) | PENDING § A |
| `eyeCount` kaldırılacak | Hâlâ `admin_items.py`, `item_mapper.py`, `catalogPreviewRenderer.js` içinde; yerine gerçek cam raf yerleşimi | PENDING B.1 |
| Z clamp politikası DB'ye | `modulePlacement.js` `TODO(DB)`: `z_frame` / `z_max` tip davranışına | PENDING B.3b |
| Parent BOM ↔ leaf ölçü uyarısı (admin) | Karar var, kod yok; auto-fix yok | PENDING B.7 (ikinci) |
| Item snap kalanı | Tip/kural CRUD ve motor kısmen canlı | PENDING B.9 |
| `is_render` child çizimi | Karar var; reçete child mesh tam değil | PENDING B.10 |
| Color / Image / Lighting / Delete mekanizmaları | Yapılmadı; config şeması yok | PENDING D.1 |
| Item-first kalanları | Wall sanal hat kapanışı + E2E; self-parent / cycle admin ve seed tutarlılığı; rotation alanlarının tek kaynağı (`ROTATION.md`); yeni leaf için rotate kodu gerekmemesi | [`ITEM_FIRST_ROADMAP.md`](https://github.com/kyrox-studio/kyrox-fair-stand/blob/main/docs/refactor/ITEM_FIRST_ROADMAP.md) |
| Admin Item formu ↔ DB | Serbest metin alanlarının açılır listeye çevrilmesi, ghost alanlar, ölü kolonlar | [ADMIN_FORM_DB_AUDIT.md](ADMIN_FORM_DB_AUDIT.md) |
| `normalizeModuleItemState` dikme dalı | İçe aktarılan eski projelerde dikme sahne ölçüsü (2026-10-23 ertelendi) | — |

## 4. Veri bekleyen

| İş | Bekleyen |
|---|---|
| `shelf_leg` ölçüsü | DB'de ölçüsüz; ayak yalnız BOM'da. Ölçü girilirse çizim ayrı iş |
| `floor_showcase_100` ve `brochure_stand` fiyatı | `fair_stand_cost_items` fiyatı kullanıcıdan |
| `profile_346_5` maliyet kaydı | Admin ekranından girilecek |

## 5. Fazlar

| Faz | Durum | Ayrıntı |
|---|---|---|
| FAZ 4 — modül reçetesi + parametrik / yapısal editör | Aktif | [`ROADMAP_PHASE_4.md`](https://github.com/kyrox-studio/kyrox-fair-stand/blob/main/ROADMAP_PHASE_4.md) |
| FAZ 5 — Final BOM / Excel doğrulama | Planlandı | [ROADMAP_PHASE_5_6.md](ROADMAP_PHASE_5_6.md) |
| FAZ 6 — maliyet / Fair CRM (canlı fiyat ve teklif özeti dahil) | Planlandı | [ROADMAP_PHASE_5_6.md](ROADMAP_PHASE_5_6.md) |

## 6. Teknik borç

| İş | Durum | Ayrıntı |
|---|---|---|
| Audit bulguları (F-*) | Açık/kapalı durum yalnız orada | [`audit/FINDINGS.md`](https://github.com/kyrox-studio/kyrox-fair-stand/blob/main/audit/FINDINGS.md) |
| Işıklı Strafor seçim mesajı | `main.js` ışık rengi değişiminde mesajı `selectionFeedback.js` formatter'ı yerine elle yazıyor | — |
| `main.js` ve `scene3d.js` parçalanması | 2026-10-23 ertelendi | — |
| Yalnız testlerde kullanılan export'lar (~35) | Gözden geçirilecek | — |
| Performans P4 ve kalan ölçümler | 0 / 20 görselli proje açılışı, `rebuildWall` aynı kare birleştirme, 30 modülde `buildWall` | [PERFORMANCE_ROADMAP.md](PERFORMANCE_ROADMAP.md) |

## 7. Gelecek (faza yerleşmemiş)

| İş | Ayrıntı |
|---|---|
| PDF teklif, montaj sırası, depo/paketleme, stok, proje onay/paylaşım | [PRODUCT_FUTURE.md](PRODUCT_FUTURE.md) |
| Render kalitesi: HDRI, AO, PBR, post-processing | [RENDER_FUTURE_BACKLOG.md](RENDER_FUTURE_BACKLOG.md) |
| Tenant'a özel kataloglar | — |
| Cost-catalog slug birleşimi | Fair CRM cost-catalog mevcut fazının dışında |
