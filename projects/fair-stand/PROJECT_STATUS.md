# Fair Stand — Proje Durumu

| Alan | Değer |
|------|--------|
| Son doğrulama | **2026-10-27** (ölü kod temizliği #353; canlı kabul ayrı) |
| Implementation deposu | `kyrox-studio/kyrox-fair-stand` |
| Hosted backend | Fair Stand FastAPI (`:8002`) / PostgreSQL `fair_stand` |
| Item master | İlişkisel PostgreSQL; catalog bootstrap API |
| Category master | `fair_stand_categories` |
| Catalog | Yalnız Item projeksiyonu |

## 2026-10-27 — Dikme dikey taşıma (merge + deploy)

[Stand #354](https://github.com/kyrox-studio/kyrox-fair-stand/pull/354): seçili dikme ↑/↓ ile Z'de taşınır: `0088_upright_vertical_move` dikmelerde `vertical_move`'u açar (0076'da kapalıydı), seed aynı kural. Tuş kameraya göre ekranda dikeye denk gelince çalışır (küp blok/profil ile aynı kural); adım tip `moveSnapCm` (10 cm); taban 0, tepe stand tavanı (`stepWallShortFloorZCm`). `upright_297` (sahnede 300) 350 cm standda 0…50 cm, `upright_346_5` (sahnede 346,5) en çok 3,5 cm kalkar. Yükseltilmiş dikmenin boyu uzayınca taban tavana sığacak kadar iner.

Doğrulama: **1548 Node testi**, **221 backend testi**, build ve contract:verify PASS; hedefli Chromium `vertical-move-upright-height` **4/4 PASS**; kullanıcı yerelde denedi ve onayladı. Yerel DB `0088`. **Deploy 2026-10-27:** sunucu Stand `4d04f885`, prod DB `0088_upright_vertical_move` (öncesinde yedek `/var/backups/kyrox/manual/fair_stand-pre0088-20261027-165327.dump`), `fair-stand.service` yeniden başladı, CRM ön yüzü yeniden build edildi; health OK, sekiz dikmede `vertical_move` true. Aynı deploy #352 ve #353'ü de taşıdı.

## 2026-10-27 — Belge arşivi ve ölü kod temizliği

[Stand #352](https://github.com/kyrox-studio/kyrox-fair-stand/pull/352) (merge `76720929`): `docs/items` tarihî audit/current-system belgeleri [platform arşivine](../../archive/fair-stand/docs-items/README.md) taşındı (422 dosya, sha256 manifest); iki betik `scripts/archive` altına alındı.

[Stand #353](https://github.com/kyrox-studio/kyrox-fair-stand/pull/353) (merge `36e9fafa`): kod haritasında adı tanımı dışında depoda hiç geçmediği doğrulanan ölü kod silindi. Fonksiyonlar: `toastNotifications.js` `dismissToast`/`clearToasts`, `scene3d.js` `formatMeasureCm`/`createMeshBrandaAlphaMap`, `CostItemService.get_cost_item`, `test/wallHeightProjects.mjs` `buildWallHeightProjectStand`. Tek seferlik betikler: `scripts/export-catalog-seed.mjs` (artık olmayan CRM `catalog_seed_data.py` dosyasına yazıyordu), `scripts/strip-legacy-dimension-columns.mjs`, `scripts/screenshot-glb-compare.mjs`, `backend/scripts/ensure_db.py`, `backend/scripts/verify_dimension_backfill.py`. Davranış, şema ve Alembic head değişmez.

Doğrulama: **1547 Node testi**, **220 backend testi**, build ve contract:verify PASS; hedefli Chromium `reload-restore` + `wall-media-items-contract` **6/6 PASS**.

## 2026-10-26 — Kompakt kart, dönüş ve ahşap ayak paketi (CI bekliyor)

[Stand #350](https://github.com/kyrox-studio/kyrox-fair-stand/pull/350), kaynak `4eec7970`: içerik yüksekliğinde kart; −45/−15/+15/+45 düğmeleri ve mevcut açı; yan yana Kopyala/Sil; belirgin Kilitli düğmesi; kart üstünden gerçek seçimi kaldırma. Sahnede çoklu panel/modül korunur. Düğmeler Shift+R ile ortak açı imlecini kullanır; reddedilen ara açı bir sonraki basışı durdurmaz. Sıkıştırılmış GLB'nin ahşap ayak maskesi özgün varlık koordinatlarında hesaplanır: takım, tekli ve çiftli ayrı ayrı kontrol edildi.

Yerel 0087 migration tanımlı Item dönüş adımlarını 15° yapar; NULL alt parçalar, varsayılan yönler ve kayıtlı müşteri proje açıları korunur. Önceki değerler downgrade için tutulur. Yerel DB 0087; bu paket üretime deploy edilmedi, son doğrulanmış üretim head'i 0086. Işık/pozlama denemeleri kullanıcı isteğiyle tamamen geri alındı; bu paket ilk ışık davranışını değiştirmez. Serbest sürükleme/kenar snap'i ve zorunlu strafor desteği henüz uygulanmadı.

Doğrulama: **1547 Node testi**, **209 backend testi**, build/syntax/change-contract PASS; değişen akışlara ait **43 Chromium senaryosu PASS**. Eski sentetik dokunmalar native touch olaylarına taşındı ve ilgili iki senaryo ayrıca PASS; beklenti atlama veya çakışma kontrolü gevşetme yok. Test kataloğu 15°'ye güncellendi; kopyanın 180° beklentisi 12×15° ile korunur. Migration tek-head/geri dönüş testi vardır. Tam GitHub E2E #350'de çalışır; auto-merge açık, birleşme ve canlı kabul henüz tamamlandı sayılmaz. Fiziksel telefon/gerçek müşteri projesi kabulü ayrı açıktır.

[CRM #374](https://github.com/kyrox-studio/kyrox-fair-crm/pull/374), kaynak `ad9e88f9`: mode-aware yerel FAIR_STAND_ROOT seçimi, varsayılan kardeş depo ve süreç ortamı önceliğini korur. Makine yolu/.env.local commit edilmedi. **685 frontend testi**, entegre build ve feature-contract PASS; 5173 gerçek güncel modül kaynağı doğrulandı. CRM backend/yetki değişikliği yok.

Ayrıntı ve kullanıcı kararları: [Seçim kartı yol haritası](SECIM_OZELLIK_PANELI_YOL_HARITASI.md). Aşağıdaki tarihli kayıtlar tarihsel teslimlerdir; bu bölüm yeni kart/çubuk yerleşimi için önceliklidir.

## 2026-10-24 — Özellikler paneli, katalog ve vitrin rengi (yerel)

Öğeye/yüzeye tek tık Özellikler panelini açar; boş alana tık kapatır. Seçim çubuğundaki Döndür, Kopyala, Sil ve Çoklu panel/modül korunur; kullanıcı kararıyla ⋯ kaldırılmıştır. Mevcut sağ tık ve uzun basma da aynı paneli kullanır. Renk/görsel ve desteklenen ölçüler seçimin yetenekleriyle gösterilir. Lightbox/Mesh taşıma kilidinin tıklama yayılımını durdurması, panelin capture aşamasında tıklamayı gözlemesiyle giderildi.

0084 Işıklı Strafor'u katalog kartı yapar; logo SVG'si Özellikler panelinden seçilir. 0085 vitrinin yan/yatay sunta Item kayıtlarında renk kabulünü açar; renderer veya panelde zorlanmış bir yetenek eklenmez. İki/üç gözlü vitrinin dört suntası ortak renge boyanır, cam raflar değişmez. Renk kayıt/açmada korunur. Seçim açıklaması aynen korunur; kısa pencerede kutunun sıkışıp işlemlere taşması düzeltildi. Kullanım kılavuzuna vitrin gövde rengi açıklaması eklendi. Ek düzeltme: editör kapanışında seçim çubuğu temizlenir; yeniden girişte tek etkin çubuk korunur ve eski callbacks yeni çubuğu değiştiremez. Bildirimler masaüstü/mobilde alt güvenli alan +12 px konumuna taşındı. Bu ek değişiklikle tam E2E 242/242 geçti. Önceki PR #339 kullanıcı tarafından kapatıldı; güncel teslim PR #340’tır.

**Doğrulama:** 1.515/1.515 JavaScript birim testi, 198/198 backend testi, 242/242 Chromium E2E (6,9 dk; yeniden deneme olmadan), build ve contract:verify başarılı. Vitrin E2E yalnız varsayılan rengi okumaz: normal tıkla paleti açar, dört mesh rengini ve kayıt/açmayı kontrol eder; 600 px pencerede açıklamanın işlemle çakışmadığını ölçer. Sonraki yalnız kılavuz metni eklemesi ayrıca kılavuz regresyonuyla doğrulanır.

**DB ve teslim:** Yerel PostgreSQL fair_stand 0083'ten sıralı 0084/0085 ile güncellendi; katalog bootstrap çıktısında sunta acceptsColor=true, acceptsImage=false doğrulandı. Kod son 3d03c0c6 commit’i ile [PR #340](https://github.com/kyrox-studio/kyrox-fair-stand/pull/340) üzerinden gönderildi; squash auto-merge açık, GitHub CI takip ediliyor. Önceki ortak kısa dikme PR #338 main'e birleşti (a098e723). Bu kayıt yeni bir canlı deploy veya uzak DB güncellemesi değildir.

**SaaS etkisi:** Ortak katalog yetenek bayrakları ve editör sunumu; organization verisi/yetkisi, Core sahipliği, entitlement/usage, lifecycle, secrets, job ve veri saklama sınırları değişmez. Yeni Item veya proje şeması yoktur. İlgili plan: [Seçim Özellik Paneli](SECIM_OZELLIK_PANELI_YOL_HARITASI.md).

## 2026-10-24 — Ortak kısa dikmenin taşıyıcıyla ikamesi (yerel)

Kısa panel uç uca / köşe birleşiminde kalan tek ortak kısa dikme, aynı birleşim noktasını ve yükseklik bandını karşılayan bağımsız dikme varsa BOM'dan düşülür. Kural reçetedeki 49,5 ve 99 dikmeler için ortaktır; taşıyıcının kendi satırı korunur. Konum, bandı kapsama ve mevcut 3,5 cm üst boşluk toleransı yeniden kullanılır. Aynı ortak uç ikinci kez düşülmez. Üst üste kolonların uzun dikmeye çevrilmesi kuralı değişmez.

İki eski 99 beklentisi bu kurala göre güncellendi; yeni testler genişlik/yön, karma bant, yetersiz yükseklik, uzak konum, mükerrer taşıyıcı, sıra bağımsızlığı ve kayıt normalizasyonunu kapsar. İlgili 85 birim testi ile gerçek üretim listesi ve kayıt/açma için iki Chromium senaryosu başarılı. Son tam doğrulama: 1.509 birim testi, 232/232 Chromium E2E (6,4 dk), build ve contract:verify başarılı. Yeni Item, migration veya proje şeması yoktur. Kod `2c6646f5`, [PR #338](https://github.com/kyrox-studio/kyrox-fair-stand/pull/338) ile gönderildi; PR main’e birleşti (a098e723); yerel main tabanı eşitlendi. Canlı deploy yapılmadı.

## 2026-10-24 — Kesik çizgili seçim sınırları (yerel)

Tek öğe mavi, grup/çoklu seçim tek turuncu kesik çizgili 3B sınırla gösterilir. Grup sınırı görünen üyelerin birleşik dünya sınırlarından üretilir, taşıma ve döndürmede güncellenir. Üyelerin ayrı kutuları gizlenir; yüzey seçimi mevcut düzenleme semantiğini korur. Çerçeveler raycast hedefi değildir ve Render Al çıktısından gizlenir. Proje, BOM, kilit ve taşıma kuralları değişmez. Doğrulama: 1.460 birim testi, build ve contract:verify başarılı. 230 Chromium senaryosundan 229 tanesi ilk denemede geçti; proje silme senaryosu yeniden denemede geçti (229 passed + 1 flaky, 6,8 dk). Grup senaryosu tek mavi kutuyu, tek ortak turuncu kutuyu, kesik malzemeyi ve grup işlemlerini gerçek sahnede doğrular. Uygulama commit’i `3a5a34db`, [PR #336](https://github.com/kyrox-studio/kyrox-fair-stand/pull/336) ile gönderildi; CI ve otomatik birleştirme bekleniyor.

## 2026-10-23 — Basit / Gelişmiş mod ve Ayarlar

**Implementation:** `kyrox-fair-stand`, commit `401bd238`, dal `feat/basit-mod-olcu-gorunumu` GitHub’a gönderildi. İlk paket [PR #334](https://github.com/kyrox-studio/kyrox-fair-stand/pull/334) ile main dalına birleştirildi; canlı deploy yapılmadı.

- Tek Basit / Gelişmiş seçici; Basit varsayılan. Ayarlar, Gruplar, Şablonlarım ve Standartlar yalnız Gelişmişte görünür. Ortak açılır/kapanır bölüm şablonu kullanılır.
- Gelişmiş → Ayarlar: **Herşeyi Kilitle**, **Ölçüleri göster**, **Otomatik kayıt**. Kilit kutusu bütün mevcut öğeleri kilitler veya öğe/grup kilitlerini açar; grupları çözmez. Kısmi kilitte ara durum, boş sahnede pasif; geri alma ve proje kaydı mevcut kilit akışını kullanır.
- Ölçüler Basitte kapalı, Gelişmişte açık başlar; Gelişmişte kullanıcı kapatabilir. Tercih oturumluktur, projeye yazılmaz. Izgara tekil/grup taşıma, katalog yerleştirme ve kopya önizlemesinde ölçü tercihinden bağımsız görünür.
- Sol menü ve sihirbazda genişlik/derinlik metreyle, 0,5 m adımlarıyla girilir; virgül/nokta kabul edilir. Duvar yüksekliği metre gösterir. Proje/BOM hesapları cm kalır; migration yoktur.
- Depo, alınlık ve zemin ifadeleri sadeleştirildi; sihirbaza zemin seçimi eklendi. Kaydet üst alana taşındı; paylaşım düğmesi **Ekranı paylaş** olarak adlandırıldı.
- Kılavuzda tek **Gelişmiş > Ayarlar** açılır bölümü bu üç kontrolün işlevini ve tercihlerin saklanmasını açıklar.

**Yerel kanıt:** 1.459 birim testi, 230 Chromium E2E testi, build ve contract:verify başarılı. Bu sayılar ilgili uygulama commit’inin yerel doğrulamasıdır; PR CI veya canlı kabul kaydı değildir.

**Bölgesel zemin takibi (yerel, uygulama commit’i 48fcb9c4, PR #335):** Tüm stand kaplaması iki modda da erişilebilir; Alan X/Y, genişlik/derinlik, alan kaplaması ve ekle/çiz/sil kontrolleri yalnız Gelişmişte görünür. Mod geçişi uygulanmış bölgesel zemini sahneden, proje kaydından veya BOM’dan kaldırmaz; yalnız tamamlanmamış çizimi kapatır. Genel kural: mod geçişi yalnız arayüz/yardımcı gösterimleri değiştirir, standı veya tasarım verisini değiştirmez. Ek paket doğrulaması tamamlandı: 1.459 birim testi, build ve contract:verify başarılı. 230 E2E senaryosunun 225 tanesi ilk denemede, 2 tanesi koşu içi yeniden denemede geçti; kalan 3 senaryo tek işçiyle tekrar çalıştırılıp geçti. Bölgesel zemin testi görüntünün, tüm stand/modules verisinin ve Basitte kayıt/açma sonucunun korunduğunu doğrular.

**Açık kalanlar:** Grup toplam ölçüleri uygulanmadı. Stand / Ürün ekle / Tasarım menü düzeni kullanıcı kararıyla ertelendi. Teknik kartları gizleyerek otomatik yapı çözme veya bütün duvar/depo/alınlık düzenleme tamamlanmış sayılmaz.

**SaaS etkisi:** Fair Stand editör sunumu ve mevcut proje kilitleri kapsamındadır. Mod seçimi yetki değildir. Organization izolasyonu, backend yetkilendirme, Core sahipliği, entitlement/usage, lifecycle, secrets, job ve veri saklama sınırları değişmez; yeni API/DB şeması yoktur.

### Geçmiş ve üretim ayrıntıları — 2026-10-23 yerel takip

Revizyonlar Proje içindeki başlangıçta kapalı **Geçmiş** bölümünden açılır; revizyon yoksa gizlenir. Üretim Listesi 2026-10-24’te yeniden üst banta, İleri al’ın yanına alındı; Ayrıntılar bölümü kaldırıldı. Modül ekle kartında yalnız sürüklenebilir katalog kalır (Modül Ekle düğmesi kaldırıldı); Sahneyi Sıfırla sol menünün en altında Tüm Özellikleri Kaldır’ın üstündedir. Her iki modda da erişilir; sahne yokken liste düğmesi pasiftir. Revizyon oturum/API davranışı, BOM hesabı ve mevcut liste paneli değişmez; bölüm açma/kapatma tasarıma yazmaz. Kılavuz ve hedefli gerçek tarayıcı akışları güncellendi; hedefli dört senaryo geçti. Son tam doğrulama: 1.459 birim testi, 230/230 Chromium E2E testi (6,4 dk), build ve contract:verify başarılı. Üst bant küçülünce seçim çubuğunun Kaydet üzerine gelmesi düzeltildi; gerçek seçim/kayıt akışında çakışma kontrol edildi. Bu ek paket [PR #335](https://github.com/kyrox-studio/kyrox-fair-stand/pull/335) ile gönderildi; GitHub CI başarılı; PR #335 otomatik olarak main dalına birleştirildi (merge `9ba4b6c3`).

## 2026-10-22 — Profil kartı ve boy seçimi (migration 0078)

**[Fair Stand PR #322](https://github.com/kyrox-studio/kyrox-fair-stand/pull/322) main'e merge edildi
(2026-10-22). Canlı deploy yapılmadı; canlı veritabanına `0078` alınmalı.**

- Profil dikme gibi katalogda tek kartla gelir (`profile_289`); bırakırken boy sorulur
  ("Bir daha sorma" tercihi, sağ tık **Profil boyu…**). Gerçek boy (BOM) → sahne boyu:
  41,5 → 50, 91 → 100, 140,5 → 150, 190 → 200, 239,5 → 250, 289 → 300, 388 → 400, 487 → 500.
- Migration `0078_profile_lengths`: admin ekranında açılmış `profile_289_scene` kartı
  `profile_289` olur (proje ve grup şablonu JSON'u dahil); `profile_239_5`, `profile_388`,
  `profile_487` eklenir; katalogda yalnız `profile_289` görünür. Gizli boylar sahnede çizilir.
- BOM: her profil kendi `itemKey` ×1 adet; duvar reçeteleri değişmez.
- Doğrulama: 1408 frontend unit/integration, 181 backend, build, change gate, tam Chromium
  E2E 210/210, CI yeşil. Yerel PostgreSQL'de `0077 → 0078` denendi (3 projede anahtar çevrildi).
- SaaS etkisi: katalog verisi değişir (migration); API ve yetki modeli aynı.

## 2026-10-22 — Sahne ışığı, alınlık aydınlatması, uzak panel titremesi

**[Fair Stand PR #321](https://github.com/kyrox-studio/kyrox-fair-stand/pull/321) main'e merge edildi
(2026-10-22). Canlı deploy yapılmadı.**

- Standın tepesinde gölgesiz sahne spotu: zemine stand merkezli, köşeleri 2 m aşan
  yumuşak kenarlı ışık havuzu. Sahne ışığında ortam ışığı ters çevrilir (gök rengi koyu,
  yer rengi açık): zemin ve hol karanlık, alınlık gibi dikey yüzler aydınlık. Editörde ana
  ışık kapanır; ışık sayısı sahne ışığı kapalı editörle aynı kalır (ilk CI'da tablet çift
  dokunuşu 350 ms eşiğini aşıyordu; test gevşetilmeden ışıklar azaltıldı).
- Üst bantta **Sahne Işığı** düğmesi: varsayılan açık, oturumluk, projeye kaydedilmez.
  CRM içindeki konfigüratörde de (`configuratorMarkup.js`) aynı düğme vardır; iki HTML
  kaynağının üst bandı testle eşit tutulur.
  Render Al her zaman sahne ışığıyla çizer; projektör/raf altı ışıkları yalnız Render Al'da.
- Otomatik alınlık gövdesi içeriden açık renk; BOM ve arka yüz paneli değişmez.
- Perspektif kamera `near` değeri bakılan noktaya uzaklığın %1'i (0.05–1 m): büyük
  standlarda uzak panellerdeki derinlik titremesi (z-fighting) kalktı.
- Doğrulama: 1402 frontend unit/integration, build, change gate, tam Chromium E2E
  209/209 ve CI yeşil. Timeout/tolerans gevşetilmedi.
- SaaS etkisi: yalnız istemci görünümü; backend şeması, API ve Alembic migration yok.

## 2026-10-22 — Yerel seçim ve kilit değişiklikleri

**[Fair Stand PR #318](https://github.com/kyrox-studio/kyrox-fair-stand/pull/318) main'e merge edildi (2026-10-21).
Canlı deploy bu kayıtta doğrulanmadı.**
Ürün kuralları [Grup/Kilit kaydında](GROUP_LOCK_ROADMAP.md#2026-10-22--yerelde-uygulanan-seçim-ve-kilit-ekleri).

- Yeni standın otomatik duvarları kilitli gelir; Ctrl/Cmd+A tüm sahne modüllerini
  seçer. Toplu kilit, kilit açma, grup kurma/çözme ve onaylı silme uygulandı.
- Kılavuz, teknik uygulama kaydı ve makine tarafından okunan change contract
  güncellendi. Vitrin/broşürlük ürün ölçüleri kılavuzdan kaldırıldı.
- Yerel doğrulama: 1353 frontend unit/integration ve 177 backend testi,
  syntax, build ve change gate geçti.
  Tam Chromium koşusunda 203 senaryonun 202'si geçti; eski kilitsiz başlangıç
  beklentisi güncellenen test aynı koşullarla yeniden geçti (dosyanın tamamı 5/5,
  son başarısızları yeniden çalıştırma 1/1). Timeout/tolerans gevşetilmedi.
- 78 öğelik gerçek sahne kaydı açıldı: 26 duvar, 26 lamba, 26 masa; tamamının
  seçimi ve toplu kilitlenmesi doğrulandı. Silme iptali, kilit koruması, tek adımda
  geri alma, grup çözme ve metin alanındaki Ctrl+A da tarayıcıda doğrulandı.
- SaaS etkisi: mevcut proje örneği düzenleme/kayıt mekanizması kullanılır;
  organizasyon sınırı, yetki kataloğu, Core, kota, job, veri saklama ve backend
  şeması değişmez. Yeni API veya Alembic migration gerekmez.

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
| Migration | Kod, yerel DB ve prod DB head `0088_upright_vertical_move` (prod'a 2026-10-27 deploy edildi). Önceki kayıt: yerel kod ve DB head `0086_variant_family_cards` (yerel DB 2026-10-25 0085 → 0086 yükseltildi; admin kaynaklı `tv_85` de 0086 ile TV ailesine katıldı; ortamda yoksa etkilenmez). `0086`: Vitrin (2/3 göz, kart 3 göz; Panel & Duvar altına taşındı), Raf (tekli/2/3 sıralı, kart tekli), TV (42/55/65, kart 42), Video Wall (2×2/3×3, kart 2×2) ve Uzun Saksı (100/150/200, kart 100) `variant` (`wall-showcase`, `shelf-row`, `tv`, `video-wall`, `long-planter`) ile tek kart; Ayaklı Vitrin Banko & Baza altına taşındı; Item/reçete/proje değişmez. Varyant Özellikler panelinde seçilir (bırakmada pencere yok). 0084/0085 yerelde uygulanmış, henüz gönderilmemiştir. Uzak DB için son burada kayıtlı deploy 0081’dir. Main’e birleşen #337: `0082_panel_separator_cards` + `0083_banko_base_cards` (`feat/katalog-tekli-ekle`; 0083: Banko / Köşe Banko / Baza `variant` `desk-banko` / `desk-banko-l` / `base`, kart 100'lük, 150 ve 200 gizli) — Panel (50/100/150/200) ve Separatör (50/100 + sarmaşıklı) aileleri `variant` ile işaretlenir (`wall-panel`, `wall-separator`, `wall-separator-sarmasik`), katalogda `wall_100_350` ve `wall_separator_100_350` kalır, diğer altı Item gizlenir; Item/reçete/proje değişmez. `0081_profile_346_5`: katalogda sahne profili kalmaz; admin kaynaklı `profile_X_scene` kartları aynı ölçülü normal profile çevrilir (proje/revizyon/şablon anahtarları dahil), `profile_346_5` (gerçek 346,5 / sahne 350) eklenir. 2026-10-23 DB eşitleme: lokal Fair Stand DB'si (şema, katalog, projeler, görseller) sunucuya bütünüyle kopyalandı; 28 tablo ve şema iki tarafta birebir aynı (md5 ile doğrulandı). Projeler güncel kataloğa göre normalize edildi (panel kimlik/ölçüleri, profil kesiti 10, dikme sahne ölçüsü 10 × 10 × 350). Lokal Core/CRM kendi verisini korur. Yedekler: sunucu `/var/backups/kyrox/manual-20261023-*`, lokal `Kyrox/db-yedek/`. `0080_item_length_parts`: `0080_item_length_parts` uzun profil/dikmenin parça bölünüşünü BOM etiketine ekler (`fair_stand_item_length_parts`; `upright_495` = 346,5 + 8 + 140,5; sahnede çizilmeyen `profile_8`); BOM miktarı değişmez. `0079_wall_short_cards` (#323): kısa panel tek kart — 6 genişlik katalogda gizlenir, Tek/Çift Sıra için birer kart kalır, ölçü ailelerinin adları DB'de ölçülü ("Panel Tek Sıra W", "Profil 41,5", "Dikme 346,5"); yerel DB 0079. `0078_profile_lengths` profil boylarını düzenler: `profile_289_scene` → `profile_289`, `profile_239_5` / `profile_388` / `profile_487` eklenir, sahne boyları 50…500, katalogda yalnız `profile_289`. `0077_upright_render` bütün dikmeleri sahnede çizilir yapar (`is_render`; 400+ cm dikme bırakma hatası). `0076_vertical_move` dikey taşınmayı ve taban referansını öğe ayarı yapar (`fair_stand_items.vertical_move`, `z_reference`: floor | wall-top). `0075_glass_bar_table` Bar Masası'nı (`glass_bar_table`, cam masa ailesi, 108 cm) ekler. `0073_move_snap_10cm` bütün türlerin taşıma adımını 10 cm yapar. `0074_showcase_brochure` ayaklı vitrin (`floor_showcase_100`) ve broşürlük (`brochure_stand`) tekil Item'larını, türlerini ve katalog önizlemelerini ekler. `0072` raf kartlarını (Tekli / 2 / 3 sıralı), `fair_stand_item_width_variants` tablosunu ve sıralı raf reçetelerini ekler. `0068`–`0071` duvar yüksekliği bant reçeteleri: `fair_stand_item_components.per_band` ve `height_matched`; düz duvar + dikmeler (`0068`), kapı ve 2/3 gözlü vitrin (`0069`), 100'lük ayırıcılar (`0070`), 50'lik ayırıcılar ve 350 reçete düzeltmesi (`0071`). `0067` organizasyon grup şablonlarını (`fair_stand_group_templates`) ekler. `0065_metal_separator` katalogda görünmeyen `metal_separator` Item'ını ekler; `0066` bu Item'ı `metre_kare` birimiyle maliyetlendirilebilir yapar (dönüştürülmüş panel bloğu adet değil en × boy olarak faturalanır). `fair_stand_cost_items` organization Item fiyatını ve manuel kalemi aynı tabloda tutar. ITEM satırında `item_key` zorunlu, `name` ve `unit` boştur. MANUAL satırında `item_key` boştur, `name` ve `unit` zorunludur. `unit` → `fair_stand_units.unit_key`, ON UPDATE CASCADE, ON DELETE CASCADE. Aynı organization aynı Item’ı iki kez yazamaz. Manuel ad ve birim için ek unique yoktur. `fair_stand_items` fiyat kolonu taşımaz. `fair_stand_items.is_cost_enabled` NOT NULL, default false; mevcut satırlar false kalır. Bu bayrak fiyat, birim ve reçete miktarı değildir. `fair_stand_item_type` sınıflandırmadır. Sahne davranışı isteğe bağlı `fair_stand_item_type_scene_behavior` satırıdır; satır yoksa tip non-scene’dir. `production` behavior satırı taşımaz. `digital_print`, `mesh_fabric`, `lightbox_fabric` bu tipe bağlı gizli Item’lardır. `foam_logo` kaldırıldı. Strafor metrekare `illuminated-foam` sahne kaydına yazılır. `tulle_fabric` (görünen ad Tül, tip `tulle-fabric`) tek Item’dır: katalogda görünür, sahneye sürüklenir, BOM miktarı en × boy / 10000 m² olarak aynı `itemKey` ile yazılır ve maliyet kalemi bu `itemKey` ile eşleşir. Kalınlık item master `depth_cm` 0.4 değeridir. İlk yerleşim `default_z_cm` 350 değerini kullanır. `fair_stand_items.unit` → `fair_stand_units.unit_key` FK’sidir: ON UPDATE CASCADE, ON DELETE RESTRICT. Proje BOM’u katalogdaki `elektrik_panosu` satırını her listeye 1 adet yazar; bu satır modül reçetesi değildir. Aynı liste Dijital Baskı, Mesh ve Lightbox alanlarını kendi production kalemlerine, strafor alanını `illuminated-foam` kaydına bağlar. |

Mimari ayrıntı: [ITEM_CATALOG_ARCHITECTURE.md](ITEM_CATALOG_ARCHITECTURE.md).

## Düz duvara arka yüz paneli — 2026-10-21

`kyrox-fair-stand` dal `feat/arka-yuz-panel` (push edildi, PR yok; raf dalının üstünde, #313 sonrası main'e alınır). Migration yok.

- Sağ tık "Arka yüze panel ekle / kaldır": düz duvarlar (50/100/150/200, kısa duvarlar). Arka paneller seçilir, görsel/cam/Lightbox/metal alır.
- BOM tek çerçeve + iki yüz panel: `wall_100` → 2 × `profile_91`, 2 × `upright_346_5`, 14 × `panel_98`. Köşede içe bakan yüz `panel_corner_*` olur; T'de gövde bugünkü kuralla düz kalır (karar bekliyor).

## Bar Masası — 2026-10-22

`kyrox-fair-stand` dal `feat/bar-masasi`. Migration `0075_glass_bar_table` (yerelde uygulandı; canlıya alınmadan önce sorulacak).

- `glass_bar_table` "Bar Masası": tekli cam masa ile aynı tür (`table-glass`), çizim ve BOM politikası; yalnız yükseklik 108 cm. Ayak masa yüksekliğine göre uzar. Extra kategorisinin sonunda.

## Izgara, ölçüler, tepeden ışık, 10 cm adım; ayaklı vitrin ve broşürlük — 2026-10-21

`kyrox-fair-stand` PR #316 (merge edildi). Migration `0073_move_snap_10cm`, `0074_showcase_brochure` (yerelde uygulandı; canlıya alınmadan önce sorulacak).

- Taşırken stand zemininde 1 m kalın / 50 cm ince ızgara; boşta gizli. Tek öğe seçiliyken taban alanı + genişlik/derinlik/yükseklik; taşırken stand kenarlarına dört uzaklık. Ölçü ve ızgara Render Al'a girmez.
- Ana ışık tepeden stand merkezine; editörde gölge kapalı, Render Al'da açık.
- Bütün taşıma adımları 10 cm; serbest öğe kenara yalnız 10 cm içinde yaslanır (önce 50 cm'den köşeye çekiliyordu).
- Ayaklı Vitrin 100: 100 × 40, 70 cm gövde + 30 cm cam fanus; ön/sol/sağ yüz renk, görsel, Lightbox. Broşürlük: 30 × 30 × 150, 5 cep. İkisi de BOM 1 adet; fiyat bekleniyor.
- Hata: kopya/katalog yerleştirirken Shift+R seçili asıl öğeyi de döndürüyordu; düzeldi.

## Raf: Tekli / 2 sıralı / 3 sıralı — 2026-10-21

`kyrox-fair-stand` PR #313 (merge edildi). Migration `0072_shelf_rows` 2026-10-21 deploy ile canlıda.

- Katalogda üç kart: Tekli raf, 2 sıralı raf, 3 sıralı raf; kartta genişlik yok. Genişlik takıldığı duvar modülünden gelir (100 / 150 / 200); 50 cm duvar ve duvar dışı reddedilir; başka duvara taşınınca genişlik yeniden seçilir.
- BOM tekli raf reçetesinin sıra sayısı katıdır: 2 sıralı 150 → 2 tahta + 6 ayak; 3 sıralı 100 → 3 tahta + 6 ayak.
- Sıralar 50 cm arayla, alt tahta panel birleşimine oturur, üst tahta duvarın içinde kalır. Raf altı ışık bütün sıralarda yanar. Ayak çizilmez (DB ölçüsü yok), yalnız BOM'dadır.
- Eski projelerdeki tekli raflar (`shelf_100/150/200`) aynen açılır, BOM değişmez.

## Panel duvarı çerçevesi: dört kenar aynı kesit — 2026-10-21

`kyrox-fair-stand` PR #312 (açık; dal `feat/panel-cerceve-gorunumu`, kullanıcı lokalde denedi). Yalnız 3B görünüm; migration yok, BOM ve kayıt biçimi değişmez.

- Düz panel, bazalı duvar, kapı ve vitrinde iki dikme ile alt/üst profil aynı kesitte çizilir (3ds Max referansındaki kapalı çerçeve); profil kenarlarında koyu ton kenar çizgisi.
- Kesit modül reçetesindeki profil Item'ının sahne ölçüsüdür (`fair_stand_item_scene_dimensions`, yerel DB'de 10 × 10). Reçetesinde profil olmayan modülde stand `frame_depth_cm` ile kare kesit. Stand `frame_width_cm` sahnede okunmaz.
- Paneller profillerin 15 mm önünde durur ve çerçevenin dış kenarından alt, üst, sağ ve sol 4 cm içeride biter; çerçeve panel çevresinde görünür. Panel kalınlığı reçetedeki panel Item'ından (0,8 cm). TV, raf ve strafor panel yüzüne oturur. Vitrinde dikme gövde yüksekliğinde çizilmez (gövdenin içinden geçmez).
- Baza, banko ve L banko aynı görünüm: kare kesitli köşe dikmeleri ve raylar, paneller profilin 15 mm önünde ve dört yanda 4 cm içeride. Tabla ve ölçüler değişmez.
- Bütün profil ve dikmeler kanallı alüminyum profil görünümünde: dört yüzün ortasında 2 cm ağızlı, 1,5 cm derin kanal; dış ölçü aynı.

## Sahne editörü: yalnız Kaydet, Kopyala, çift tık seçimi — 2026-10-20

`kyrox-fair-stand` PR #309 (açık; merge ve deploy yok, kullanıcı lokalde deniyor).

- **Otomatik kayıt kapalıyken yalnız Kaydet:** görsel ekleme/kaldırma/silme, sekme gizleme ve editörden çıkış kaydetmez; revizyon yalnız Kaydet ile ilerler. Aç / Sahneyi Oluştur / İçe Aktar / proje seçimi onay sorar, Dışarı Aktar son kaydı aktarır, "← Standlar" "Kaydetmeden çıkılsın mı?" diye sorar. CRM değişmedi.
- **Kopyala:** sağ tık menüsündeki Çoğalt Sağ / Sol kaldırıldı; seçim çubuğundaki Kopyala ve Ctrl/Cmd + C modülü hafızaya alır, her tıklama/dokunuşta katalogla aynı kuralla bir kopya bırakır (Esc / Kopyalamayı bitir; Ctrl/Cmd + V yeniden başlar). Grubu çoğalt aynı.
- **Seçim:** çift tık / çift dokunuş modülün tamamını seçer; Ctrl/Cmd + tık seçilenin çoğulu (modül seçiliyse modül ekle/çıkar, panel veya banko yüzü seçiliyse çoklu yüzey). **Shift + tık ile modül seçimi kaldırıldı.**
- **Tablet:** Çoklu panelde ikinci panele dokunmak Ctrl + tık gibi seçer.

## Yeni proje sihirbazı — 2026-10-20

`kyrox-fair-stand` (PR açık; merge kullanıcı testinden sonra). Yeni projede boş sahnede 3 adımlı kurulum (stand tipi, ölçü + duvar yüksekliği + depo, özet + Oluştur) mevcut kurulum alanlarını ve Sahneyi Oluştur akışını kullanır. İlk sahneden sonra 3 ipucu bir kez gösterilir. Veritabanı / migration yok. Depo seçimi tek liste: "Depo yok" + ölçüler; depo seçiliyken "Depo içeriği eklensin" açılır (sol menü ve sihirbaz).

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
