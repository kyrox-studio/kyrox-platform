# Fair Stand — rakip analizi (ürün / stack / hız)

**Durum:** Gözlem notu (kod yok). Rakip analizinin tek kaynağı bu dosyadır.  
**Tarih:** 2026-10-18 (2026-10-19 düzeltme: kalite varsayılanı, demand render, bundle boyutları)  
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
