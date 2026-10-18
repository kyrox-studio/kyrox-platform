# Fair Stand — performans + ürün yol haritası

**Durum:** Aktif plan. Ağ katmanı YAPILDI (§ 3 P-ağ); P0–P4 için kod yok, uygulama kyrox-fair-stand ayrı PR.  
**Kaynak:** Prod gözlem (2026-09-24) + kod (2026-10-19 doğrulandı) + rakip tarayıcı analizi (2026-10-18).  
**Hedef ölçek:** Eşzamanlı ~100 aktif stand tasarımı.  
**Kanon:** bu dosya (kyrox-platform/projects/fair-stand/PERFORMANCE_ROADMAP.md). Rakip analizi: [COMPETITOR_ANALYSIS.md](COMPETITOR_ANALYSIS.md) (tek kaynak).

İlgili (kod deposu): kyrox-fair-stand — `audit/evidence/A17_PERFORMANCE_BUNDLE_RENDER_LIFECYCLE.md`, `docs/refactor/PENDING_ITEM_DECISIONS.md` B.3b.

---

## 0. Teşhis (rakip vs biz)

Rakip hızlı hissettiriyor çünkü sahneyi **yerinde güncelliyor** ve **kaliteyi ekran kartına göre seçiyor** (BoothDesign: oto → zayıf GPU’da Hızlı).  
Biz hantal hissediyoruz çünkü birçok UI aksiyonunda `disposeWall` + tam `buildWall` çalışıyor (`main.js` içinde 27 `rebuildWall()` çağrısı). Stack aynı aile (Three.js) — sorun güncelleme yolu. JS paketi sorun değil: bizimki rakiplerden küçük. Ayrıntı: [COMPETITOR_ANALYSIS.md](COMPETITOR_ANALYSIS.md).

| Konu | Kyrox (bugün) | Alınacak |
|---|---|---|
| Sahne update | Sık full rebuild | Incremental (P3) |
| Çizim döngüsü | Talep üzerine (boşta kare yok) | Korunur |
| Kalite | PCF 2048, sabit | GPU’ya göre oto profil (P1) |
| 3D modeller | Sıkıştırılmamış GLB | Meshopt/Draco (P0-M) |
| Ürün | Üretim konfigüratörü | Derinliği koru; hız katmanı |

**Bilinçli erteleme:** R3F rewrite yok. Unity yok. Dimension gizmo P3 sonrası.

---

## 1. Gözlemlenen belirtiler

| Belirti | Şiddet | Not |
|---|---|---|
| Kayıtlı projeyi sahneye açma | **5–10 sn** (2026-09-24; yeniden ölçülecek — P0 adım 1) | En acil UX (P0) |
| Sahneyi döndürme (orbit) | Takılma | Gölge (P1) |
| Modül sürükleme | Ara sıra spike | Snap + rebuild çakışması (P2) |
| Nedensiz donma | Bazen | Arka plan `rebuildWall` / texture |

---

## 2. Kök nedenler (özet)

1. **Proje açma:** her açılışta bütün görseller yeniden iner (paralel, sınırsız) + sıkıştırılmamış GLB + tam `buildWall`.  
2. **Orbit:** `shadowMap` 2048 (PCF) + 46 yerde `castShadow`.  
3. **UI:** `main.js` → sık `rebuildWall()` (dispose + tüm fabrikalar).  
4. CRM iframe / bundle: ikincil (JS paketi küçük; gzip + önbellek açık).

### 2.1 Proje açma pipeline (detay)

Sıra: `openStoredProject` → `loadProject` → `restoreProject` → `loadAssetsForActiveProject` → `createStage` → **`rebuildWall` / `buildWall`**.

| Katman | Dosya / fonksiyon | Sorun |
|---|---|---|
| Remote proje + görseller | `projectRemote.js` → `cacheProjectAndAssets` | Her açılışta API; görseller **paralel** (`Promise.allSettled`) ama eşzamanlılık sınırı yok ve önbellek yok — aynı görsel her açılışta yeniden iner |
| Görsel kayıt | `main.js` → `registerAsset` | Her asset için `createObjectURL` |
| Sahne rebuild | `scene3d.js` → `buildWall` | `disposeWall` + tüm modül fabrikası |
| Panel texture | `applyStoredImage` / `createFittedCanvas` | Ana thread |
| GLB | `loadGltfScene` / `loadItemModel` | Sıkıştırılmamış; ilk indirme ve parse pahalı |

### 2.2 Orbit

| Ayar | Etki |
|---|---|
| PCF shadow, 2048 map | GPU maliyeti yüksek |
| 46 yerde `castShadow` | Her karede shadow pass |
| Çizim talep üzerine; orbit/sürüklemede üst sınır 50 fps (dokunmatikte 30), boşta kare çizilmez | Sınır sorun değil; takılmanın kaynağı gölge |

### 2.3 Sürükleme / rebuild

- `pointermove`: snap, ghost, duvar hattı.  
- Birçok UI aksiyonu → **`rebuildWall()`**; orbit/sürükle ile çakışınca donma.

### 2.4 Ağ yükü

| Kalem | Boyut | Not |
|---|---|---|
| Ana JS (`index-*.js`) | 1,99 MB ham → **0,53 MB gzip** | YAPILDI (P-ağ) |
| `mountFairStand-*.js` | 0,54 MB ham | gzip ile ~0,16 MB |
| `public/models/` toplam | **34 MB** | Sıkıştırma yok (P0-M) |
| `indoor_plants.glb` | **23,7 MB** | Tek dosya; en büyük kalem |
| `bar_chair.glb` | 3,7 MB | |
| Draco / Meshopt / KTX2 yükleyici | Yok | Rakiplerin ikisinde de var |

