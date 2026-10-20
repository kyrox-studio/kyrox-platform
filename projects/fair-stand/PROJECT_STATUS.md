# Fair Stand — Proje Durumu

| Alan | Değer |
|------|--------|
| Son doğrulama | **2026-10-20** |
| Implementation deposu | `kyrox-studio/kyrox-fair-stand` |
| Hosted backend | Fair Stand FastAPI (`:8002`) / PostgreSQL `fair_stand` |
| Item master | İlişkisel PostgreSQL; catalog bootstrap API |
| Category master | `fair_stand_categories` |
| Catalog | Yalnız Item projeksiyonu |

## Uygulama durumu

**IMPLEMENTATION STATUS:** LOCAL READY

**REMOTE / GITHUB STATUS:** bu görevde değerlendirilmedi / değiştirilmedi.

Lokal runtime acceptance PASS. `deployed` / `merged` / `production` iddiası yoktur.

## Mevcut yetenek durumu

| Alan | Durum |
|------|--------|
| Ürün sahipliği | ADR-0007 ile kanonik |
| Item / Category DB | PostgreSQL `fair_stand`; Fair Stand API |
| Configurator | Fair Stand runtime; Item registry API bootstrap |
| Proje kaydet / yükle | Sunucu PostgreSQL `fair_stand_projects` (payload `{stand, modules}`). IndexedDB yalnız önbellek. Item master değil |
| Tenant-scoped Item katalog | Kapsam dışı; katalog global |
| Static JS Item master | Kaldırıldı |
| Fallback / dual SoT | Yok |
| Migration | Fair Stand Alembic head `0071_separator_50_band_recipe` (canlı veritabanı da `0071`, 2026-10-19 deploy). `0068`–`0071` duvar yüksekliği bant reçeteleri: `fair_stand_item_components.per_band` ve `height_matched`; düz duvar + dikmeler (`0068`), kapı ve 2/3 gözlü vitrin (`0069`), 100'lük ayırıcılar (`0070`), 50'lik ayırıcılar ve 350 reçete düzeltmesi (`0071`). `0067` organizasyon grup şablonlarını (`fair_stand_group_templates`) ekler. `0065_metal_separator` katalogda görünmeyen `metal_separator` Item'ını ekler; `0066` bu Item'ı `metre_kare` birimiyle maliyetlendirilebilir yapar (dönüştürülmüş panel bloğu adet değil en × boy olarak faturalanır). `fair_stand_cost_items` organization Item fiyatını ve manuel kalemi aynı tabloda tutar. ITEM satırında `item_key` zorunlu, `name` ve `unit` boştur. MANUAL satırında `item_key` boştur, `name` ve `unit` zorunludur. `unit` → `fair_stand_units.unit_key`, ON UPDATE CASCADE, ON DELETE CASCADE. Aynı organization aynı Item’ı iki kez yazamaz. Manuel ad ve birim için ek unique yoktur. `fair_stand_items` fiyat kolonu taşımaz. `fair_stand_items.is_cost_enabled` NOT NULL, default false; mevcut satırlar false kalır. Bu bayrak fiyat, birim ve reçete miktarı değildir. `fair_stand_item_type` sınıflandırmadır. Sahne davranışı isteğe bağlı `fair_stand_item_type_scene_behavior` satırıdır; satır yoksa tip non-scene’dir. `production` behavior satırı taşımaz. `digital_print`, `mesh_fabric`, `lightbox_fabric` bu tipe bağlı gizli Item’lardır. `foam_logo` kaldırıldı. Strafor metrekare `illuminated-foam` sahne kaydına yazılır. `tulle_fabric` (görünen ad Tül, tip `tulle-fabric`) tek Item’dır: katalogda görünür, sahneye sürüklenir, BOM miktarı en × boy / 10000 m² olarak aynı `itemKey` ile yazılır ve maliyet kalemi bu `itemKey` ile eşleşir. Kalınlık item master `depth_cm` 0.4 değeridir. İlk yerleşim `default_z_cm` 350 değerini kullanır. `fair_stand_items.unit` → `fair_stand_units.unit_key` FK’sidir: ON UPDATE CASCADE, ON DELETE RESTRICT. Proje BOM’u katalogdaki `elektrik_panosu` satırını her listeye 1 adet yazar; bu satır modül reçetesi değildir. Aynı liste Dijital Baskı, Mesh ve Lightbox alanlarını kendi production kalemlerine, strafor alanını `illuminated-foam` kaydına bağlar. |

