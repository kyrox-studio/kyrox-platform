# Web ve Ürünler — Tarayıcı Depolaması ve Dış Servisler

**Tarih:** 26 Ekim 2026. **Sahip:** Codex; yayına hazırlık 12. madde.  
**Amaç:** Gizlilik içerikleri ve sonraki tercih/ölçümleme işleri için mevcut teknik kullanımı kaydetmek.

**Sonraki form doğrulaması:** 26 Ekim'de [yayına hazırlık 16. maddesi](WEB_LAUNCH_CHECKLIST.md#tamamlanan-işler) kapsamında tek gerçek web talebi, özel kayıt ve alıcının INBOX teslimi doğrulandı; kontrollü hata yolları geçti ve deneme kayıtları temizlendi. Aşağıdaki ilk anonim incelemede “form gönderilmedi” ifadesi o incelemenin sınırıdır. Form/SMTP kontrolü artık bu kanıta sahiptir; WhatsApp'a gerçek mesaj gönderimi veya diğer uygulama/veri yaşam döngüsü işleri bu sonuçla kapanmaz. Kullanıcının şu an reklamı olmadığı için reklam ölçümlemesi beklemededir.

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
| `kyrox-stand-scenes-v1` | `kyrox.studio`, Stand tanıtımında model önbelleğine izin verilmişse | Yalnız herkese açık 3D model; yeniden indirmeyi önler | Sabit model TTL'si yok. İzin geri alındığında temizlenir; tercih süresi dolmuşsa sonraki sayfa açılışında kaldırılır. Yeni model sürümü başarılı yüklenince aynı yolun eskisi temizlenir | İlk envanterde otomatikti; aşağıdaki Web PR #14 ile izinli hale getirildi. `kyrox-web/tasarim/stand-v3/viewer.js` |
| `kyrox.storage-preference.v1` | `kyrox.studio`, ziyaretçinin izin/ret seçimi, localStorage | Yalnız sürüm, model önbelleği seçimi ve süre sonu; kullanıcı kimliği/token/reklam profili yok | Seçimden sonra 180 gün geçerli; süre dolduğunda yeniden seçim, varsayılan kapalı. Alt bilgiden değiştirilebilir. Depolama engelliyse yalnız sayfa belleği | Aşağıdaki canlı tercih kontrolü; `kyrox-web/tema/kyrox/assets/js/depolama.js` |
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

## Açık eksikler ve kapanış koşulları — 26 Ekim 2026

Kullanıcı kararı: Web, CRM ve Stand birlikte uçtan uca incelenecek. Kaynak kodu envanteri veya tek bir başarılı giriş, tüm veri akışlarının kabulü değildir. Aşağıdaki işler açık kalır; sahip Codex'tir. Doğrulanmış güvenlik açığı bulunursa ayrı bir bulgu defteri açılmaz, ortak güvenlik programına kaydedilir.

