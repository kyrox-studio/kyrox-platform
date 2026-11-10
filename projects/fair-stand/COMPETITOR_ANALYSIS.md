# Fair Stand — rakip analizi (ürün / stack / hız)

**Durum:** Gözlem notu (kod yok). Rakip analizinin tek kaynağı bu dosyadır.  
**Tarih:** Rakip gözlemleri 2026-10-18 ve 2026-10-20; 2026-11-10'da iki site yeniden gezildi (§ 1b, § 2c; hesap açılmadı, kayıt yapılmadı). Kyrox Fair Stand satırları 2026-11-06'da `main` koduna göre güncellendi.  
**Amaç:** Canlı app üzerinden BoothDesign ve StandBuilder’ı kaydetmek; Fair Stand hız stratejisine girdi.  
**Uygulama planı (yapılacaklar):** [PERFORMANCE_ROADMAP.md](PERFORMANCE_ROADMAP.md). Kod: `kyrox-fair-stand` ayrı PR.

Tahmin yazılmaz; doğrulanamayan gözlem “doğrulanmadı” diye işaretlenir. URL veya ürün değişirse bu dosya güncellenir.

---

## İzlenen ürünler

| Ürün | Firma / ekosistem | App URL |
|---|---|---|
| Booth Designer | Exphora | https://boothdesign.app/app/ |
| Stand Builder | AetherEx | https://www.standbuilder.app/app |

---

## Teşhis (kısa)

Rakip hızlı hissettiriyor çünkü sahneyi **yerinde güncelliyor** (BoothDesign), **ekran kartına göre kalite seçiyor** (BoothDesign) ve (StandBuilder’da) ürün kapsamı dar.  
Kyrox Fair Stand'de ekleme, silme, taşıma ve geri al artık sahneyi yerinde günceller; editörde gölge kapalıdır, Render Al'da açılır. Tam kurulum ölçü değiştirme gibi işlerde sürer. Stack aynı aile (Three.js). JS paketi boyutu fark değil: bizimki daha küçük (aşağıdaki tablo).

| Konu | BoothDesign | StandBuilder | Kyrox Fair Stand | Alınacak (hız) |
|---|---|---|---|---|
| Sahne update | Taşı = mesh UUID reuse | Doğrulanmadı | Ekle / sil / taşı / geri al yerinde; ölçü değişimi gibi işlerde tam kurulum | Kalan tam kurulumlar (P3) |
| Çizim döngüsü | — | R3F; uygulamanın `frameloop` ayarı doğrulanmadı | Talep üzerine (boşta kare çizilmez) | Var; korunur |
| Kalite | **Oto:** zayıf GPU’da Hızlı, değilse Yüksek | Hafif satış sahnesi | Editörde gölge kapalı, Render Al'da açık; GPU'ya göre profil yok | Kullanıcı kararıyla profil yerine bu çözüm (P1) |
| Ürün | Modüler sistem + BOM + quote | Erken planlama + fiyat | Üretim konfigüratörü | Derinliği koru; hız katmanı |

**Bilinçli erteleme:** R3F rewrite yok. Unity yok. Dimension gizmo hız P0–P2’den sonra.

---

## 1. Booth Designer — Exphora

- **URL:** https://boothdesign.app/app/  
- **Marka / sürüm ipucu:** “Booth Designer · Exphora”; UI’da `v17.10`.  
- **Ürün tipi:** Modüler fuar sistemi konfigüratörü (Aluvision, beMatrix, Octanorm, Maxima, Canopy…). Inline / Corner / Peninsula / Island; ölçü ft↔m; panel / kaplama / zemin / tavan / kapı / raf-TV / depo; kg özet; BOM; artwork / print list; quote; Operation sekmeleri.  
- **Katalog:** Exphora furniture API — Displays2go, IKEA, T-EXO, Amazon vb. GLB (`files.exphora.app`). Ürün ölçüsü gerçek SKU; düzenlenemez; fiyat/SKU paneli.  
- **Stack (gözlem):**
  - Vanilla JS (React/Vue fiber yok; `#app`)
  - Three.js **r169** CDN `importmap` (jsDelivr) + OrbitControls, GLTFLoader, DRACOLoader, **USDZExporter** (AR)
  - Uygulama çekirdeği: `/api/js/core.*.js?t=<token>` — token site adresi, ziyaretçi IP’si ve süreye bağlı; süresi geçince kod inmez (kod koruma). Lisans kökü (`_LISANS_KOK`) ve GLB dışa aktarma koruması var.
  - Paket: çekirdek ~0,96 MB + ek parça ~1,6 MB + three.js 1,27 MB (CDN); ilk açılışta toplam ~26 MB (çoğu zemin dokuları, 350–390 KB JPG).
  - Backend/katalog: `exphora.com` REST; analytics GA + Facebook Pixel
  - TR kod izleri: inline betik yorumları Türkçe ve `*.py` betiklerle eklenmiş yamalar (`kalitedugme.py`, `hatakayit.py`, `sigdir.py` …).