Mimari ayrıntı: [ITEM_CATALOG_ARCHITECTURE.md](ITEM_CATALOG_ARCHITECTURE.md).

## Sahne editörü: yalnız Kaydet, Kopyala, çift tık seçimi — 2026-10-20

`kyrox-fair-stand` PR #309 (açık; merge ve deploy yok, kullanıcı lokalde deniyor).

- **Otomatik kayıt kapalıyken yalnız Kaydet:** görsel ekleme/kaldırma/silme, sekme gizleme ve editörden çıkış kaydetmez; revizyon yalnız Kaydet ile ilerler. Aç / Sahneyi Oluştur / İçe Aktar / proje seçimi onay sorar, Dışarı Aktar son kaydı aktarır, "← Standlar" "Kaydetmeden çıkılsın mı?" diye sorar. CRM değişmedi.
- **Kopyala:** sağ tık menüsündeki Çoğalt Sağ / Sol kaldırıldı; seçim çubuğundaki Kopyala ve Ctrl/Cmd + C modülü hafızaya alır, her tıklama/dokunuşta katalogla aynı kuralla bir kopya bırakır (Esc / Kopyalamayı bitir; Ctrl/Cmd + V yeniden başlar). Grubu çoğalt aynı.
- **Seçim:** çift tık / çift dokunuş modülün tamamını seçer; Ctrl/Cmd + tık seçilenin çoğulu (modül seçiliyse modül ekle/çıkar, panel veya banko yüzü seçiliyse çoklu yüzey). **Shift + tık ile modül seçimi kaldırıldı.**
- **Tablet:** Çoklu panelde ikinci panele dokunmak Ctrl + tık gibi seçer.

## Yeni proje sihirbazı — 2026-10-20

`kyrox-fair-stand` (PR açık; merge kullanıcı testinden sonra). Yeni projede boş sahnede 3 adımlı kurulum (stand tipi, ölçü + duvar yüksekliği + depo, özet + Oluştur) mevcut kurulum alanlarını ve Sahneyi Oluştur akışını kullanır. İlk sahneden sonra 3 ipucu bir kez gösterilir. Veritabanı / migration yok.

## Tablet kullanımı — 2026-10-20

`kyrox-fair-stand` PR #308 (merge + deploy, 2026-10-20).

- Seçili öğenin üstünde seçim çubuğu: Döndür / Kopyala / Sil / Çoklu panel (mavi) veya Çoklu modül (turuncu) / ⋯.
- Dokunmatikte uzun basma sağ tık menüsünü açar; menüye Döndür eklendi.
- Sağ tık menüsü sade + **Gelişmiş** (Cam, Metal Separatör, Lightbox, Mesh).
- Çoklu panel: parmakla tarama (Ctrl + tık dikdörtgeni, komşu modül ve köşe dahil). Katalog kartına dokunup sahnede yere dokunarak ekleme.
- Üst banttaki ipucu yazıları kaldırıldı; görünüm küpü sağ üst köşede; bant küpün ve menü aç/kapa düğmesinin altına hiçbir boyutta girmez; iki parmakla yakınlaştırırken öğe sürüklenmez; konsolda ölçüm satırı varsayılan kapalı (`localStorage.fairStandTiming = '1'` ile açılır) (2026-10-20, ayrı PR).

## Duvar yüksekliği 250–500 cm — 2026-10-19

`kyrox-fair-stand` PR #307 (merge + deploy, 2026-10-19; migration `0068`–`0071`).

