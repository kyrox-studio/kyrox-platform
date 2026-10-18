# Fair Stand — rakip analizi (ürün / stack / hız)

**Durum:** Gözlem notu (kod yok).  
**Tarih:** 2026-10-18  
**Amaç:** Canlı app üzerinden BoothDesign ve StandBuilder’ı kaydetmek; Fair Stand hız stratejisine girdi.  
**Uygulama planı (yapılacaklar):** [PERFORMANCE_ROADMAP.md](PERFORMANCE_ROADMAP.md) (P0–P4). Kod: `kyrox-fair-stand` ayrı PR.

Tahmin yazılmaz. URL veya ürün değişirse bu dosya güncellenir.

---

## İzlenen ürünler

| Ürün | Firma / ekosistem | App URL |
|---|---|---|
| Booth Designer | Exphora | https://boothdesign.app/app/ |
| Stand Builder | AetherEx | https://www.standbuilder.app/app |

---

## Teşhis (kısa)

Rakip hızlı çünkü **transform in-place** (veya demand render) + **düşük kalite varsayılanı** + (StandBuilder’da) daha dar ürün kapsamı.  
Kyrox Fair Stand hantal hissediliyor çünkü birçok UI aksiyonunda `disposeWall` + tam `buildWall`. Stack aynı aile (Three.js) — asıl fark güncelleme yolu ve kalite varsayılanı.

| Konu | BoothDesign | StandBuilder | Kyrox Fair Stand | Alınacak (hız) |
|---|---|---|---|---|
| Sahne update | Taşı = mesh UUID reuse | R3F demand render | Sık full rebuild | Incremental scene (P3) |
| Kalite | Varsayılan FAST | Hafif satış sahnesi | PCF shadow 2048 | `low` / `balanced` / `high` (P1) |
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
  - Uygulama çekirdeği: `/api/js/core.*.js` (token’lı; dışarıdan 403)
  - Backend/katalog: `exphora.com` REST; analytics GA + Facebook Pixel
  - TR kod izleri (hata/event alan adları, fonksiyon isimleri)
- **Performans gözlemi:**
  - Visual quality varsayılan **FAST** (gölge / AA kapalı; “High” seçilebilir)
  - Transform (pozisyon): mesh UUID **%100 reuse** — sahne yıkılmıyor
  - Booth tipi / sistem değişimi: kısmi yenileme (~60 ms sync tıklama; mesh sayısı değişir, bir kısmı reuse)
  - Ağır sahne mümkün (~3M+ üçgen, çok texture) ama FAST + incremental his
- **Çıktılar:** BOM / CSV, artwork list, quote, AR (QR), snapshot, turntable video, share link, PDF/print (Tools menüsü).
- **Fair Stand için ders:** Incremental graph + varsayılan düşük kalite; üretim derinliğini kopyalamadan hız katmanı.

---

## 2. Stand Builder — AetherEx

- **URL:** https://www.standbuilder.app/app  
- **Marka:** AetherEx / StandBuilder; pazarlama: planlama, anlık fiyat, şablon.  
- **Ürün tipi:** Satış / erken planlama aracı — booth boyutu, duvar, mobilya, storage, TV setleri, fiyat (€); üretim panel şeridi / ilişki BOM yok.  
- **Stack (gözlem):**
  - React SPA (`#root`, `createRoot`) + **Vite** hashed assets (`/assets/index-*.js`)
  - **Three.js r165** + **React Three Fiber** (`@react-three/fiber`; canvas’ta `gl` / `camera` props)
  - GLB modeller + Google Draco 1.5.5
  - i18next, Redux izi, leva, postprocessing, react-helmet
  - Backend: **Supabase** (`api.aetherex.app/rest/v1/...`)
  - Host: **Vercel** (`_vercel/insights`, speed-insights); Sentry; GA
- **Performans gözlemi:** Hazır GLB + hafif sahne + R3F demand render; full dispose/rebuild modeli yok. “Hızlı his” büyük ölçüde ürün kapsamından (satış sahnesi) gelir.
- **Fair Stand için ders:** R3F zorunlu değil (BoothDesign vanilla ile de hızlı). Kapsam sadeleştirmesi ≠ hedefimiz; üretim state’ini koruyup update yolunu düzeltiriz.

---

## 3. Üçlü özet

| | BoothDesign | StandBuilder | Kyrox Fair Stand |
|---|---|---|---|
| Odak | Modüler sistem + quote | Erken planlama + fiyat | Üretim konfigüratörü |
| 3D | Three vanilla CDN | Three + R3F + Vite | Three (`scene3d` monolit) |
| Update | In-place / kısmi | Demand render | Sık `rebuildWall` |
| Kalite varsayılan | FAST | Hafif sahne | Yüksek gölge (hedef: balanced) |
| BOM | Sistem + satır | Sepet / fiyat | Panel / ilişki / üretim |

**Sonuç (ürün kararı değil, hız stratejisi):** Rakibi kopyalamıyoruz. Yapılacaklar [PERFORMANCE_ROADMAP.md](PERFORMANCE_ROADMAP.md) P0–P4; uygulama `kyrox-fair-stand` ayrı PR.
