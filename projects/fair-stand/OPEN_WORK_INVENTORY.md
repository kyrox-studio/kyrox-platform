# Kyrox — Açık iş / dağınık not envanteri

> Tarih: 2026-10-20 (okuma sırası: 2026-10-18 tek-kaynak temizliği; 2026-10-20 belge eşitleme)  
> Amaç: Audit, PENDING, roadmap ve stub notlarını tek bakışta görmek.  
> **Kanonik kaynaklar değişmez** — bu dosya özet haritadır; çelişkide aşağıdakiler kazanır.

## Okuma sırası (oturum buradan)

1. **Bu dosya** — özet harita (iş kuyruğu değil)
2. **`kyrox-fair-stand/audit/FINDINGS.md`** — açık/kapalı F-*
3. **`kyrox-fair-stand/docs/refactor/PENDING_ITEM_DECISIONS.md`** — ertelenmiş karar

| Konu | Kanonik dosya |
|---|---|
| Audit bulguları | `kyrox-fair-stand/audit/FINDINGS.md` |
| Item / tavan / şerit kararları | `kyrox-fair-stand/docs/refactor/PENDING_ITEM_DECISIONS.md` |
| Kilit / grup / şablon fazları | `kyrox-platform/projects/fair-stand/GROUP_LOCK_ROADMAP.md` |
| FAZ 4–6 | `kyrox-fair-stand/ROADMAP.md` + `ROADMAP_PHASE_4.md` + bu ağaçta `ROADMAP_PHASE_5_6.md` |
| CRM kuyruk | `kyrox-platform/projects/fair-crm/ROADMAP.md` |

**Bakma (bayat sayaç / ikinci kopya):** `kyrox-fair-stand/SYSTEM_AUDIT_CHECKLIST.md`, `kyrox-fair-stand/audit/FULL_SWEEP_STATE.md`, eski `audit/evidence/*` OPEN satırları. Açık bulgu için yalnız `FINDINGS.md`.

`src/` içinde anlamlı `TODO` / `FIXME` neredeyse yok; iş belgelerde.

---

## 1. Fair Stand — açık audit bulguları

Kaynak: `kyrox-fair-stand/audit/FINDINGS.md`  
Özet: ~11 açık ürün bulgusu, 0 P0, 6 P1, 5 P2 (+ F-034 kabul riski). ~35 CLOSED.

| ID | Sev | Durum | Özet |
|---|---|---|---|
| F-014 | P1 | OPEN / DECISION_REQUIRED | 18 aktif modül (+ foam) için nihai BOM sınıflandırması (recipe / self / decision-required) |
| F-018 | P2 | OPEN | Strip/panel sayısı katalog ↔ state oluşturucuları arasında kopyalı |
| F-021 | P2 | OPEN | Proje `version` var; kanonik doğrulama / migration hattı yok |
| F-030 | P1 | OPEN (kod var) | Final BOM üreteci — `resolveProjectBom` çalışıyor; kapanış kaydı yok |
| F-031 | P1 | OPEN (kod var) | İlişki/köşe BOM — `relationshipBom` var; baza/short-up dışarıda; kapanış yok |
| F-035 | P2 | OPEN | ZIP arşiv sürüm/şema ortak sahibi / migration kaydı yok |
| F-040 | P1 | OPEN | Playwright var; ZIP ve GLB-hata yolu E2E yok |
| F-042 | P1 | OPEN | Deploy SHA pin yok; `origin/HEAD` çekilir; CI deploy kapısı yok |
| F-046 | P2 | OPEN | CI’da ESLint / format kapısı yok (`syntax:check` var) |
| F-047 | P2 | OPEN | Katalog-dışı yeni nesne ekleme kapısı yok (foam contract var) |
| F-048 | P1 | OPEN | Her öğe için zorunlu kanonik BOM politikası yok |
| F-034 | P2 | ACCEPTED_RISK | 9 GLB atıfsız; envanter kısmi — `CLOSED` sayılmaz |

### Hardening (FINDINGS dışı)

Kaynak: `kyrox-fair-stand` release hardening notları (varsa `RELEASE_HARDENING_ROADMAP.md`).