- **Performans gözlemi:**
  - Kalite anahtarı `exphora_kalite_secim`: `oto` / `hizli` / `yuksek`; varsayılan **`oto`**. Oto karar: GPU adı Intel HD/UHD/Iris, Mali, Adreno, PowerVR, SwiftShader ise veya ≤4 çekirdek / ≤4 GB RAM ise **Hızlı**, değilse **Yüksek**. (Zayıf makinede “varsayılan FAST” görünür; güçlü makinede Yüksek açılır.)
  - Transform (pozisyon): mesh UUID **%100 reuse** — sahne yıkılmıyor
  - Booth tipi / sistem değişimi: kısmi yenileme (~60 ms sync tıklama; mesh sayısı değişir, bir kısmı reuse)
  - Ağır sahne mümkün (~3M+ üçgen, çok texture) ama oto kalite + incremental his
- **Çıktılar:** BOM / CSV, artwork list, quote, AR (QR), snapshot, turntable video, share link, PDF/print (Tools menüsü).
- **Fair Stand için ders:** Incremental graph + GPU’ya göre kalite; üretim derinliğini kopyalamadan hız katmanı.

### 1b. Güncel gözlem (2026-11-10, anonim, giriş yok)

Arayüz artık Türkçe ("Stand Tasarımcı · Exphora"); sürüm etiketi yine `v17.10`. Açılışta "Terms of Use" bandı çıkar: uygulamanın ve 3D modellerin kopyalanması yasak (band kabul edilmedi).

| Alan | Gözlem |
|---|---|
| Mod | Basit / Uzman |
| Sistem | Custom (ahşap), Aluvision (Omni-55 / Hybrid-55, Hybrid-62), Maxima (BETA), beMatrix, Octanorm (BETA), Çadır (BETA) |
| Booth | Birim Feet / Metre; Inline / Köşe / Yarımada / Ada; hazır ölçüler 10×10 … 40×40 ft ve Özel |
| Sol bölümler | Yükseklik, Köşe, Kaplama, Zemin, Tavan (yok / ahşap lata / tam kapalı / yarım + yarım / truss + ışık), kol spot, tavan armatürü (gömme spot / spot / ray / sarkıt), armatür sayısı, renk sıcaklığı (3500K / 5000K / soğuk), duvar yıkayıcı, özel ışık ekle, header bandı (fascia), koridora bakan uç, köşe bağlantısı, birleşim noktaları |
| Alt araç çubuğu | Elektrik ana şalteri (ışık ve ekranlar), açık / koyu tema, maket görünümü, üstten, dört yönden yandan, ziyaretçi göz hizası (zemine tıkla → kamera oraya), kaydır, ortala, zemin ve kaplama, el çizimi bölücü (üstten serbest çizim → katalog ölçüsünde panel), görsel kalite, iki nokta arası ölçü, stüdyo görünümü (beyaz zemin, ölçüler gizli) |
| Üst bant | Add item, AI düğmesi, Giriş yap; ağırlık (kg) ve kalem sayısı rozeti; hazır stand tasarımları ("Browse pre-made booth designs") |
| Araçlar → Satış | **Fiyat al** (tasarımı üreticiye gönder, 24 saatte dönüş), **Teklif** (her kalem fiyatlı satır) |
| Araçlar → Üretim | **Operasyon** (onay, elektrik, malzeme, baskı sekmeleri), **Görsel Listesi** (basılacak yüzey, ölçü, dosya isteği) |
| Araçlar → Görünüm | Studio (yakında, Enterprise), Wireframe, **AR** (QR ile telefonda gerçek ölçü), Teknik özet (parsel, yapı, ağırlık), Görsel kalite (Yüksek: yumuşak gölge + salon yansıması) |
| Araçlar → Çıktı | **Anlık görüntü** (altı açıdan kare, filigran isteğe bağlı), **Tur videosu** ("Kamera stand çevresinde 360° · tarayıcıda kayıt · 1280×720") |
| Araçlar → Paylaşım | **Paylaş** (müşteriye link: izin, süre, PIN, filigran), **Görüntülenmeler** (linki kim açtı: ülke, şehir, cihaz, zaman), arkadaş daveti (EXP-XXXX kodu), Embed (Enterprise) |
| Katalog | Konsol kaydı: 1407 ürün, 824 grup, 5 marka |
| Açılış sahnesi | Hazır mobilya: danışma, tabure, masa, 4 sandalye, raf, bitki |

