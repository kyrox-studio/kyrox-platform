# Web ve Ürünler — Tarayıcı Depolaması ve Dış Servisler

**Tarih:** 26 Ekim 2026. **Sahip:** Codex; yayına hazırlık 12. madde.  
**Amaç:** Gizlilik içerikleri ve sonraki tercih/ölçümleme işleri için mevcut teknik kullanımı kaydetmek.

## İncelemenin sınırı

Ana sayfa, iki ürün tanıtımı, CRM giriş ve WordPress giriş ekranı temiz Chromium oturumlarında incelendi. Form gönderilmedi, hesaba giriş yapılmadı, canlı ekran paylaşımı başlatılmadı. Oturum sonrası kayıtlar kaynak koduyla sınıflandırıldı; bunların tamamı canlı oturumda gözlenmiş sayılmaz. Saklanan değerler, tokenlar, parolalar veya gerçek kullanıcı oturumu okunmadı/kaydedilmedi.

Canlı tarayıcı kanıtı: [anonim ziyaret depolama ve ağ kaydı](evidence/web-storage-20261026.json). İncelenen sunucu kaynakları: Web `0d279cb`, CRM `ae5dbc7d`, Stand `3d212ed5`. Yerel uygulama dalındaki önceki düzenlemeler canlıya uygulanmış kabul edilmez.

## Canlı anonim ziyaret sonucu

| Ekran | Çerez | Tarayıcı depolaması | Otomatik dış istek |
|---|---|---|---|
| `kyrox.studio/` | Gözlenmedi | localStorage, sessionStorage, IndexedDB, CacheStorage kaydı gözlenmedi | Gözlenmedi; kaynaklar aynı origin |
| `/fuar-crm/` tanıtımı | Gözlenmedi | Kayıt gözlenmedi | Gözlenmedi; fontlar yerel |
| `/fuar-stand/` tanıtımı | Gözlenmedi | `kyrox-stand-scenes-v1` CacheStorage; sürümlü Elikon GLB dosyası | Gözlenmedi; 3D motor/model/fontlar yerel |
| `fuar.kyrox.studio/login` | Giriş yapmadan gözlenmedi | İlk ziyarette kayıt gözlenmedi | `fonts.googleapis.com`, `fonts.gstatic.com` |
| `/wp-login.php` | `wordpress_test_cookie`; oturumluk, Secure, HttpOnly, SameSite=Lax | Yalnız giriş sayfası incelendi | Yönetici ekranı; halka açık tanıtım akışıyla karıştırılmaz |

Bu akışlarda Google Analytics, Google Tag Manager, Meta Pixel, Hotjar veya benzeri analiz/reklam isteği gözlenmedi. Sonradan eklenen veya giriş arkasındaki davranış için yeniden inceleme gerekir.

## Depolama envanteri

Amaç sütunu teknik kullanımdır; tercih/izin uygulamasının kararı 13–14. maddelerde gerçek kullanım üzerinden verilecek.

| Kayıt | Yer / tetikleyici | İçerik ve amaç | Süre / temizleme | Kanıt |
|---|---|---|---|---|
| `kyrox-stand-scenes-v1` | `kyrox.studio`, Stand tanıtımının otomatik açılışı | Yalnız herkese açık 3D model; yeniden indirmeyi önler | Sabit gün süresi yok. Yeni model sürümü başarılı yüklenince aynı yolun eski sürümü temizlenir; kullanıcı/tarayıcı da silebilir | Canlı gözlendi; `kyrox-web/tasarim/stand-v3/viewer.js` |
| `fair_crm_refresh_token` | CRM, başarılı giriş/yenileme | Oturum yenileme tokenı; JavaScript'ten okunmayan çerez | Sunucuda 15 gün; yenilemede yeniden atanabilir, çıkışta silinir. Host cookie, path `/api/v1/auth`, HttpOnly, Secure=true, SameSite=Lax | CRM `backend/app/modules/auth/api/cookies.py`; production ayarları yalnız izinli alanlar okunarak doğrulandı; canlı giriş yapılmadı |
| `fair-crm.auth.session` | CRM, giriş sonrası localStorage | Access token, organizasyon bilgisi, e-posta ve yetki listesi | localStorage kaydında otomatik süre sonu yok; tokenın geçerliliği ayrıdır. Çıkış/oturum temizleme kaydı kaldırır. Refresh token yeni kayda yazılmaz | CRM `frontend/src/auth/session.ts`; giriş sonrası tarayıcı kaydı incelenmedi |
| `fair-crm.sidebar.collapsed` | CRM, menü tercihi | Menünün dar/açık olması | Kodda sabit TTL yok; kullanıcı/tarayıcı temizleyene veya yeni tercihe kadar | CRM `components/layout/AppLayout.tsx`, `hooks/usePersistedCollapsed.ts` |
| `fair-stand-items-flash` | CRM katalog yönetimi, sessionStorage | Sayfalar arasında tek seferlik işlem mesajı | Okununca silinir; sessionStorage oturumuyla sınırlı | CRM `pages/FairStandItemsAdminPage.tsx` |
| `fair-stand-configurator` v2 | Stand editörü, IndexedDB | `projects` ve `image-assets`: yerel proje/görsel önbelleği | Sabit TTL yok; uygulamanın proje/görsel silme yolları veya tarayıcı temizliği | Stand `src/configuratorDb.js`, `projectStore.js`, `assetStore.js`; tanıtım görüntüleyicisinde kullanılmaz |
| `fair-stand.asset-sync.v1` | Stand editörü, localStorage | Proje/görsellerin senkronizasyon durumu | Sabit TTL yok; senkronizasyon/silme akışları günceller veya kaldırır | Stand `src/projectRemote.js` |
| `fairStandTipsSeen` | Stand editörü, ipuçları | İpuçlarının görüldüğü bilgisi | Sabit TTL yok; kullanıcı/tarayıcı temizliği | Stand `src/onboardingTips.js` |
| `fairStand.autosaveMinutes` | Stand editörü, otomatik kayıt seçimi | Dakika tercihi veya kapalı seçeneği | Sabit TTL yok; tercih değişikliği/temizlik | Stand `src/autosaveSettings.js` |
| `fairStand.profileLengthItemKey`, `fairStand.uprightHeightItemKey` | Stand editörü, “Bir daha sorma” seçimi | Profil/dikme için hatırlanan katalog öğesi | Tercih kaldırılınca silinir; sabit TTL yok | Stand `src/profileLength.js`, `uprightHeight.js`, `itemSizeDialog.js` |
| `fairStandTiming` | Stand editöründe elle açılan tanılama | Açılış ölçümünü etkinleştirme bayrağı | Varsayılan izleme servisi değildir; elle kaldırılana kadar | Stand `src/projectOpenTiming.js` |
| WordPress yönetici oturum/ayar çerezleri | Yalnız yetkili WordPress kullanımı | Yönetici kimliği ve panel tercihleri | Giriş sonrası kapsam/süre bu incelemede ölçülmedi | Ayrı yönetici oturumu incelemesi gerekirse yapılacak; normal ziyaretçide varmış gibi raporlanmaz |
| Standart HTTP önbelleği | CSS, JS, font, görsel dosyaları | Statik kaynakların tekrar kullanımını sağlar | Yanıt başlıkları ve tarayıcı politikasına bağlı | Özel ziyaretçi kimliği veya reklam kaydı değildir |