| Madde | Durum | Not |
|---|---|---|
| Selection feedback sızıntısı | Açık | Foam ışık renginde `main.js` mesajı formatter’ı bypass edebiliyor |
| Dead code `syncWallLengthFromSetup` | Açık | Kullanılmayan kapasite hesabı |

---

## 2. Fair Stand — ertelenmiş kararlar (PENDING)

Kaynak: `kyrox-fair-stand/docs/refactor/PENDING_ITEM_DECISIONS.md`

### § A — Tavan / şerit kaldırma sırası (canonical operasyon)

Her item kendi ölçü/BOM’unu taşır; stand tavanı yalnız max zarf.

| Adım | İş | Not |
|---:|---|---|
| 1 | Her item: eksiksiz `scene_dimensions` + yüzey/BOM | Örn. `wall_200_250`; gizli 350 / ×7 sözleşmesi biter — **sıradaki kod adımı** |
| 2 | `designState.strips[]` → item surface slot’ları | Eski proje migration zorunlu |
| 3 | Raf snap → host item anchor | Global seam yok |
| 4 | `fair_stand_item_strip_occupancy` + CRM kaldır | B.4–B.5 |
| 5 | `strip_count` / `strip_height_m` + CHECK kaldır | C.1 |
| 6 | Tavan yalnız max zarf | Ürün mesh/collision tavan fallback okumaz — C.2 |

### § B / C / D / E — açık veya ertelenmiş

| Madde | Durum | Ne |
|---|---|---|
| B.1 `eyeCount` | SİLİNECEK | Gerçek glass shelf yerleşimine geçilecek |
| B.2 `wallGapCm` | KORUNACAK | Foam tüketicisi; grup taşınır |
| B.3 mount vs `default_z_cm` | ARAŞTIRILACAK | İki kolon; biri seçilmeden silinmez |
| B.3b Z clamp → tip davranışı (DB) | BACKLOG — şimdilik kodda | `placement=top` → `clampMountBottomZCm` (taban 0…tavan); floor-relative / overlay ayrı; yeni item için JS case yasak — PENDING B.3b |
| B.4–B.5 strip occupancy | KALDIRILACAK | § A.4 sonrası |
| B.7 gizli-350 seed SKU | BACKLOG | `wall_200_100` vb. ayrı yükseklik SKU’ları |
| B.7 BOM↔leaf soft-warning | KARAR; kod yok | Admin uyarı; auto-fix yok |
| B.8 `variant` | KORUNACAK (şimdilik) | Occupancy kalkınca yalnız kimlik |
| B.9 Item snap | UYGULAMA | Tip/kural CRUD + motor (kısmen canlı) |
| B.10 `is_render` | KARAR; mesh kısmi | Child çizim hedefi tam değil |
| C.1–C.2 strip / tavan fallback | KALDIRILACAK | § A.5–6 |
| D.1 Color/Image/Lighting mekanizmaları | YAPILMADI | Config şeması yok |
| E.1 proje–org kuralları | MEVCUT; genişletme | Paylaşım / varsayılan org ayrı karar |
| E.2 stand `frame_width_cm` | KALDIRILACAK (karar bekliyor) | 2026-10-21'den beri sahnede okunmaz (kesit = reçete profil Item sahne ölçüsü). Kolonun ve CRM Temel Ayarlar alanının kaldırılması migration + CRM işi; CRM'e dokunmadan önce kullanıcıya sorulur |
| E.6 T'de gövde köşe paneli | KARAR BEKLİYOR | Bugünkü kural: T'de gövde duvarı köşe paneli almaz (`payCorner: false`), arka yüz de bu kuralı izler. Kullanıcının T görseli gövdenin kol tarafını da köşe gösteriyor; değişirse ön yüz BOM'u da değişir |
| E.4 raf ayağı ölçüsü | VERİ | `shelf_leg` DB'de ölçüsüz; ayak sahnede çizilmez, yalnız BOM'da. Ölçü girilirse çizim ayrı iş |
| E.5 0072 canlıya | KAPANDI | `0072_shelf_rows` 2026-10-21 deploy ile canlıda |
| E.7 0073–0075 canlıya | BEKLİYOR | `0073_move_snap_10cm` (bütün taşıma adımları 10 cm), `0074_showcase_brochure` (ayaklı vitrin, broşürlük) ve `0075_glass_bar_table` (bar masası) yerelde; deploy kullanıcıya sorulur |
| E.8 vitrin / broşürlük fiyatı | VERİ | `floor_showcase_100` ve `brochure_stand` BOM'a 1 adet yazılır; `fair_stand_cost_items` fiyatı yok, kullanıcıdan bekleniyor |
| E.3 dikme sahne ölçüsü | VERİ | `upright_*` sahne kesiti 8 × 8, profiller 10 × 10; sahne dikmeyi profil kesitiyle çizer. Dikme kayıtları profil ile eşitlenmeli mi kararı |