#### Açılan ekranlar (2026-11-10, giriş yapılmadan; hiçbir form gönderilmedi, dosya indirilmedi)

| Ekran | Giriş gerekir mi | İçerik |
|---|---|---|
| **Teklif** | Görmek için hayır; göndermek ve PDF için ücretsiz hesap | Alıcı (firma, yetkili, e-posta, telefon, fuar, salon/stand no); teklif no, tarih, geçerlilik (gün), para birimi (EUR, USD, TRY, GBP, AED, SAR). Kalemler sahneden kendiliğinden gelir ve gruplanır: Yapı (sistem frame adedi, bağlantı ve köşe elemanları), Baskı (SEG kumaş ft²), Zemin, Ekler, Mobilya (marka, SKU, ölçü, kg, liste fiyatı; ör. IKEA, Amazon, T-EXO), Hizmet (nakliye, montaj/demontaj, elektrik 3 kW, grafik hazırlık, depolama, sigorta). Birim fiyat elle girilir; ara toplam, iskonto %, KDV %, genel toplam; "+ Satır ekle"; teknik özet (fiyata girmez); "Bu rakamlar tahmindir" notu |
| **Operasyon → Tasarım onayı** | Hayır | Dört açıdan stand görseli (perspektif sol/sağ, önden, üstten plan); parsel, yapı ölçüsü, yükseklik, toplam ağırlık, palet sayısı; duvar tablosu; brief metni; "Organizatör onay e-postası" metni (kopyala); CSV indir, PDF/Yazdır |
| **Operasyon → Elektrik planı** | Hayır | Üstten planda priz yerleşimi (priz ekle, main drop, 230 V / 120 V, sahneden yenile, SVG indir); yük tablosu (öğe, W, A, V, faz) ve sipariş edilen güç (3 kW monofaze) |
| **Operasyon → Malzemeler** | Hayır | Toplam ağırlık, hacim (ölçülen + tahmin), palet adedi ve ölçüsü; malzeme listesi üretici ürün koduyla (frame, kumaş, aydınlatma, bağlantı, zemin, mobilya); palet hesabının kabulleri açıkça yazılı; teyit bekleyen kod "⚠ üretici teyidi bekliyor" işaretli |
| **Operasyon → Görsel Listesi** | Hayır | Basılacak yüzeyler: duvar iç/dış, banko ön/yanlar, zemin; sipariş, görünen ve kesim ölçüsü ayrı; görseli olmayan yüzey "Marka görseli — temin edilecek" |
| **AR görünümü** | Hayır | USDZ / GLB üretir; QR, AR linki kopyala, WhatsApp / e-posta / SMS ile gönder, GLB indir; iOS ve Android |
| **Fiyat al** | Göndermek için evet | Tasarım (ölçü, duvar, sistem, zemin, ağırlık, mobilya, görsel, malzeme listesi) **Exphora Builders Club**'a gider: doğrulanmış stand üreticileri ağı, fuar ve bölgeye göre eşleşme, karşılaştırmalı teklif. Üreticiler için "Builders Club'a katıl" bağlantısı var |
| **Paylaş** | Evet ("Paylaşmak için Exphora hesabına gir") | Açılmadı |
| **Anlık görüntü** | Tıklanınca kayıt / giriş penceresi çıktı (e-posta, şifre, Google, LinkedIn) | Açılmadı |

**İş modeli gözlemi:** BoothDesign ücretsiz tasarım aracıyla stand alıcısını toplayıp talebi üretici ağına (Builders Club) yönlendiriyor; stand üreticisi rakip değil, ağın üyesi olarak konumlanıyor. Kyrox Fair Stand ise üreticinin kendi aracı.

**Doğrulanmadı:** Tur videosu üç denemede başlamadı (konsolda kayıt satırı yok); Paylaş ve Anlık görüntü giriş istediği için tur videosu da girişe bağlı olabilir. Görüntülenmeler, Wireframe, Teknik özet ve alt araç çubuğundaki düğmeler denenmedi.

---

## 2. Stand Builder — AetherEx