---

## 3. Fazlar (sıra sert)

### P-ağ — nginx gzip + statik önbellek — **YAPILDI (2026-10-19)**

JS, CSS, JSON, SVG, GLB gzip ile; `/assets/` 1 yıl `immutable`; `index.html` `no-cache`; `/models/` ve `/textures/` ETag ile 304. İlk kurulum ve her deploy eksik kuralları ekler. Kod: `kyrox-fair-crm` PR #355; belge: [NGINX_SSL_CONFIGURATION.md](../fair-crm/ops/NGINX_SSL_CONFIGURATION.md). Canlı ölçüm: ana JS 1,99 → 0,53 MB; değişmemiş model 304.

### P0 — Proje açma (hedef tipik &lt; 2–3 sn)

1. **Ölçüm önce:** `performance.mark` (loadProject / cacheAssets / GLB / buildWall); 0, 5, 20 görselli projede ilk ve ikinci açılış süresi. Hedefler bu sayılara göre kesinleşir.  
2. Asset cache: `updatedAt` + metadata aynıysa HTTP yok.  
3. Eşzamanlı indirme sınırı (4–6) — bugün sınırsız paralel.  
4. Aşamalı overlay.

**Kod:** `projectRemote.js`, `main.js`. **Durum:** BACKLOG.

### P0-M — 3D model sıkıştırma (P0 ile paralel; düşük risk)

1. GLB’leri Meshopt (veya Draco) + gerekiyorsa doku küçültme ile sıkıştır; önce `indoor_plants.glb`.  
2. `GLTFLoader`’a `MeshoptDecoder` / `DRACOLoader` ekle.  
3. Görsel karşılaştırma: sıkıştırılmış model eskisiyle aynı görünmeli (ölçü, malzeme, yön).

**Kod:** `kyrox-fair-stand` model yükleme (`loadGltfScene`) + `public/models/`. **Durum:** BACKLOG.

### P1 — Orbit / kalite profili

1. Profiller: `high` / `balanced` (shadow 1024) / `low` (gölge kapalı).  
2. Varsayılan **oto**: ekran kartına göre seçim (zayıf GPU → `low`, diğerleri → `balanced`); kullanıcı seçimi saklanır.  
3. Orbit sırasında `balanced`/`low` için gölge geçici kapalı.  
4. UI: sahne köşesinde Oto / Hızlı / Denge / Yüksek.

**Kod:** `visualQuality` + `scene3d.js`. **Durum:** BACKLOG.

### P2 — `rebuildWall` budama

1. Aynı frame’de debounce (rAF coalesce).  
2. Sonra: renk / glass kısmi update (tam build yok).  
3. Drag snap throttle.

**Kod:** `main.js` (+ sonra scene3d). **Durum:** BACKLOG.

### P3 — Incremental scene graph

Ekle / sil / taşı = tek modül grubu; `disposeWall` yalnız stand tipi / hard reset. Instancing.

**Durum:** BACKLOG (P2 partial sonrası).

### P4 — Ürün / veri (paralel)

| Madde | Not |
|---|---|
| B.3b Z clamp → tip davranışı (DB) | Lamba mount-bottom şimdilik kodda (PR #291) |
| PENDING §A adım 1 | scene_dimensions + yüzey/BOM |
| B.3 mount vs `default_z_cm` | Araştırma |
| CRM lazy Fair Stand route | Bundle; sahne takılmasını çözmez |

---

## 4. 4 haftalık iskelet

| Hafta | Odak | Başarı |
|---:|---|---|
| 1 | P0 ölçüm + P0-M model sıkıştırma | Gerçek açılış süreleri kayıtlı; `public/models/` ≥ %60 küçük |
| 2 | P0 asset cache + eşzamanlılık sınırı | 20 görsel 2. açılış &lt; 1.5 sn |
| 3 | P1 kalite + orbit | oto/low’da akıcı orbit |
| 4 | P2 partial renk/glass | Renkte `buildWall` yok |

---

## 5. Doğrulama kapısı

- [ ] Ölçüm: 0 / 5 / 20 görselli projede ilk ve ikinci açılış süresi kayıtlı  
- [ ] Proje 0 görsel: açılış &lt; 1 sn  
- [ ] Proje 20 görsel: 2. açılış &lt; 1.5 sn; gereksiz asset GET yok  
- [ ] Modeller sıkıştırılmış; görünüm eskisiyle aynı  
- [ ] Kalite profili (oto dahil) UI + unit test  
- [ ] `rebuildWall` aynı frame coalesce  
- [ ] Orbit: oto/balanced’ta akıcı his  
- [ ] 30 modül: hâlâ full yol varsa `buildWall` &lt; 500 ms  
- [x] gzip + statik önbellek canlıda (2026-10-19)

**Ölçek (ops, P4+):** 50 eşzamanlı bootstrap + 20 eşzamanlı proje açma p95 hedefe uyum; asset CDN / rate limit.

---

## 6. Güncelleme kuralı

Performans PR merge → ilgili P* **YAPILDI** + tarih; `REFACTOR.md` bir satır. Rakip gözlemi değişirse yalnız [COMPETITOR_ANALYSIS.md](COMPETITOR_ANALYSIS.md) güncellenir.