**Not:** B.6, B.11–B.15, C.3–C.5 gibi birçok madde **kodda bitti**; PENDING’de “KOD / KAPANDI” diye durur, tekrar açılmaz (§ F).

---

## 3. Fair Stand — yol haritası / faz

**Sıradaki ilk iş — Varyant aileleri admin'den (2026-10-25, plan):** [Plan](VARYANT_AILELERI_ADMIN_PLANI.md). Aile tanımı (panel başlığı, kart adı, etiket, sıra, eş aile) bugün `src/catalogWidthFamily.js` içinde; DB tablosu + `/admin/variant-families` + CRM ekranı ile admin'e taşınacak. Yetki: Item admin izinleri. Kod yok; üç açık karar planda.

**Görsel seçimi — iki yol (2026-10-25, not; yöntem açık):** Özellikler panelinde görsel iki yoldan seçilecek: (1) mevcut görsel arşivinden (assets), (2) yeni görsel yükleyerek. İki yolun panelde nasıl sunulacağı henüz kararlaştırılmadı; tasarım konuşulacak.

**Görsel arşivinde çift tık uygular (2026-10-25, iş):** Görsel arşivindeki bir görsele çift tıklanınca seçili yüzeye doğrudan uygulansın.

**Varyant tek kart + Özellikler paneli (2026-10-25, yerel):** 0086 ile Vitrin, Raf, TV (85" dahil), Video Wall, Uzun Saksı tek kart; varyant panelde canlı seçilir, öğe snaplı kenara göre büyür, itilen öğelerin üstündeki TV/logo taşınır. Üretim Listesi üst banda döndü, Modül Ekle düğmesi kalktı, sahnede silme onaysız (Geri al; geçmiş revizyonda da). Ayrıntı: [Proje durumu](PROJECT_STATUS.md).

**Ortak kısa dikme düzeltmesi (2026-10-24, yerel):** Taşıyıcı aynı birleşim noktasını ve bandı karşılıyorsa kalan ortak 49,5 / 99 kısa dikme de düşülür. Eski iki 99 beklentisi ve sözleşme kapsamı güncellendi; genişletilmiş regresyon ile üretim listesi/kayıt-açma testleri eklendi. Son tam doğrulama: 1.509 birim ve 232/232 Chromium E2E, build ve contract:verify başarılı. Kod PR #338 main’e birleşti (a098e723), yerel taban eşitlendi. Ayrıntı: [Proje durumu](PROJECT_STATUS.md).

**Seçime Göre Özellik Paneli (2026-10-24, yerel):** [Plan](SECIM_OZELLIK_PANELI_YOL_HARITASI.md). İşlevsel adım uygulandı: öğeye tek tık, sağ tık, ⋯ ve uzun basma sahnenin sağındaki aynı paneli açar; mevcut sağ tık işlemleri, renk / görsel, çoklu panel / çoklu modül / grup bağlamı paneldedir; araç çubuğu korunur. ⋯ kullanıcı kararıyla kaldırıldı. 0084 katalog kartı ve 0085 vitrin renk yeteneği yerel DB’ye uygulandı; dört sunta renk/kayıt akışı ve açıklama taşması doğrulandı. Son tam koşu: 1.515 birim, 198 backend, 242/242 E2E; build ve contract:verify başarılı. Kod PR #340 ile gönderildi; auto-merge açık, GitHub CI bekleniyor. Açık: 3. adımın kalan görsel düzeni. Ayrıntı: [Proje durumu](PROJECT_STATUS.md).
### Kullanılabilirlik — 2026-10-24 takip

Açık maddelerin tek kaynağı: [Kullanılabilirlik Yol Haritası](KULLANILABILIRLIK_YOL_HARITASI.md). Güncel uygulama ve doğrulama: [PROJECT_STATUS.md](PROJECT_STATUS.md#2026-10-23--basit--gelişmiş-mod-ve-ayarlar). Basit/Gelişmiş seçici, Ayarlar, toplu kilit, ölçü tercihi, taşıma ızgarası, metreyle kurulum ve kılavuz tamamlandı; ilk paket PR #334 ile birleşti; ek paket PR #335 GitHub CI başarılı olduktan sonra otomatik birleşti; canlı deploy yapılmadı.

2026-10-24 yerel ek: tek öğede mavi, grup/çoklu seçimde tek ortak turuncu kesik çizgili sınır uygulandı ve doğrulandı; henüz gönderilmedi.

Açık: grup dış sınır ölçüleri; Stand / Ürün ekle / Tasarım menü düzeni (kullanıcı tarafından ertelendi); bütün yapı düzenleme ve teknik parçaların otomatik çözülmesi. Bunlar yeni audit bulgusu veya tamamlanmış özellik olarak kaydedilmez.

### Grup / kilit / şablon — `GROUP_LOCK_ROADMAP.md`

| Faz | Durum |
|---|---|
| 1 Kilit | Uygulandı (PR #278) |
| 2 Geri al / ileri al | Uygulandı (PR #281) |
| 3 Grup | Uygulandı (PR #284) |
| **3b Grup aç / kapat** (3ds Max gibi) | **Karar alındı (2026-10-18); kod yok** — tık=grup, kapalıda özellik yok, açıkta tek taşıma OK, persist yok, kilitliyken açılmaz |
| 4 Grup döndürme | Uygulandı (PR #285) |
| 5 Kurumsal şablon | Uygulandı (PR #286, migration 0067); görsel/Lightbox/Mesh **taşınmaz** (kalıcı) |
| 6 Opsiyonel “raf ev sahibi duvar” | Plan |

### Son teslimler (2026-10-19 → 22)

| İş | PR | Durum |
|---|---|---|
| Duvar yüksekliği 250–500 cm, bant reçetesi (migration `0068`–`0071`) | #307 | Merge + deploy (canlı DB `0071`) |
| Tablet kullanımı: seçim çubuğu, uzun basma, sade menü + Gelişmiş, Çoklu panel/modül, dokunarak ekleme | #308 | Merge + deploy |
| Otomatik kayıt kapalıyken yalnız Kaydet; Kopyala (Çoğalt Sağ/Sol kaldırıldı); çift tık modül seçimi, Ctrl seçilenin çoğulu, Shift+tık kaldırıldı | #309 | Merge |
| Otomatik alınlık: sihirbaz "Alınlık olsun → Tek / Çift sıra" + sol menü listesi; açık kenarlara wall-short, açık köşelere dikme, 5 m'yi aşan kenarda eşit aralıklı ara dikme (`autoFascia.js`); short (wall-short-1/2) birleşimlerinde arka yüz paneli BOM'da tam boy duvar kuralıyla köşe paneline döner (`relationshipBom.js`) | #320 | Merge |
| Sahne ışığı: tepe spotu + ışık havuzu, ters ortam ışığı (alınlık dış yüzü aydınlık, ek ışık yok), üst bantta **Sahne Işığı** düğmesi (varsayılan açık, oturumluk; CRM'de de); otomatik alınlık gövdesi içeriden açık renk; perspektif kamera dinamik `near` (büyük standda uzak panel z-fighting) | #321 | Merge |
| Profil kartı ve boy seçimi: tek kart `profile_289`, bırakırken boy penceresi; gerçek → sahne 41,5→50 … 289→300, 388→400, 487→500 (migration `0078`, `profile_289_scene` → `profile_289`) | #322 | Merge |

| Kısa panel tek kart: katalogda Tek Sıra / Çift Sıra için birer kart (`wall_100_short_1/2`); adlar DB'de "Panel Tek Sıra W" / "Panel Çift Sıra W", "Profil 41,5", "Dikme 346,5"; sağ tık başlığı sahnedeki öğenin adını yazar; bırakırken genişlik 50/100/150/200 konum hesabından önce sorulur (`wallShortWidth.js`, migration `0079_wall_short_cards`); profil işinin (#322, merge) üstüne kurulu | #323 | Merge |
| Alınlık profilleri tek profil: uç uca kısa panellerin (Tek/Çift Sıra) üst/alt profili BOM'da taşıyıcıdan taşıyıcıya (dikme / tam boy duvar ucu) tek parça; tam boy yoksa en uzun uygun profil + kalan parça (`relationshipBom.js` `applyShortRunProfiles`); kısa panelin bağlantı çizgisine konan profil o raydaki panel profilini düşürür (`applyProfileRail`) | #325 | Merge + deploy |
| Stand API: Core yetki sorgusu oturumu tanımazsa 401, Core'a ulaşılamazsa 503 (önceden yakalanmayan hata → 500; CRM istemcisi 401'de yenileme/girişe yönlenir) | #326 | Merge |
| Stand ölü kod taraması: yetim `cornerPlacement.js`, çağrılmayan frontend/backend yardımcıları ve importlar, `alembic.ini` `path_separator`; davranış değişmez | #327 | Merge |
| Profil 346,5 (`0081`), katalogda sahne profili yok; ölçü soran kartlar ölçüsüz başlık ("Profil", "Dikme", "Panel Tek Sıra"); stand kapasite uyarısı kaldırıldı; E2E kendi sunucusunu açar (#329) | #330 | Merge + deploy |
| Katalog tekli ekleme (`0082_panel_separator_cards`): Panel ve Separatör tek kart, bırakırken genişlik (Separatörde "Sarmaşıklı" çentiği); katalog penceresi sol sekme + tekli seçim; sağ tık tek "Ekle…" (sağ → sol → önü → müsait yer; duvar üstü/tepe öğeler tıklanan modülün üstüne); kopyalamada "… hafızaya alındı" bildirimi; Banko / Köşe Banko / Baza tek kart (`0083`); bildirimler en fazla 2; katalog penceresi sabit yükseklik; yeni proje kaydında adres `/stand-projects/{id}`; "?" üst bantta, Persp/Ortho küp kutusunda; alınlık dikmesi birleşime ortalı. Dal `feat/katalog-tekli-ekle` | — | PR + auto-merge |

Açık kalan: açık standın duvar yüksekliğini değiştirmek (`PENDING_ITEM_DECISIONS.md` B.16). Ertelendi (2026-10-23): `z_frame` / `z_max` dikey sınır davranışının DB'ye taşınması (`modulePlacement.js` TODO(DB)); `scene3d.js` ve `main.js`'in parçalanması; yalnız testlerde kullanılan ~35 export'un gözden geçirilmesi; `normalizeModuleItemState`'e dikme dalı (içe aktarılan eski projelerde dikme sahne ölçüsü). `profile_346_5` maliyet kaydı admin ekranından girilecek.

Gelecek (karar 2026-10-20): **hazır stand tasarımları** — masa, sandalye vb. dolu hazır standlarla hızlı başlangıç (yeni proje sihirbazına bağlanabilir). Grup şablonu bu iş için yeterli görülmedi.

### Master roadmap — `kyrox-fair-stand/ROADMAP.md`

| Faz | Durum | Özet |
|---|---|---|
| 1–3 | Kapandı | Temel / yerleşim / görsel |
| **4** | **Aktif** | Recipe var; instance Raw BOM, parametric engine, wizard, connection graph, custom library çoğunlukla açık |
| 5 | Planlandı | Final BOM / sahne malzeme listesi / Excel doğrulama |
| 6 | Planlandı | Maliyet / Fair CRM entegrasyonu |

### Gelecek (faz numarası yok veya ayrı)

| Belge | İçerik |
|---|---|
| `PRODUCT_FUTURE.md` | PDF teklif, montaj sırası, depo/paketleme, stok, proje onay/paylaşım |
| `RENDER_FUTURE_BACKLOG.md` | HDRI, AO, PBR, post-processing |
| Platform `PERFORMANCE_ROADMAP.md` | Teşhis + P0–P4 hız planı (P0, P1 yapıldı; P2/P3 büyük ölçüde; P4 açık) |
| Platform `COMPETITOR_ANALYSIS.md` | BoothDesign / StandBuilder gözlem (analiz; kod yok) |
| Platform `ROADMAP.md` (fair-stand) | Tenant katalog, cost-catalog slug, admin form↔DB audit |

---

## 4. Fair CRM / platform

Kaynak: `kyrox-platform/projects/fair-crm/ROADMAP.md` + `backlog/*`

| Madde | Durum | Not |
|---|---|---|
| TOBB final workflow | Merge OK | **Prod deploy yetkisi bekliyor** |
| P0.1 tenant isolation | DONE | Sertifika baseline |
| P0.2 identity / onboarding | DONE | |
| SaaS P0.3+ | Plan / gated | MFA, rate limit, session vb. |
| Mail “İptal” UI | OPEN stub | `MailOperationsPage` — henüz bağlanmadı |
| MailerSend kalan | Backlog | Webhook, status sync, unsubscribe |
| Email iletişim tercihleri | Backlog | |
| Mail send ops Faz 2 | Backlog | Worker, rate limit, retry |
| Operation pause vs cancel | Planlandı | |
| Admin form ghost alanları | OPEN | `ADMIN_FORM_DB_AUDIT.md` — ghost alanlar fiilen kullanılmıyor |
| Permission UI consistency | Aktif kalite | UI gating ↔ backend |
| DB index / perf | Planning | `INFRASTRUCTURE_PERFORMANCE_ROADMAP.md` |

---

## 5. Doküman / hijyen borcu

| Madde | Risk |
|---|---|
| `SYSTEM_AUDIT_CHECKLIST.md` ikinci kopya | Yanıltıcı OPEN sayaçları |
| `audit/FULL_SWEEP_STATE.md` | Tarihî mühür; güncel değil |
| Eski `audit/evidence/*` OPEN satırları | Kapanmış bulguları OPEN gösterebilir |
| `GROUP_LOCK` gövde metinleri | Bazı “açık kalan” cümleleri header tablodan geride kalabilir — header’a güven |
| GitHub eski dallar (`ROG`, `RefactorItem`, `Version2`) | Aktif iş değil; ürün kodu değil (F-044 kapsam dışı) |

---

## 6. Öncelik önerisi (sıkıştırılmış)

1. **Ürün kararı:** F-014 / F-048 + PENDING § A (tavan/şerit) + B.7 soft-warning  
2. **Teknik kapanış:** F-030 / F-031 kapanış kanıtı; F-040 ZIP/GLB e2e; F-018 / F-021 / F-035; F-046 ESLint; F-042 deploy  
3. **Editör UX:** Grup 3b kararlı — uygulama PR’si (şablon görsel taşıma yok / iptal)
4. **FAZ 4 checklist:** Raw BOM instance, parametric, connection graph  
5. **CRM:** mail cancel stub + MailerSend/preferences; TOBB prod deploy yetkisi  
6. **Doc:** checklist / FULL_SWEEP’i kanonik FINDINGS ile karıştırmamak  

---

## 7. Bu dosyanın güncellenmesi

- Yeni açık bulgu → önce `FINDINGS.md`, sonra bu özet.  
- Yeni ertelenmiş karar → önce `PENDING_ITEM_DECISIONS.md`.  
- Faz durumu → ilgili ROADMAP.  
- Bu dosyayı tek başına “iş kuyruğu” yapma; çelişkide kanonik dosya kazanır.