- **URL:** https://www.standbuilder.app/app  
- **Marka:** AetherEx / StandBuilder; pazarlama: planlama, anlık fiyat, şablon.  
- **Ürün tipi:** Satış / erken planlama aracı — booth boyutu, duvar, mobilya, storage, TV setleri, fiyat (€); üretim panel şeridi / ilişki BOM yok.  
- **Stack (gözlem):**
  - React **18.3.1** SPA (`#root`, `createRoot`) + **Vite** hashed assets — tek parça `/assets/index-*.js` **6,4 MB**; ilk açılış toplam ~13 MB.
  - **Three.js r165** + **React Three Fiber** + drei (`@react-three/fiber`; canvas’ta `gl` / `camera` props)
  - GLB modeller (40 KB – 1,5 MB) + Draco / Meshopt / KTX2 yükleyicileri; `InstancedMesh` ve `BatchedMesh` kullanımı
  - i18next, Redux izi, leva, postprocessing, html2canvas + jsPDF, Rapier
  - Backend: **Supabase**; ödeme Stripe izi
  - Host: **Vercel** (`_vercel/insights`, speed-insights); Sentry; GA
- **Performans gözlemi:** Hazır, küçük GLB + hafif sahne + instancing; full dispose/rebuild modeli görülmedi. Bundle’da R3F’nin kütüphane varsayılanı `frameloop: "always"` görünüyor; uygulamanın `demand` kullandığı **doğrulanmadı**. “Hızlı his” büyük ölçüde ürün kapsamından (satış sahnesi) gelir.
- **Fair Stand için ders:** R3F zorunlu değil (BoothDesign vanilla ile de hızlı). Model sıkıştırma ve instancing ucuz kazanç. Kapsam sadeleştirmesi ≠ hedefimiz; üretim state’ini koruyup update yolunu düzeltiriz.

---

## 2b. Stand Builder — kullanım karşılaştırması (2026-10-20)

**Kapsam:** Canlı app tarayıcıda gezildi ve denendi (aşağıda “Uygulamalı deneme”). Hesap açılmadı, tasarım kaydedilmedi, logo yüklenmedi; kayıt, logo ve PDF akışı **doğrulanmadı**.

### Akış