## Dış servis envanteri

| Servis | Ne zaman kullanılır? | Aktarılan işlev / veri kapsamı | Durum |
|---|---|---|---|
| Google Fonts (`fonts.googleapis.com`, `fonts.gstatic.com`) | CRM girişinde otomatik | Font CSS/dosya istekleri ziyaretçinin tarayıcısından gider | Canlı gözlendi. Tanıtım sayfalarında yerel font kullanılıyor; CRM fontlarını yerelleştirme ayrı uygulama işi olarak açık |
| WhatsApp / `wa.me` | Kullanıcı bağlantıya tıklayınca | Önceden doldurulmuş mesaj; iş fişinde yazılan talep/iletişim bilgileri WhatsApp bağlantısına eklenebilir | Gömülü sohbet/piksel yok; incelemede tıklanmadı veya mesaj gönderilmedi |
| Instagram ve LinkedIn | Sosyal bağlantıya tıklanınca | Kullanıcı ilgili profil/sayfaya gider | Gömülü akış/SDK isteği gözlenmedi |
| Google Haritalar | Adres bağlantısına tıklanınca | Stüdyo adresi araması | Gömülü harita veya otomatik Maps isteği gözlenmedi |
| `mail.kurumsaleposta.com:465` SMTP | Web iş fişi gönderildikten sonra, sunucudan | Talep ve iletişim bilgisinin stüdyo e-postasına bildirimi | Canlı SMTP host/port doğrulandı. Şifre okunmadı, deneme talebi/e-postası gönderilmedi |
| Google STUN `stun:stun.l.google.com:19302` | Stand uygulamasında canlı ekran paylaşımı başlatılınca | WebRTC bağlantı kurulumu; tanıtım GLB görüntüleyicisinin parçası değil | Production yapılandırmasında var; paylaşım başlatılarak ağ ölçümü yapılmadı. TURN URL'si yapılandırılmamış |
| WordPress, CRM, Stand ve Core API'leri | İlgili sayfa/özellik kullanıldığında | Birinci taraf servisler; CRM/Core entegrasyonu ürün işlemlerini yürütür | Dış reklam sağlayıcısı olarak sınıflandırılmaz; erişim/organizasyon kontrollerinin kaynağı ortak güvenlik programıdır |

## Form, sunucu kayıtları ve sonraki işler

- İş fişi `ne` ve `iletisim` alanlarını WordPress yönetiminde özel talep kaydı olarak saklar; SMTP başarısız olsa da kayıt kalır. Bu bir tarayıcı çerezi değildir.
- Bot sınırı aynı IP'nin özetinden türetilen geçici anahtarla 600 saniyelik sayaç tutar. Talep kaydı veya nginx/uygulama günlüklerinin saklama süresi bu 600 saniyeden türetilmez.
- Talep kayıtları, sunucu günlükleri ve yedekler için saklama/silme süreleri henüz burada doğrulanmış veya kararlaştırılmış değildir. 13. maddede metin hazırlanırken açık karar olarak ele alınacak.
- Analiz ve reklam pikseli henüz kurulmadı. 15. maddede seçilecek sağlayıcı, olaylar, aktarılacak alanlar ve saklama süreleri bu envantere eklenmeden “envanter güncel” sayılmaz.
- Oturumlu CRM/Stand işlemleri ve canlı paylaşımın davranış doğrulaması yapılırsa bu kayda kanıt eklenir. Bu belge yeni bir güvenlik bulgu defteri oluşturmaz: [ortak güvenlik programı](../standards/security/SECURITY_PROGRAM.md).

Takip: [Web ve Ürünler — Yayına Hazırlık](WEB_LAUNCH_CHECKLIST.md).