- Stand kurulurken duvar yüksekliği 250, 300, 350, 400, 450 veya 500 cm seçilir (varsayılan 350). Duvar ailesi (düz duvar, kapı, 2/3 gözlü vitrin, 50/100 ayırıcı ve sarmaşıklı, kısa duvar) seçilen yüksekliğe göre türetilir.
- Reçete DB'de 350 cm olarak kalır; bant sayısı B = yükseklik / 50: panel B, tekli aparat 2B − 1, başlangıç 2; dikme yüksekliğe göre eşlenir. Köşe / T / yan yana bağlantı BOM'u aynı kurala göre.
- Açık bir standın duvar yüksekliğini sonradan değiştirmek ertelendi: `PENDING_ITEM_DECISIONS.md` B.16.

## Sahne editörü: kurumsal grup şablonu — 2026-10-18

[GROUP_LOCK_ROADMAP.md](GROUP_LOCK_ROADMAP.md) Faz 5. `kyrox-fair-stand` PR #286.

**Kullanıcı tarafı**
- Gruplar kartı → **Şablon olarak kaydet**; kenar çubuğu **Şablonlarım** → Yerleştir / Adlandır / Sil.
- Şablon Item değildir; yerleştirince normal parçalar + yeni kilitsiz grup oluşur.
- **Korunan:** renk, cam panel, metal separatör, ölçü ve göreli diziliş.
- **Korunmayan:** görseller, Lightbox, Mesh (5a).

**Veri.** Tablo `fair_stand_group_templates` (migration `0067`); org-scoped; API proje yetkileriyle.

**SaaS etkisi.** Organization izolasyonu korunur. Item master ve katalog fiyatı etkilenmez; BOM parçalardan çıkar.

## Sahne editörü: grup döndürme — 2026-10-18

[GROUP_LOCK_ROADMAP.md](GROUP_LOCK_ROADMAP.md) Faz 4. `kyrox-fair-stand` PR #285.

**Kullanıcı tarafı**
- Gruptaki bir öğe seçiliyken **Shift+R** bütün grubu 45° döndürür.
- Parçalar birbirine göre yerini korur; art arda döndürmede grup kaymaz.
- Duvar, kapı ve panel de 45° açıya geçebilir.
- Döndürülmüş grup standa sığmazsa işlem yapılmaz. Kilitli grup dönmez.
- Taşıma, döndürme ve çoğaltma aynı sınır kuralını kullanır: öğenin dönmüş kaplama alanı stand içinde kalmalı. Önceden 45° döndürülmüş duvar paneli sürüklenince stand dışına taşabiliyordu; bu hata aynı PR'da düzeltildi.
- Sağ tık menüsünde ve Gruplar kartında döndürme düğmesi yoktur.

**Veri.** Yalnız üyelerin konum ve açı değerleri değişir; kayıt biçimi, şema ve migration değişmez.

**SaaS etkisi.** Organization izolasyonu, yetki, entitlement ve kullanım ölçümü değişmez. Item master, katalog ve BOM etkilenmez.

## Sahne editörü: grup — 2026-10-18

[GROUP_LOCK_ROADMAP.md](GROUP_LOCK_ROADMAP.md) Faz 3, öğeden bağımsız tanımla. `kyrox-fair-stand` PR #284.