| Adım | Stand Builder | Kyrox Fair Stand |
|---|---|---|
| Başlangıç | 5 adımlı sihirbaz: Etkinlik → Ölçü → Yapı → Başlangıç → Editör | 4 adımlı sihirbaz: Stand tipi → Yerleşim → Ölçü → Özet → proje adı |
| Etkinlik | Fuar aranır (ör. "Istanbul" → 10 fuar); konum ve fiyat pazarı ondan gelir. Listede yoksa form | Yok (müşteri / proje CRM'de) |
| Ölçü | Genişlik / derinlik (m), açık kenar: Inline, Peninsula, Corner sağ/sol, Island; bölge para birimi (GBP / EUR / USD) | Genişlik / Derinlik (m, 0,5 adım), Sırt / U / L sol / L sağ / Ada |
| Yapı | Custom Stand (2,7 / 3,0 / 3,5 / 4,0 m), Shell Scheme (2,4 m), **Maxima** (3 m panel + 1 m alın, 4 m'ye kadar) | Stand tipi Maksima aktif; Özel Tasarım ve Paket pasif; duvar 250–500 cm (50 cm bant) |
| Başlangıç | Custom için şablon; Maxima boş yerleşimle açılır | Otomatik duvar (yerleşime göre), istenirse depo + depo içeriği, otomatik alınlık, hazır mobilya takımı |

### Editör

| | Stand Builder | Kyrox Fair Stand |
|---|---|---|
| Düzen | 4 sekme: Booth / Walls / Furniture / Cart | Basit / Gelişmiş mod; kenar çubuğunda katlanır bölümler + katalog grupları; seçime göre Özellikler paneli |
| Katalog | Az ve satışa dönük ürün; **her kartta fiyat (€)**; sahneye konan ürün DRAG, sahnesiz sarf (su, kahve, priz, iPad) ADD ile listeye eklenir | Üretim parçaları da katalogda (dikme, profil, kısa duvar, ayırıcı; Basit modda gizleme açık); varyantlar tek kartta; fiyat görünmez |
| Duvar | Custom Wall **çizilerek** (DRAW), kapı, Clear Wall | Genişliğe göre panel modülleri, şeride ekleme, snap, kısa duvar, ayırıcı |
| Fiyat | Sepet anlık toplam: "Maxima · Basic Stand €5.825" (panel, zemin, nakliye, işçilik, kurulum dahil); zemin seçeneklerinde fark fiyatı (Wood +€1.125) | Editörde fiyat yok, maliyet CRM'de; üretim listesi ekranda okunur adlarla (kod ipucunda), TXT indirilir |
| Marka | Ayrı bölümler: Graphics & Logos, Texts; varsayılan alın "YOUR BRAND" | Panel bazında renk / görsel (Doldur / Sığdır), lightbox, mesh, cam; logo için ışıklı strafor; duvara yazı aracı yok |
| Sunum | Fuar salonu arka planı, Cinematic mode (salon ışığı; § 2c), Hide UI → Cinematic rotation, Camera Views, Export | Render Al (PNG), view cube, Persp / Ortho; salon yalnız beton zemin ve sis; dönen kamera yok |
| Yardım | Yapay zekâ asistanı (Vexa), "AI Generate (Beta)" (kilitli), Get Consultation | Kılavuz (?), "5 Dakikada İlk Standın", ilk kullanım ipuçları |
| Bitiş | Save / Load, **Submit Design** (ekip iletişime geçer), Reset | Otomatik kayıt, revizyonlar, canlı paylaşım, CRM projesi |
| Upsell | Yükseltilmiş zemin (+€1.000), asma yapı, ön kemer (+€325) | Yok |

### Onlarda daha kolay
1. **Hazır başlangıç:** Etkinlik seçimi ve Custom Stand için hazır şablon. Bizde de sihirbaz var; birden çok hazır tasarım arasından seçim yok.
2. **Fiyat görünürlüğü:** Ürün kartında fiyat ve anlık sepet toplamı; satış konuşmasının merkezi bu.
3. **Sade katalog:** Kullanıcı dikme, profil ya da kısa duvar görmez; sahnesiz ürünler tek tıkla listeye girer.
4. **Sunum:** Fuar salonu ortamı, sinematik dönüş, hazır marka alnı; ilk görüntü "satış görseli" gibi.
5. **Marka:** Logo ve yazı için ayrı, bulunması kolay bölümler.

### Bizde güçlü
1. **Üretim gerçeği:** Panel / dikme / aparat düzeyinde BOM; yan yana, köşe ve T bağlantıları; duvar yüksekliğine göre reçete. Onların Maxima'sı sabit 3 m panel + alın; bant ayrıntısı yok (gözlem: dikey tam boy paneller).
2. **Hassas yerleşim:** 50 cm ızgara, snap, çarpışma, şerit reflow, grup / kilit / şablon, geri al / ileri al.
3. **Panel düzeyinde tasarım:** Her panelde ayrı renk / görsel, lightbox, mesh, cam.
4. **İş akışı:** CRM müşterisi / projesi, otomatik kayıt, revizyonlar, canlı paylaşım, çok kiracılı yapı.

### Bizde zor (alınabilecekler, karar kullanıcıda)
1. Birden çok hazır stand tasarımı arasından seçim (sihirbaz var; bugün tek sabit mobilya takımı).
2. Basit modda yapısal parçaları (dikme, profil, kısa duvar) gizleyen katalog görünümü (mod var; kart gizleme açık).
3. Müşteriye görünen fiyat / teklif özeti (CRM maliyetinden); en azından modül kartında ve toplamda.
4. Sahnesiz ürünler için "listeye ekle" (sarf, hizmet) — 3D yerleşim gerektirmeden.
5. Sunum katmanı: tam salon ortamı, varsayılan marka alnı, otomatik dönen kamera.

### Uygulamalı deneme (2026-10-20, Custom Stand + "Front Arc" şablonu, anonim, kayıt yok)

| Denenen | Sonuç |
|---|---|
| Duvar yüksekliği | Editörün içinde, stand kurulduktan sonra 2,7 / 3,0 / 3,5 / 4,0 m arasında değişiyor; sahnedeki TV ve mobilya yerinde kalıyor. Bizde de var (2026-11-10): önizleme + onay, bantlar ve kaplama korunur, Geri al döner; genişlik/derinlik ve yerleşim sırada ([ROADMAP.md](ROADMAP.md) § 2). |
| Duvara metin | Texts → Back / Left / Right Wall → pencere: metin, yazı tipi, renk, boyut → "Add Text to Wall". Sol panelde "Placed Texts" listesi (sil). Seçilen metnin üstünde ← ↑ ↓ → (taşı), − / + (boyut), kopyala, sil. **Maxima sisteminde metin yok** ("Custom wall text is not available for Maxima"). |
| Sağ tık | Sahnede sağ tık menüsü yok. Karşılığı: panele bir tık = panel yeşil vurgulanır + "Click again for options"; ikinci tık = **Wall Options**: Graphics, Generate AI Design, Add Your Graphic, Add Custom Text, Lightbox, Arc, Circular Arc, Glass Door, Basic Door, TV ›, Storage ›, Clear Wall. |
| Nesne seçimi | TV: Kopyala / Sil. Bırakılan broşürlük: Döndür / Kopyala / Sil. Seçim yeşil vurgu. |
| Sürükle-bırak | Kartın DRAG düğmesinden zemine; çalıştı, nesne seçili halde gelir. |
| Duvar çizme | Custom Wall → DRAW: zeminde ızgara, 50 cm'ye yapışır, basılı sürükle; "Finish" ile çıkılır. |
| Geri al | Ctrl+Z bir şey geri almadı; geri al / ileri al düğmesi görülmedi. |
| Kamera | Center / Top / Right / Left View. Top View'da salon tavanı standı kapattı (gözlem). |
| Sepet | Anlık güncellendi: stand €6.402,78 + şablon parçaları + metin €0 + broşürlük €40 = €8.057,78. Çizilen özel duvar (katalogda €169) sepette görünmedi (neden doğrulanmadı). |
| Export | Düğme açıklaması "Export stand as paid PDF": dışa aktarma ücretli. |

**Bu denemeden alınabilecekler (karar kullanıcıda):**
1. Açık standda duvar yüksekliğini değiştirme (B.16) rakipte var ve içerik korunuyor; bizde 2026-11-10'dan beri var.
2. Panel üstü seçenekler dokunmatikte de çalışır. Bizde karşılığı var: tek tık, sağ tık ve uzun basma ortak Özellikler panelini açar.
3. Seçili nesnenin işlemleri: bizde seçim çubuğunda Döndür / Kopyala / Sil var, taşıma sahnede sürüklemeyle.
4. Duvar metni için ayrı, basit pencere (yazı tipi / renk / boyut) ve yerleştirilen metinler listesi.
5. Bizde güçlü kalan: geri al / ileri al, Maxima'da tam panel tasarımı, üretim BOM'u.

---

## 2c. Stand Builder — güncel gözlem (2026-11-10, anonim, kayıt yok)

| Alan | Gözlem |
|---|---|
| Akış | 5 adımlı sihirbaz değişmemiş: Event (fuar arama; "Istanbul" → Prohunt, Stationery Fair, IFF…) → Size → Structure → Start (Start from blank veya Custom Stand şablonları: Front Arc, Overhead Arch …) → Build |
| Üst bant | Hide UI, dil, Cinematic mode, Template, Export, Camera Views (Center / Top / Right / Left) |
| Cinematic mode | Kamerayı döndürmez; salonu karanlık / sinematik ışığa alır, düğme "Bright mode" olur |
| Dönen kamera | Yalnız **Hide UI** ile arayüz gizlenince sağ üstte **Cinematic rotation** çıkar. Kamera standın çevresinde sabit hızla, hafif yukarıdan sürekli döner; bir tur yaklaşık 30 sn; "Stop rotation" ile durur. **Video kaydı yok.** |
| Sol panel | Booth / Walls / Furniture / Cart; AI Generate (Beta, kilitli); Wall panel height 2,7 / 3,0 / 3,5 / 4,0 m; Booth Customization; Graphics & Logos; Texts |
| Bitiş | Save Design, Load, Submit Design, Reset Design, Get Consultation |
| Asistan | Vexa açılır pencere ("Design with Vexa") |

## 2d. Senaryo: içerik varken stand ölçüsünü değiştirme (2026-11-10, anonim, giriş yok)

**Yöntem:** Aynı senaryo iki rakipte adım adım koşuldu. BoothDesign'da konumlar sahne nesnelerinden (`_booth` çocukları, stand merkezli metre) okunup her adımda kaydedildi; ekran görüntüsü yalnız destek. Etiketler: **denendi** / **yalnız menüde görüldü** / **denenemedi**.

### BoothDesign (denendi)

Başlangıç: varsayılan "Inline" stand (sırt + iki yan duvar; bizim U'ya karşılık) 6,1 × 3,05 m; içinde sırtta tek parça baskı, TV, masa + 4 sandalye, banko, figür.

| Adım | Duvarlar | Mobilya / öğeler | Not |
|---|---|---|---|
| Genişlik 6,1 → 9 m | Stand merkezde kalır, iki yandan eşit büyür. Sırt duvarı 6 yerine 9 modülle **baştan dizilir**; baskı yeni genişliğe yayılır | x konumları **oranla ölçeklenir** (×1,49): banko 2,23 → 3,33 m; masanın iki yanındaki sandalyeler 0,62 → 0,92 m açılır (grup korunmaz). Boyutlar değişmez | Silinen yok |
| Genişlik 9 → 3 m | Sırt duvarı dar dolgu panellerle kapanır (eşit olmayan modüller) | Oranla sıkışır; taşan 1 öğe içeri itilir; öğeler üst üste binebilir | Bildirim: "Layout refitted — 1 item(s) moved to stay inside the booth." Silinen yok, onay penceresi yok |
| Genişlik 3 → 6 m | Yeniden dizilir | İçeri itilen öğe eski yerine dönmez (2,23 yerine 2,42 m) | Ölçü değişimi kayıplı |
| Derinlik 3 → 6 m | Stand yine merkezde: **sırt duvarı geriye gider**, açık kenar öne gelir; yan duvarlar iki uçtan uzar | z konumları oranla ölçeklenir (×1,96); masa–sandalye arası açılır | — |
| Derinlik 6 → 2 m | Kısalır | Oranla sıkışır, 1 öğe içeri itilir | Aynı bildirim |
| Ctrl+Z | Ölçü 6 × 6'ya döner | Bir öğe 1,99 yerine 2,72 m'ye döner | Geri al anlık görüntüden değil, yeniden ölçekleyerek; birebir değil |
| Tip Inline → Ada | Bütün duvarlar ve sırttaki baskı kalkar | Mobilya kalır | Uyarı yok |
| Ada'da ölçü değişimi | — | İçerik ortada kalır, oranla yayılır | Yalnız ekran görüntüsü; ölçülmedi |

### StandBuilder (denendi)

- 5 adımlı sihirbazda Size adımında 6 × 3 m Inline + "Basic" şablon ile kuruldu.
- **Editörde genişlik / derinlik alanı yok.** Booth, Walls, Furniture ve Cart sekmelerinde ölçü değiştiren bir şey bulunmadı. Editörden yalnız duvar yüksekliği değişiyor (2b).
- Sayfa yenilenince tasarım gidiyor, sihirbaz baştan başlıyor (giriş yapmadan kayıt yok). Ölçüyü değiştirmenin tek yolu standı baştan kurmak.

### Kyrox Fair Stand (kod, `feat/stand-yukseklik-degistir`, commit'siz)

Kaynak: `src/standResize.js` → `planStandSizeChange` ve birim testleri. Tarayıcıda bu oturumda yeniden koşulmadı.

- Önizleme + onay penceresi ("eklenecek / kalkacak / taşınacak" sayıları), tek adımda uygulanır, Ctrl+Z ile birebir geri döner.
- **Sırt / U:** genişlikte sırt duvarı iki uçtan eşit uzar / kısalır (50 cm adım), içerik ortada kalır. **L:** köşe sabit, açık uç uzar. **Ada:** bütün içerik farkın yarısı kadar kayar, ortada kalır.
- **Derinlik:** sırt duvarı yerinde kalır, açık ön kenar uzar; öğeler en yakın kenara mesafesini korur.
- Mobilya **ölçeklenmez**; mesafe korunur, gruplar tek parça kayar. Kısalan kenarda dışarıda kalan panel ve görseli kalkar (önizlemede sayılır), taşan öğe kenara yaslanır.

### Karşılaştırma

| Konu | BoothDesign | StandBuilder | Bizim karar |
|---|---|---|---|
| Ölçüyü sonradan değiştirme | Var, anında, onaysız | Yok (baştan kur) | Var, önizleme + onay |
| Sırt duvarı genişlikte | İki uçtan eşit (stand merkezli) | — | Sırt / U'da iki uçtan eşit: **aynı** |
| Derinlik | Stand merkezli: sırt duvarı geriye gider | — | Sırt duvarı sabit, ön uzar: **farklı** |
| Mobilya | Oranla ölçeklenir; grup bozulur | — | Mesafe korunur, grup tek parça |
| Küçülünce | Hiçbir şey silinmez, öğeler içeri itilir / üst üste biner | — | Dışarıdaki panel/görsel kalkar, önizlemede sayılır |
| Geri al | Var, birebir değil | — | Birebir |
| Ada | Ortada kalır | — | Ortada kalır: **aynı** |

**Tek davranış farkı:** Derinlikte rakip standı merkezde tutup sırt duvarını geriye itiyor; bizde sırt duvarı yerinde kalıyor. **Karar (kullanıcı, 2026-11-10): sırt duvarı yerinde kalır**; fuar parselinde sırt komşuya dayalıdır.

## 2e. Senaryo: içerik varken yerleşimi değiştirme (2026-11-10, anonim, giriş yok)

**BoothDesign (denendi; duvar sayısı ve öğe konumları sahne nesnelerinden okundu):** Başlangıç Inline 6,1 × 3,05 m (sırt 6 modül, sol 3, sağ 3), 12 öğe.

| Geçiş | Duvarlar | Öğeler | Not |
|---|---|---|---|
| Inline → Köşe | Sağ duvar kalkar; sırt ve sol baskısıyla kalır | 12 öğe yerinde | Onay / uyarı yok |
| Köşe → Yarımada (yalnız sırt) | Sol duvar da kalkar; sırt 7 modülle yeniden dizilir, baskı kalır | Yerinde | — |
| Yarımada → Inline | Yan duvarlar **eski baskılarıyla** geri gelir | Yerinde | Kaldırılan duvarın baskısını hatırlıyor |
| Inline → Ada | Bütün duvarlar ve baskılar kalkar | Mobilya kalır | Uyarı yok (2d) |

**StandBuilder (denendi):** Editörde yerleşim (Open Sides) değiştirme yok; yalnız sihirbazın Size adımında seçiliyor.

**Bizim karar (kullanıcı, 2026-11-10):** Rakip gibi kalan duvar korunur, kalkan kenar gider, yeni kenara duvar gelir; farklı olarak önizleme + onay var, kalkan görsel hatırlanmaz (Geri al döner), depo ve alınlık yeni yerleşime göre kurulur.

---

## 3. Üçlü özet

| | BoothDesign | StandBuilder | Kyrox Fair Stand |
|---|---|---|---|
| Odak | Modüler sistem + quote | Erken planlama + fiyat | Üretim konfigüratörü |
| 3D | Three vanilla CDN | Three + R3F + Vite | Three (`scene3d` monolit) + Vite |
| Update | In-place / kısmi | Doğrulanmadı | Ekle / sil / taşı / geri al yerinde; kalan işlerde tam kurulum |
| Çizim | — | R3F (`frameloop` doğrulanmadı) | Talep üzerine |
| Kalite varsayılan | Oto (GPU’ya göre) | Hafif sahne | Editörde gölge yok, Render Al'da var |
| JS paketi | ~2,6 MB + three 1,27 MB | 6,4 MB tek parça | ~1,4 MB (three 0,70 + uygulama 0,67; yerel `dist`, 2026-11-05) |
| 3D model sıkıştırma | Draco | Draco / Meshopt / KTX2 | Meshopt (`public/models` 12,5 MB) |
| Kod koruma | Token’lı, süreli çekirdek | Minify | Minify, sourcemap yok |
| BOM | Sistem + satır | Sepet / fiyat | Panel / ilişki / üretim |
| Fiyat / teklif | Sahneden otomatik teklif kalemleri, fiyat elle; PDF / gönderme girişle; Fiyat al → üretici ağı | Sepet anlık toplam | Editörde yok; CRM maliyet + Excel |
| Operasyon | Tasarım onayı, elektrik planı, malzeme + palet, görsel listesi; CSV / PDF | Yok | Üretim listesi (TXT), baskı satırları |
| Ölçüyü sonradan değiştirme | Var: merkezli, mobilya oranla ölçeklenir, onaysız (2d) | Yok, baştan kurulur (2d) | Var: önizleme + onay, mesafe korunur (2d) |
| Dönen kamera / video | Tur videosu 360°, 1280×720 (çalışması doğrulanmadı) | Cinematic rotation, video yok | Yok |
| Paylaşım | Link: izin, süre, PIN, filigran; kim açtı takibi | Submit Design | Canlı sekme paylaşımı (tek izleyici, 2 saat) |
| AR | Var, girişsiz (USDZ / GLB, QR, WhatsApp / e-posta / SMS) | Yok | Yok |
| Tavan / ışık | Tavan tipleri, armatür, renk sıcaklığı, duvar yıkayıcı, elektrik şalteri | Doğrulanmadı | Tül, LED projektör, raf altı LED, sahne ışığı |

**Sonuç (ürün kararı değil, hız stratejisi):** Rakibi kopyalamıyoruz. Yapılacaklar [PERFORMANCE_ROADMAP.md](PERFORMANCE_ROADMAP.md); uygulama `kyrox-fair-stand` ayrı PR.
