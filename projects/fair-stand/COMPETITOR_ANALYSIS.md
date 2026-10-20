# Fair Stand — rakip analizi (ürün / stack / hız)

**Durum:** Gözlem notu (kod yok). Rakip analizinin tek kaynağı bu dosyadır.  
**Tarih:** 2026-10-18 (2026-10-19 düzeltme: kalite varsayılanı, demand render, bundle boyutları; 2026-10-20: Stand Builder kullanım karşılaştırması § 2b)  
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
Kyrox Fair Stand hantal hissediliyor çünkü birçok UI aksiyonunda `disposeWall` + tam `buildWall` çalışıyor. Stack aynı aile (Three.js) — asıl fark güncelleme yolu ve kalite seçimi. JS paketi boyutu fark değil: bizimki daha küçük (aşağıdaki tablo).

| Konu | BoothDesign | StandBuilder | Kyrox Fair Stand | Alınacak (hız) |
|---|---|---|---|---|
| Sahne update | Taşı = mesh UUID reuse | Doğrulanmadı | Sık full rebuild | Incremental scene (P3) |
| Çizim döngüsü | — | R3F; uygulamanın `frameloop` ayarı doğrulanmadı | Talep üzerine (boşta kare çizilmez) | Var; korunur |
| Kalite | **Oto:** zayıf GPU’da Hızlı, değilse Yüksek | Hafif satış sahnesi | PCF shadow 2048, sabit | GPU’ya göre oto profil (P1) |
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
| Başlangıç | 5 adımlı sihirbaz: Etkinlik → Ölçü → Yapı → Başlangıç → Editör | Tek panel: stand tipi, X / Y, duvar yüksekliği, zemin, depo → Sahneyi Oluştur → proje adı |
| Etkinlik | Fuar aranır (ör. "Istanbul" → 10 fuar); konum ve fiyat pazarı ondan gelir. Listede yoksa form | Yok (müşteri / proje CRM'de) |
| Ölçü | Genişlik / derinlik (m), açık kenar: Inline, Peninsula, Corner sağ/sol, Island; bölge para birimi (GBP / EUR / USD) | X / Y (cm, 50'nin katı), Sırt / U / L sol / L sağ / Ada |
| Yapı | Custom Stand (2,7 / 3,0 / 3,5 / 4,0 m), Shell Scheme (2,4 m), **Maxima** (3 m panel + 1 m alın, 4 m'ye kadar) | Maxima sistemi; duvar 250–500 cm (50 cm bant) |
| Başlangıç | Custom için şablon; Maxima boş yerleşimle açılır | Otomatik duvar (stand tipine göre), istenirse depo + depo içeriği |

### Editör

| | Stand Builder | Kyrox Fair Stand |
|---|---|---|
| Düzen | 4 sekme: Booth / Walls / Furniture / Cart | Kenar çubuğunda çok sayıda katlanır bölüm + katalog grupları |
| Katalog | Az ve satışa dönük ürün; **her kartta fiyat (€)**; sahneye konan ürün DRAG, sahnesiz sarf (su, kahve, priz, iPad) ADD ile listeye eklenir | Üretim parçaları da katalogda (dikme, profil, kısa duvar, ayırıcı); fiyat görünmez |
| Duvar | Custom Wall **çizilerek** (DRAW), kapı, Clear Wall | Genişliğe göre panel modülleri, şeride ekleme, snap, kısa duvar, ayırıcı |
| Fiyat | Sepet anlık toplam: "Maxima · Basic Stand €5.825" (panel, zemin, nakliye, işçilik, kurulum dahil); zemin seçeneklerinde fark fiyatı (Wood +€1.125) | Müşteriye fiyat yok; üretim listesi (BOM) teknik |
| Marka | Ayrı bölümler: Graphics & Logos, Texts; varsayılan alın "YOUR BRAND" | Panel bazında renk / görsel (Doldur / Sığdır), lightbox, mesh, cam |
| Sunum | Fuar salonu arka planı, Cinematic mode / rotation, Hide UI, Camera Views, Export | Render Al (PNG), view cube, Persp / Ortho |
| Yardım | Yapay zekâ asistanı (Vexa), "AI Generate (Beta)" (kilitli), Get Consultation | Kılavuz (?) |
| Bitiş | Save / Load, **Submit Design** (ekip iletişime geçer), Reset | Otomatik kayıt, revizyonlar, canlı paylaşım, CRM projesi |
| Upsell | Yükseltilmiş zemin (+€1.000), asma yapı, ön kemer (+€325) | Yok |

### Onlarda daha kolay
1. **Başlangıç sihirbazı:** Her adımda tek soru; kullanıcı ne yapacağını düşünmez. Bizde stand kurulumu tek ve yoğun bir panel.
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
1. Stand kurulumunu adım adım sihirbaza çevirmek (tip → ölçü → yükseklik → depo → oluştur).
2. "Basit mod": satış kullanıcısına yapısal parçaları (dikme, profil, kısa duvar) gizleyen katalog görünümü.
3. Müşteriye görünen fiyat / teklif özeti (CRM maliyetinden); en azından modül kartında ve toplamda.
4. Sahnesiz ürünler için "listeye ekle" (sarf, hizmet) — 3D yerleşim gerektirmeden.
5. Sunum katmanı: hazır ortam (salon), varsayılan marka alnı, otomatik dönen kamera.

### Uygulamalı deneme (2026-10-20, Custom Stand + "Front Arc" şablonu, anonim, kayıt yok)

| Denenen | Sonuç |
|---|---|
| Duvar yüksekliği | Editörün içinde, stand kurulduktan sonra 2,7 / 3,0 / 3,5 / 4,0 m arasında değişiyor; sahnedeki TV ve mobilya yerinde kalıyor. Bizde backlog (`PENDING_ITEM_DECISIONS.md` B.16). |
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
1. Açık standda duvar yüksekliğini değiştirme (B.16) rakipte var ve içerik korunuyor.
2. Panel üstü bağlam menüsü yaklaşımı (tek tık seç, ikinci tık seçenekler) dokunmatikte de çalışır; bizdeki sağ tık menüsüne alternatif.
3. Seçili nesnenin üstünde küçük işlem düğmeleri (taşı, boyut, döndür, kopyala, sil).
4. Duvar metni için ayrı, basit pencere (yazı tipi / renk / boyut) ve yerleştirilen metinler listesi.
5. Bizde güçlü kalan: geri al / ileri al, Maxima'da tam panel tasarımı, üretim BOM'u.

---

## 3. Üçlü özet

| | BoothDesign | StandBuilder | Kyrox Fair Stand |
|---|---|---|---|
| Odak | Modüler sistem + quote | Erken planlama + fiyat | Üretim konfigüratörü |
| 3D | Three vanilla CDN | Three + R3F + Vite | Three (`scene3d` monolit) + Vite |
| Update | In-place / kısmi | Doğrulanmadı | Sık `rebuildWall` |
| Çizim | — | R3F (`frameloop` doğrulanmadı) | Talep üzerine |
| Kalite varsayılan | Oto (GPU’ya göre) | Hafif sahne | Yüksek gölge, sabit (hedef: oto) |
| JS paketi | ~2,6 MB + three 1,27 MB | 6,4 MB tek parça | ~1,2 MB (three 0,67 + uygulama 0,52); gzip ile ana JS 0,53 MB |
| 3D model sıkıştırma | Draco | Draco / Meshopt / KTX2 | Yok (`indoor_plants.glb` 23,7 MB) |
| Kod koruma | Token’lı, süreli çekirdek | Minify | Minify, sourcemap yok |
| BOM | Sistem + satır | Sepet / fiyat | Panel / ilişki / üretim |

**Sonuç (ürün kararı değil, hız stratejisi):** Rakibi kopyalamıyoruz. Yapılacaklar [PERFORMANCE_ROADMAP.md](PERFORMANCE_ROADMAP.md); uygulama `kyrox-fair-stand` ayrı PR.