**Kullanıcı tarafı**
- Öğeler çift tık + Ctrl/Cmd+tık ile seçilir (2026-10-20'ye kadar Shift+tık), sağ tık → **Grup oluştur…** → isim verilir.
- Her türlü öğe gruplanabilir: duvar, profil, lamba, logo, TV, kutu blok, mobilya.
- Bir öğe tek grupta olur; başka gruba eklemek sorularak yapılır.
- Grup her yöne birlikte taşınır (sürükleme, ok tuşu). Tek kısıt stand alanıdır.
- Kenara bağlı duvar parçaları gruplanınca serbest öğe olur.
- **Grup kilidi** bütün parçaları kilitler.
- **Grubu çoğalt** yeni, kilitsiz bir kopya grup üretir.
- **Grubu sil** onay ister.
- Kenar çubuğunda **Gruplar** kartı var.

**Veri.**
- Grup bilgisi her üye öğede `group` alanı olarak durur.
- Proje kaydı biçimi, şema ve migration değişmez.

**SaaS etkisi.**
- Organization izolasyonu, yetki, entitlement ve kullanım ölçümü değişmez.
- Grup bir Item değildir; Item master, katalog ve BOM etkilenmez.

**Bilinçli sonuç.** Kenardan ayrılan duvar parçaları duvar kapasitesi hesabına ve otomatik duvar lambası dizisine girmez.

## Eksik görselle proje açma ve müşterisiz içe aktarma — 2026-10-18

`kyrox-fair-stand` PR #283.

**Kullanıcı tarafı**
- **Önce:** Projenin tek bir görsel dosyası sunucuda eksik olduğunda proje hiç açılmıyordu ("Proje açılamadı").
- **Şimdi:**
  - Proje açılıyor; görseli bulunamayan yüzey kendi rengiyle gösteriliyor.
  - Durum satırında kaç görselin bulunamadığı yazıyor.
  - Görsel bağlantısı projede kalıyor; dosya geri gelirse görsel yeniden görünüyor.
- **İçe aktarma:** Paket müşteri bilgisi taşımıyorsa ve editör müşterisiz açıldıysa, ham API hatası yerine yol gösteren bir mesaj çıkıyor: "Müşteriler → müşteri → Yeni Proje". Her projenin bir müşteriye bağlı olması kuralı değişmedi.

**Veri.**
- Proje kaydı, şema, migration ve API değişmez.
- Kaydetme, sunucudaki görselleri silmez.

**SaaS etkisi.** Organization izolasyonu, yetki, entitlement ve kullanım ölçümü değişmez.

**Açık.** ZIP dışa aktarma, eksik görsel olduğunda hâlâ hata veriyor.

## Otomatik kayıt süresi — 2026-10-18

`kyrox-fair-stand` PR #282.

**Kullanıcı tarafı**
- Proje kartında Kaydet düğmelerinin altında **Otomatik kayıt** listesi var: Kapalı (Otomatik Kayıt Kapalı), 3, 5, 7 ve 10 dk. Varsayılan 3 dk.
- Önceki sabit 30 sn bekleme ve 60 sn üst sınır kalktı. Seçilen süre hem bekleme hem üst sınırdır.
- Kapalı seçilince süreli kayıt yapılmaz. Durum satırı kaydedilmemiş değişikliği gösterir; kayıt **Kaydet** ile elle yapılır.
- Kaydedilmemiş değişiklik iki durumda hemen kaydedilir: sekme gizlenince ve editörden çıkılınca. Bu, Kapalı seçiliyken de geçerlidir.
- Sayfa kapatılırken veya yenilenirken tarayıcı uyarı verir.

**Veri.**
- Süre seçimi yalnız tarayıcıda tutulur (`localStorage`, `fairStand.autosaveMinutes`).
- Proje kaydı, şema, migration ve API değişmez.

**SaaS etkisi.** Organization izolasyonu, yetki, entitlement ve kullanım ölçümü değişmez. Item master, katalog ve BOM etkilenmez.

## Sahne editörü: geri al / ileri al — 2026-10-18

[GROUP_LOCK_ROADMAP.md](GROUP_LOCK_ROADMAP.md) Faz 2. `kyrox-fair-stand` PR #281 merge edildi; CI (contract gate, test, pytest, build, Chromium E2E) yeşil.

**Kullanıcı tarafı**
- **Ctrl/Cmd+Z** geri al; **Ctrl/Cmd+Y** ve **Ctrl/Cmd+Shift+Z** ileri al. Üst bantta Geri al / İleri al düğmeleri.
- Kapsam: taşıma, döndürme, ekleme, silme, çoğaltma, renk, görsel, cam, separatör, kumaş, kilit ve zemin.
- Son 50 adım tutulur.
- Geçmiş kaydedilmez. Proje açılınca, yeni projede ve görsel silinince sıfırlanır; geçmiş revizyon görünümünde kapalıdır.

**Veri.** Proje kaydı, şema, migration ve proje sürümü değişmez; geçmiş yalnız tarayıcı belleğindedir.

**SaaS etkisi.** Organization izolasyonu, yetki, entitlement ve kullanım ölçümü değişmez. Item master, katalog ve BOM etkilenmez.

## Sahne editörü: öğe kilidi — 2026-10-17

[GROUP_LOCK_ROADMAP.md](GROUP_LOCK_ROADMAP.md) Faz 1. `kyrox-fair-stand` PR #278 merge edildi; CI (contract gate, test, pytest, build, Chromium E2E) yeşil. Sunucuya yayın bu kayıtta yoktur.

**Kullanıcı tarafı**
- Sağ tık menüsünde **Kilitle / Kilidi aç**.
- Çift tık + **Ctrl/Cmd+tık** ile toplu seçim (2026-10-20'ye kadar Shift+tık) ve sağ tıkla toplu kilitleme.
- Kilitli öğe sürüklenmez, ok tuşları ve Shift+R ile oynamaz, boyutlandırılamaz, silinemez.
- Zincir kaydırması (taşıma, katalogdan bırakma, Ekle/Çoğalt, toplu ekleme) kilitli öğeyi oynatacaksa işlem reddedilir.
- Görünüm işlemleri kilitliyken de çalışır.
- Kopya kilitsiz doğar; "Tüm özellikleri kaldır" kilidi korur.

**Veri.** Kilit proje örneğinin `locked: true` alanıdır. Proje `payload` (`{stand, modules}`) içinde taşınır; şema, migration ve proje sürümü değişmez. İçe aktarmada alan boolean olarak doğrulanır.

**SaaS etkisi.**
- Kilit organization'a ait proje kaydının içindedir; organization izolasyonu, yetki, entitlement ve kullanım ölçümü değişmez.
- Item master, katalog ve BOM etkilenmez; kilit Item/BOM özelliği değildir.

## Kamera etkileşimi düzeltmesi — 2026-10-13

OrbitControls için kaçırılan fare bırakma, pointer capture kaybı, iptal ve pencere odağı kaybı ele alındı. Normal bırakmada damping korunur; hesap gerçek geçen zamana bağlıdır ve 4/10 FPS koşullarında kare hızına bağlı uzama yapmaz. Blur veya görünürlük kaybı birikmiş kamera hareketini mevcut görünümü değiştirmeden temizler. Nesne yerleşimi, snap, Ctrl/Cmd seçimi, Shift+R ve zemin çizimi kendi akışlarını korur.

Lokal kanıt: 1.124 JavaScript testi; 10 Chromium kamera/etkileşim senaryosu; taşıma/snap, yerleşim ghost'u, zemin alanı ve ViewCube regresyonları. Syntax, build, dependency audit ve change-contract kapıları geçti. Düşük FPS ve blur regresyonları düzeltme öncesinde başarısız, düzeltme sonrasında başarılı olarak ölçüldü. Bu doğrulama fiziksel sorunlu bilgisayarın testinin yerine geçtiği veya canlı deployment yapıldığı anlamına gelmez.

Kapsam yalnız Fair Stand kamera UI/runtime davranışıdır. API, veritabanı, migration, Core sözleşmesi, organization izolasyonu, yetki, kayıt/BOM ve Item master değişikliği yoktur. GitHub teslimat kanıtının sahibi ilgili uygulama PR/CI kayıtlarıdır.

## Son doğrulanan lokal sayılar (fast-aging)

Invariant değildir. 2026-09-18 lokal implementation:

| Ölçüm | Değer |
|-------|--------|
| Categories | 6 |
| Items | 96 |
| Visible Items | 58 |
| Components | 186 |
| Assets | 19 |
| JSON / JSONB / EAV | Katalog ilişkisel. Proje `payload` JSONB `{stand, modules}`. EAV yok |