| Açık iş | Kapanış için gereken kanıt |
|---|---|
| CRM oturum yaşam döngüsü | Gerçek Core/CRM kimliğiyle giriş, yenileme, süre dolumu, çıkış ve iptal; önce/sonra cookie ve tarayıcı depolaması, eski oturumun reddi. Token değerleri kanıta yazılmaz. |
| CRM verisi ve yetkiler | Yetkili başarı yanında yetkisiz/başka organizasyon erişiminin reddi; gerçek API ve ekranda sonuç. Yerel başarılı Standlar giriş kontrolü bu kapsamın tamamını kapatmaz. |
| Stand proje/görsel verisi | Oluşturma, kaydetme, yeniden açma, senkronizasyon, silme ve hata durumlarında IndexedDB/localStorage ile sunucu kayıtlarının eşliği; başka organizasyon ve yetkisiz erişim negatifleri. |
| Canlı Stand paylaşımı | Paylaşımı başlatma/izleme/sonlandırma, bağlantı süresi ve iptali; gerçek STUN/WebRTC/WebSocket hedefleri ve veri kapsamı. Tanıtım GLB önbelleği bu akışın kanıtı değildir. |
| Form ve dış aktarım | Form → özel WordPress kaydı → SMTP teslimi ve hata yolu; WhatsApp'a tıklamayla aktarılan alanlar; Google Fonts ve diğer dış isteklerin gerekli/isteğe bağlı tetiklenmesi. Kontrollü deneme ve kayıt temizliği ayrıca planlanır. |
| Saklama ve silme | Talep, uygulama kaydı, günlük, yedek ve yerel önbellek için gerçek süre/ölçüt, sorumlu ve silme davranışı; sırf metne süre yazılarak tamamlanmaz. |
| Gizlilik ve kullanım metinleri | Web ziyaretçisi ve CRM/Stand kullanıcısı için gerçek amaç, veri, alıcı, hukuki sebep ve başvuru yolu eşleştirmesi; form/giriş/ürün ekranlarından erişim. WordPress yönetici oturumu ayrıca sınıflandırılır. |
| Tercihler ve ileride analiz/reklam | Gerekli olmayan akışların tercih öncesi çalışmaması; kabul/ret/geri alma ve sonraki ziyaret doğrulaması. Yeni piksel/servis eklenirse envanter ve metinler birlikte güncellenir. |
| Son hukuki inceleme | Gerçek işletme/veri sorumlusu kimliği, işleyen/sorumlu rolleri, aktarım ve saklama kararları üzerinden ayrıca yapılacak. Teknik kontroller hukuki uygunluk onayı olarak raporlanmaz. |

**Metin yönetimi için kullanıcı kararı:** Şimdilik marka adı **KYROX STUDIO** kullanılacak. Bu, resmî veri sorumlusu unvanının doğrulandığı anlamına gelmez; gerçek kimlikle güncelleme açık iştir. Belgeler WordPress panelinden düzenlenebilir kalacak, tema güncellemesi yönetici düzenlemelerini ezmeyecek. Genel bilgilendirme sayfaları aşağıdaki teslimle yayımlandı; tam hukuki aydınlatma ve veri yaşam döngüsü kabulü açık kalır.

## Hemen uygulanabilen paket — 26 Ekim 2026

- **Web yayımlandı:** [Gizlilik ve veri kullanımı](https://kyrox.studio/gizlilik/), [Kullanım koşulları](https://kyrox.studio/kullanim-kosullari/), [Çerezler ve depolama](https://kyrox.studio/cerezler/). WordPress Sayfalar panelindeki kayıtlar sırasıyla **18, 19, 20**; metinler veritabanında düzenlenir, tema koduna gömülmez. Mevcut taslak `privacy-policy` değiştirilmedi. Yeni gizlilik sayfası WordPress gizlilik sayfası olarak seçildi. Kurulum yeniden çalıştırıldığında mevcut üç içeriğin korunduğu doğrulandı.
- **Erişim:** Web/CRM tanıtım alt bilgisi, bağımsız Stand V3 alt bilgisi ve iş fişinin yanında veri kullanım açıklaması. Yalnız yayımlanmış, parolasız sayfalar bağlanır. Rıza kutusu veya kullanımla rıza varsayımı eklenmedi. Formun veri kaydı, SMTP veya WhatsApp davranışı değiştirilmedi.
- **Web kanıtı:** [Web PR #13](https://github.com/kyrox-studio/kyrox-web/pull/13), üç CI kontrolü yeşil; sunucuda `8701c919c72530b9979d20ee22d807ee3ae9f408`. Öncesinde `/opt/kyrox/backups/web-information-20261026` altında DB/tema yedeği alındı. Yayında kısıtlı umask nedeniyle yeni tema dosyalarında okuma izni hatası oluştu; yalnız halka açık tema ağacının dizin/dosya izinleri 755/644 yapılarak düzeltildi, yayın tekrar çalıştırıldı. Altı canlı sayfa HTTP 200, üç alt bilgi bağlantısı, form yanındaki açıklama, ürün bölüm hedefleri, 3D sahnenin açılması ve mobil taşma olmaması Chromium'da doğrulandı; JS hatası yok. Ürün SEO/canonical/paylaşım/sitemap/llms kontrolleri yeniden geçti.
- **CRM kodu:** [PR #370](https://github.com/kyrox-studio/kyrox-fair-crm/pull/370): giriş/kayıt/kurtarma ve uygulama kabuğuna bilgi bağlantıları, Google Fonts yerine aynı origin Inter fontları ve SIL OFL lisansı. 676 frontend testi, entegre Stand derlemesi ve gerçek yerel Core/CRM girişiyle tarayıcı kontrolü geçti. Yerel WOFF2 başlığı/HTTP yanıtı, Google Fonts isteğinin olmaması ve üç bilgi bağlantısı doğrulandı. Dört GitHub kontrolü yeşil; auto-merge açıldı. Canlı uygulama dağıtımı bu teslimde yapılmadı.
- **Stand kodu:** [PR #349](https://github.com/kyrox-studio/kyrox-fair-stand/pull/349): mevcut yardım kılavuzuna üç bilgi bağlantısı, aynı origin Inter ve SIL OFL. 1546 Node testi, build, etki sözleşmesi ve gerçek Chromium kılavuz/font kontrolü geçti. Tam GitHub E2E koşusu sürerken auto-merge açıldı; canlı uygulama dağıtımı yapılmadı. CRM/Stand font dosyaları aynı ad/içerikte olduğundan birleşik derlemenin kopyalama yoluyla uyumludur.
- **Kapsam sınırı:** Yukarıdaki ilk envanter canlı inceleme tarihinin kaydıdır. CRM/Stand kaynak değişikliği canlıya çıkmadan ilk envanterdeki Google Fonts sonucu silinmez; dağıtımdan sonra canlı ağ tekrar ölçülür. Kimlik, yetki, organizasyon sınırı, proje/BOM ve veri saklama davranışı değiştirilmedi.

**Geri dönülecek uzun/karar bekleyen işler:** Yukarıdaki açık eksik tablosunun tamamı geçerlidir. Öncelikle (1) CRM/Stand kontrollü dağıtım ve canlı erişim/font kontrolü, (2) resmî veri sorumlusu kimliği ve faaliyete özel hukuki sebeplerle ayrıntılı aydınlatma, (3) talep/günlük/yedek ve uygulama verisi için süre-silme kararları ve teknik uygulaması, (4) gerçek oturum/tenant/paylaşım/silme akışlarının tam E2E kanıtı, (5) gerekiyorsa isteğe bağlı depolama/servisler için kabul-ret-geri alma davranışı. Yeni analiz/reklam servisi bu paketle eklenmedi. Gösterilmeyen bir izin paneli varmış gibi raporlanmaz; 13–14 tamamlandı değildir.

Hukuki içerik hazırlığı için kaynak: [KVKK aydınlatma yükümlülüğü](https://www.kvkk.gov.tr/Icerik/2033/Aydinlatma-Yukumlulugu-). Genel gizlilik açıklaması, kimlik/amaç/alıcı/toplama-hukuki sebep/haklar içeren faaliyete özel aydınlatma yerine kabul edilmez.

## Web 3D önbellek tercihi — 26 Ekim 2026

[Web PR #14](https://github.com/kyrox-studio/kyrox-web/pull/14) üç CI kontrolü yeşil birleşti; canlı kaynak `9c81440544d96a754b06bc8e0a35d8cbf551d0ce`, tema 1.3.3. Öncesinde `/opt/kyrox/backups/web-preferences-20261026` altında DB/tema yedeği alındı. Kamuya açık tema dosyalarının okuma izinleri korundu; önceki yayın izin sorunu tekrarlanmadı. Paneldeki depolama metninin yalnız beklenen eski paragrafı güncellendi; eşleşme olmasaydı kullanıcı içeriğini ezmeden duracaktı.

- İlk ziyarette yalnız gerçek 3D model önbelleği için açıklama ve eşit biçimde **İzin ver / İzin verme** sunulur. Reklam/analiz kategorisi veya izni uydurulmaz. Model önbelleği izin öncesinde kapalıdır; sahne otomatik açılır, izin yoksa model `no-store` ile yüklenir.
- İzin sonrası sonraki model yüklemeleri sürümlü CacheStorage kullanabilir. Yüklenmiş modelin o anda kendiliğinden yeniden indirilmesi gerekmez; seçenek sonraki yüklemede uygulanır. Alt bilgide **Depolama tercihleri** her zaman erişilebilir.
- Ret/geri alma yalnız `kyrox-stand-scenes-v1` kaydını siler; CRM/Stand oturumlarına, yerel projelere veya başka önbelleklere dokunmaz. Yükleme sırasında geri almadan sonra modelin yeniden kaydedilmemesi de denetlendi.
- Seçim 180 gün bu tarayıcıda hatırlanır; bu süre tercih kaydının geçerliliğidir, tüm uygulama verisinin veya günlüklerin saklama kararı değildir. Süre dolumunda sonraki ziyarette tercih yok sayılıp yeniden sorulur. localStorage/CacheStorage kullanılamadığında sahne normal ağ yüklemesiyle çalışır; kaydedilemeyen tercih yalnız açık sayfada geçerli olduğunu bildirir.
- **Yerel ve gerçek canlı Chromium kanıtı:** varsayılan ret ve otomatik 3D; izin sonrası model kaydı; sonraki açılışta **0 model ağ isteği**; ret ve geri almada temizlik; başka önbelleğin korunması; yükleme devam ederken geri alma; süre dolumu; iki sekme arasında tercih değişikliği; depolama kapalı senaryo; mobilde taşma olmaması ve Escape ile panel kapatma başarılı. Canlı JS hatası gözlenmedi. Ürün SEO/canonical/paylaşım/sitemap/llms kontrolleri tekrar geçti. Tekrar çalıştırılabilir yerel kontrol: `tasarim/scripts/check-storage-preferences.mjs` (FAIR_STAND_ROOT üzerinden Playwright kullanır); görüntüleyici paketlemesi `tasarim/scripts/build-viewer.mjs` içindedir.

Teknik yaklaşım, isteğe bağlı kullanımı varsayılan kapalı tutma ve sonradan değiştirme yönünden [KVKK 2022/1358 karar özetindeki](https://www.kvkk.gov.tr/Icerik/7595/2022-1358) ilkelere göre ele alındı. Bu, tüm veri işleme faaliyetleri için hukuki uygunluk onayı değildir. İlk anonim ziyaret tablosu tarihî envanter kanıtıdır; o tarihte bulunmayan tercih kaydı artık vardır. CRM/Stand uygulamalarındaki veri sınıflandırması, gerçek canlı oturum/paylaşım/silme kontrolleri ve ayrıntılı hukuki işler açık kalır; web paneli bu ürünlerin oturumunu yönetmez. Bu adım yeni analiz/reklam, backend yetki veya tenant davranışı eklemez.

Öncelik: Önce teknik veri akışlarını ve açık eksikleri görünür tutarak ilerle; doğrulanmayan alana tamamlandı/güvenli/uygun işareti koyma. Son hukuki inceleme ayrı kapanış adımıdır, mevcut teknik eksikleri ortadan kaldırmaz.

Takip: [Web ve Ürünler — Yayına Hazırlık](WEB_LAUNCH_CHECKLIST.md), özellikle 13–14, 16 ve 20–21. maddeler.
