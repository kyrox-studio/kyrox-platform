# KYROX — Ortak Güvenlik Kuralları ve Kanıtlı Yol Haritası

**Durum:** Kullanıcı kararıyla ortak güvenlik çerçevesi; 2026-10-25. Kurallar geçerli, tüm kontrollerin kabulü tamamlanmış değildir.
**Kapsam:** Mevcut ve gelecekteki tüm KYROX projeleri, servisler, API/UI, işler, CI/CD, dosyalar, veritabanları ve dağıtım ortamları.
**Tek kaynak:** Güvenlik kuralları, kontrol kimlikleri, güvenlik iş kuyruğu, bulgular ve kanıt dizini bu dosyada tutulur. Projeler buraya bağlanır; kendi güvenlik kontrol listelerini kopyalamaz. Test/kod/CI çıktıları sahip depoda kalır, buradan referanslanır.

## 1. Bağlayıcı çalışma kuralı

Yeni proje açılırken ve mevcut projede davranış, veri, bağımlılık, yapılandırma veya dağıtım değişirken bu çerçeve uygulanır. Belge değişiklikleri de güvenlik kuralı veya iddiasını etkiliyorsa gözden geçirilir; salt yazım düzeltmesi için yapay runtime testi istenmez.

1. Değişiklik sahibi, etkilenen kontrol kimliklerini ve her kontrolün REQUIRED / N/A gerekçesini belirler. Yeni projenin ilk değerlendirmesi tüm kontrol kataloğunu kapsar; sonraki değişikliklerde etki alanı ve ortak bağımlılıklar incelenir.
2. Gereksinim ve sahiplik uygulamadan önce netleşir. Kimlik/yetki gibi mevcut ortak servisler yeniden üretilmez. Yeni bir projenin teknoloji veya ürün adı kurallara istisna oluşturmaz.
3. Kural çakışması, ihlal, eksik test/kanıt veya gereken ek iş açıkça bildirilir. Kayıt; kontrol kimliği, dosya/akış, etki, eksik iş, owner ve kabul koşulunu içerir. Varsayım ile doğrulanmış açık ayrı tutulur.
4. Uygulanabilir güvenlik kuralı ihlali, başarısız negatif test veya eksik zorunlu kanıt varken DONE / güvenli / kabul edildi denmez. İlgisiz güvenli işler ilerleyebilir; etkilenen teslimat kabul edilmez.
5. N/A yalnız somut mimari gerekçeyle verilir. “UI gizli”, “admin kullanıyor”, “testler yeşil”, “sonra bakarız” gerekçe değildir. Mevcut önceliklendirme bir kontrolü atlama izni değildir.
6. Güvenlik düzeltmesi mevcut ürün işlevlerini korur; gerekli davranış değişiklikleri açıkça kararlaştırılır. Test silme, assertion gevşetme, bypass veya görünmez istisnayla sonuç yeşile çevrilmez.
7. Yeni proje için owner, ortamlar, veri sınıfları, güven sınırları, dış bağlantılar, kimlik yöntemi, saklama/geri dönüş gereksinimi ve kanıt yolu tanımlanmadan güvenlik başlangıç kabulü verilmez.

### Mevcut standartlarla ilişki

Bu dosya güvenlik için ana giriş ve kontrol kataloğudur; mevcut ayrıntı kuralları kopyalanmaz:

- [Feature Delivery](../development/FEATURE_DELIVERY_STANDARD.md): teslimat ve kabul sırası.
- [Feature Applicability](../development/FEATURE_APPLICABILITY_STANDARD.md): REQUIRED / N/A sınıflaması.
- [Quality Gate](../quality/QUALITY_GATE_STANDARD.md): strict-green ve regresyon kuralları.
- [Tenant Isolation](../quality/TENANT_ISOLATION_SYSTEM_TEST_STANDARD.md): organization sınırının test semantiği.
- [CRUD / UI Authorization](../ui/CRUD_UI_AUTHORIZATION_STANDARD.md): UI ile backend yetkilerinin eşliği.
- [SaaS Roadmap](../../ecosystem/SAAS_ROADMAP.md): mevcut platform sahipliği ve öncelikler.

Onaylı mimari kararları bu listeyle sessizce değiştirilmez. Örneğin mevcut Platform Super Admin istisnası kimlik doğrulamayı atlatmaz; normal kullanıcı/OrganizationAdmin istisnayı header veya rol adıyla kazanamaz. Çakışma bulunursa burada bulgu olarak kaydedilir ve ilgili kararın sahibiyle çözülür; yeni özellik kendi istisnasını icat edemez.

## 2. Kontrol kataloğu — projeden bağımsız

Öncelik ilk inceleme sırasıdır; gerçek risk ortam ve istismar koşullarıyla yeniden değerlendirilir. Her satırın testleri yalnız uygulanabilir yüzeylere yöneliktir.

| Kimlik | Kontrol / kural | İlk öncelik | Asgari kanıt odağı |
|---|---|---|---|
| SEC-01 | Backend her korumalı işlemin etkili yetkisini denetler; UI güvenlik sınırı değildir. | P0 | Yetkili başarı, yetkisiz ret, bağımsız aksiyonlar, rol/yetki yükseltme negatifleri |
| SEC-02 | Sahiplik ve organization sınırı doğrudan ID, alt kaynak, liste, toplu işlem, dosya ve işlerde korunur. | P0 | A→B ve B→A ret; kendi verisine erişim; ayrı yetkili sistem aktörü senaryosu |
| SEC-03 | Kimlik, token ve oturum yaşam döngüsü sunucuda doğrulanır; kurtarma ve giriş yolları aynı güven sınırına uyar. | P0 | Bozuk/süresi dolmuş token, çıkış/iptal, hesap durumu, tek kullanımlık kurtarma ve tekrar kullanım |
| SEC-04 | Sırlar repo, tarayıcı, log veya artefakta sızmaz; amaç/kapsam ayrımı ve rotasyon yolu vardır. | P0 | Maskelenmiş tarama sonucu, sır envanteri metadata'sı, iptal/rotasyon doğrulaması; sır değeri belgeye yazılmaz |
| SEC-05 | DB erişimi en az yetki ve kontrollü ağ erişimiyle sınırlandırılır. | P0 | Uygulama/migration/backup rol kapsamları, dış erişim ve veri koruma yapılandırması |
| SEC-06 | Sorgu ve komutlarda güvenilmeyen girdi güvenli parametre/allowlist sınırını aşamaz. | P0 | Dinamik SQL/sort/filter ve shell/komut yolları; gerçek enjeksiyon negatifleri |
| SEC-07 | Dosya yükleme/içe aktarma içerik, boyut, açılmış kaynak tüketimi, yol ve erişim yönünden doğrulanır. | P1 | MIME yanıltma, SVG/aktif içerik, bozuk dosya, piksel/ZIP açılım sınırı, traversal ve yabancı dosya ID |
| SEC-08 | Sunucu girdi şeması, aralık, uzunluk, adet ve iş kuralını uygular. | P1 | Sınır/taşma/eksik/fazla alan ve geçersiz ilişki negatifleri; frontend sınırı tek kanıt değildir |
| SEC-09 | Doğrudan ve transit bağımlılıkların riski takip edilir. | P1 | Dil/lockfile kapsamı, tarama tarihi, paket/sürüm ve karar; aracın sıfır bulması tam güvenlik iddiası değildir |
| SEC-10 | CI/CD ve tedarik zinciri erişimi sınırlıdır; güvenilmeyen değişiklik sır veya yayın yetkisi kazanamaz. | P1 | Workflow/token yetkisi, action kaynağı, gerekli kontroller ve gerçek branch protection ayarları |
| SEC-11 | Dış iletişimde HTTPS ve sertifika doğrulaması korunur. | P1 | Canlı yönlendirme/sertifika ve servisler arası güven sınırı; proxy şablonu tek başına kanıt değildir |
| SEC-12 | Yedek erişimi/saklaması korunur; geri yükleme kanıtlanır. | P1 | Son yedek, ayrı ortamda restore, veri/uygulama kontrolü; kararlaştırılmış RPO/RTO |
| SEC-13 | Kritik kimlik/yetki/veri olayları sır sızdırmadan izlenebilir. | P1 | Aktör, kapsam, işlem, sonuç, korelasyon; log erişimi/saklama ve olay yanıtı |
| SEC-14 | Kullanıcı içeriği çalıştırılabilir HTML/script'e dönüşmez. | P1 | Çıktı bağlamı, sanitize/encode, SVG ve zengin içerik yolları; gerçek tarayıcı negatifleri |
| SEC-15 | Cookie oturumları uygun HttpOnly, Secure, SameSite, yol ve yaşam süresiyle korunur. | P1 koşullu | Kabul edilen ortamda Set-Cookie ve oturum sabitleme/temizleme davranışı |
| SEC-16 | Kimlik ve pahalı kaynaklar kötüye kullanım sınırlarına sahiptir. | P1 | Login/reset/upload/export/scraper sınırları, kaynak tüketimi, eşzamanlılık ve proxy güveni |
| SEC-17 | Tarayıcı güvenlik başlıkları uygulamanın gömme/paylaşma ihtiyaçlarıyla uyumludur. | P2 | Etkili CSP, HSTS, nosniff, frame-ancestors, referrer politikası; meşru akış regresyonu |
| SEC-18 | CORS gerekli origin'lerle sınırlıdır; credential ve wildcard birlikte güven sınırını açmaz. | P2 | İzinli/yabancı origin, preflight, credentials, proxy katmanı |
| SEC-19 | Cookie ile kimliklenen durum değiştiren istekler CSRF'e karşı korunur. | P1 koşullu | Eksik/yanlış koruma veya yabancı origin reddi; token taşıma mimarisine göre N/A gerekçesi |
| SEC-20 | Hata yanıtı sır/stack/input sızdırmaz; güvenlik bağımlılığı hata verince yetki açılmaz. | P2 | Güvenli 4xx/5xx, doğrulama yanıtlarında maskeleme, authorization kesintisinde ret |
| SEC-21 | Sunucunun dış URL istekleri iç ağ/metadata/yerel dosya erişimine dönüşmez. | P1 koşullu | Şema, DNS/IP, yönlendirme ve yeniden çözümleme kontrolleri; izole SSRF negatifleri |
| SEC-22 | Asenkron işler ve iş mantığı tekrar/yarış/scope değişikliğiyle yetkisiz sonuç üretmez. | P1 koşullu | Job/run sahipliği, retry/cancel, idempotency, hesap askıya alma ve eşzamanlı işlemler |

CSRF/cookie, dosya, dış URL ve işler gibi koşullu alanlar yoksa N/A kanıtlanır. MFA, yeni quota veya yeni servis gibi ürün/mimari kararları kontrol listesinden kendiliğinden türetilmez; ihtiyaç ve karar açık iş olarak kaydedilir.

## 3. Kanıt ve durum modeli

Her kontrol her proje/akış için ayrı değerlendirilir. Bir projedeki başarı diğerine taşınmaz.

- **DOĞRULANMADI:** kaynak veya çalışma kanıtı yok.
- **KOD VAR / KISMİ:** mekanizma veya test mevcut; uygulanabilir senaryoların kabulü tamamlanmadı.
- **BULGU AÇIK:** kanıtlanan kural ihlali veya doğrulanması gereken somut risk var; ikisi kayıtta ayrılır.
- **KABUL EDİLDİ:** uygulanabilir kaynak, negatif test ve gerekli runtime/CI kanıtı aynı değişikliğe bağlı.
- **N/A:** kapsam ve mimari gerekçe kayıtlı; kapsam değişirse yeniden açılır.

Her kanıt kaydı: `kontrol ID | proje/akış | kaynak dosya+sembol | repo SHA | test/senaryo | komut | tarih/ortam | sonuç | mock/bypass sınırı | CI URL+head (varsa) | runtime kapsamı | kalan iş`.

Sadece test dosyası görmek PASS değildir. Sadece mock ile yetki kontrolü gerçek Core/JWT zincirinin kabulü değildir. Önceki başarılı koşu yeni head veya canlı ortamın otomatik kanıtı değildir. Üretim kabulü denirse ortam, sürüm ve güvenli gerçek akış ayrıca belirtilir. Sırlar, JWT, ham kişisel veri ve saldırı yükleri bu belgeye yazılmaz.

## 4. Yol haritası ve bitiş koşulları

| Dalga | İş | Bitiş koşulu | Durum |
|---|---|---|---|
| 0 | Tek kaynak, ortak kurallar, mevcut kanıtlar ve bulgu kaydı | Giriş/teslimat belgeleri bu dosyaya bağlı; mevcut otomasyon sınırı açık | Bu değişiklikle belgelendi |
| 1 | SEC-01/02/03: kimlik, yetki, organization sınırı | Core + tüketici servislerde gerçek kimlikle kendi/yabancı kaynak, iptal edilmiş oturum ve sistem aktörü matrisi; eksik akışların listesi | Başladı: kaynak taraması; runtime kabulü açık |
| 2 | SEC-04/05/06/07/08/14/21: sır, DB, girdi, dosya ve dış URL | Somut yüzey envanteri, negatifler ve bulgu düzeltmeleri; meşru dosya/entegrasyon regresyonları geçer | İlk dosya bulguları aşağıda |
| 3 | SEC-09/10/11/12/13: pipeline, bağımlılık, TLS, yedek, audit | Tarama kapsamı ve zorunluluk kanıtı; ayrı restore ortamında başarı; maskelenmiş canlı yapılandırma kontrolü | Açık |
| 4 | SEC-15/16/17/18/19/20/22: tarayıcı, kötüye kullanım, hata ve işler | Uygulanabilir koşulların negatifleri; proxy ve uygulama birlikte doğrulanır | Açık |
| 5 | Yeni proje/değişiklik için otomatik uygunluk denetimi | Sahip repo mevcut contract/verify/CI akışında eksik uygulanabilir kontrol/test/kanıt için başarısız olur; yanlış N/A ve yeni yüzey regresyon testleri var | KISMİ — CRM main'de, CI pozitif/negatif kanıtlı (E-010); Core main'de, CI pozitif/negatif kanıtlı (E-012); Stand PR'da (E-011). Required check/branch protection doğrulanmadı |

Somut yüksek etkili bulgu bulunduğunda dalga sırası onu bekletmez. Canlı saldırı, yük testi, restore veya yapılandırma değişikliği bu belgenin varlığıyla otomatik yapılmaz; izole test ve açık operasyon kapsamı kullanılır.

### Otomatik uyarı / engelleme hedefi

Yeni paralel test sistemi kurulmayacak. Sahip deponun mevcut feature/change contract ve verify/CI mekanizması genişletilecek; bu dosyadaki SEC kimlikleri referans olacak, kurallar kopyalanmayacak.

- Yeni proje: ortak kontrol kataloğunun tam başlangıç sınıflaması aranacak.
- Değişiklik: etkilenen yollar, kontroller, REQUIRED/N/A nedenleri ve çalıştırılabilir test referansları aranacak. Geniş kapsamlı “etki yok” beyanı tek başına yeterli olmayacak.
- **BLOCK:** kural çakışması/ihlali, eksik zorunlu değerlendirme/test/kanıt, başarısız kontrol, belirsiz kritik sahiplik. İlgili kabul durur.
- **UYARI / EK İŞ:** değerlendirmede bulunan ek kapsam, henüz kabul iddiası olmayan inceleme veya eski kanıt. Kontrol uygulanabilir ve zorunluysa yalnız uyarıda bırakılamaz; BLOCK olur.
- Çıktı: SEC ID, dosya/akış, sebep, gereken iş, owner ve tekrar doğrulama yolu. İstisna gerekirse kapsam/süre/onay ve telafi kontrolü burada izlenir; sessiz skip kabul edilmez.
- Statik eşleştirme yalnız kapsam ve kanıt bütünlüğünü denetler; açık bulunmadığını veya insan N/A beyanının doğru olduğunu tek başına kanıtlamaz. Negatif test ve review devam eder.

**Bugünkü gerçek durum:** Ortak manuel inceleme/teslimat kuralı geçerlidir. Üç depoda SEC kimliklerini mevcut contract/pytest mekanizmasıyla denetleyen CI adımı vardır (aşağıda); her biri yalnız kayıt bütünlüğünü ve test referansının varlığını kanıtlar. Testin geçtiğini aynı CI'daki test adımları, güvenliği negatif testler ve review kanıtlar; bu gate'ler "sistem güvenli" kanıtı değildir.

| Depo | Mevcut mekanizma | SEC bağlantısı | Durum |
|---|---|---|---|
| Fair CRM | `scripts/validate_feature_contracts.py` + `.kyrox/features/*.json`; CI `development-standard.yml` → "Feature Contract / Applicability" | Contract'ta `security.controls`. PR diff'inde eklenen/değişen veya material değişikliği kapsayan contract'ta zorunlu; REQUIRED → çözümlenen test referansı, N/A → reddedilen kalıplar dışı ≥20 karakter gerekçe; `permissions`/`runtime_authorization` → SEC-01, `tenant_isolation` → SEC-02 REQUIRED. Dokunulmayan 155 eski contract muaf. | **CI'da çalışıyor, main'de** — E-010 |
| Fair Stand | `.github/change-contract.json` + `scripts/verify-change-contract.mjs` (CI "Change contract gate") | `impact.security = affected` → `security.controls` zorunlu, aynı REQUIRED/N/A kuralları; `not-applicable` iken REQUIRED yasak. Path duvarı `backend/` ve güvenlik dosyalarında `security`'yi zaten zorunlu kılar. | PR açık — E-011 |
| Core | Mevcut pytest/CI; `.kyrox/security-controls.json` + `backend/tests/test_security_controls_registry.py` | Proje düzeyi SEC-01…22 kaydı; kanıtı eksik REQUIRED kontrol yalnız açık `gap` ile yazılabilir; `backend/app` altındaki her yüzey kayıtlı olmalı (yeni modül sınıflanmadan FAIL). Sığ checkout nedeniyle diff zorunluluğu yok. | **CI'da çalışıyor, main'de** — E-012 |

## 5. İlk kaynak kanıtları — 2026-10-25

İncelenen yerel commit'ler: Core `0a4fe17d725fd540117f2d9721cbe51ba88199dd`; CRM `9e9b20cb45b1059ad39252cc0d9c45e8370722a9`; Stand `ac18bc1a5d972095652f8a449fdab2bf66375dd3`. Bunlar canlı dağıtım veya yeni başarılı CI iddiası değildir.

Aşağıdaki yollar belirtilen kod deposunun köküne göredir.

| Kayıt | Kontrol / sahip | Kaynak ve test dayanağı | Güncel sınır / sonraki iş |
|---|---|---|---|
| E-001 | SEC-01/02 — CRM | `backend/tests/system/tenant_isolation_registry.py`, `test_tenant_isolation_governance.py`; `backend/tests/modules/test_endpoint_permission_enforcement.py` | Kayıt/test var; bu çalışmada çalıştırılmadı. API, dosya, job kapsamı ve gerçek actor zinciri tekrar eşleştirilecek. |
| E-002 | SEC-03 — Core | `backend/app/modules/identity/infrastructure/authentication/security/jwt_token_service.py`; `backend/tests/modules/identity/authentication/test_core09_security_certification.py` | JWT imza/claim mekanizması ve negatif testler var; bu çalışmada Core test/runtime koşulmadı. Ürünlerin oturum iptalini tüketmesi ayrıca kanıtlanacak. |
| E-003 | SEC-15/19 — CRM | `backend/app/modules/auth/api/cookies.py`: `set_refresh_cookie`, `csrf_header_ok`; `backend/tests/modules/auth/test_auth_bridge.py` | HttpOnly ve CSRF özel başlığı var; Secure ortam ayarına bağlı. Test Core client'ı mock eder; canlı cookie/Origin kabulü değil. |
| E-004 | SEC-01/02 — Stand | `backend/app/modules/fair_stand/api/dependencies.py`: `require_permission`; `application/projects.py`: `_project_statement` (aynı fair_stand modülü altında) | Permission adapter ve organization filtreleri var. Yabancı proje/revizyon/asset/export ve gerçek Core oturumu matrisi açık. |
| E-005 | SEC-07/08 — Stand | `backend/app/modules/fair_stand/application/image_optimize.py`: `optimize_uploaded_image`; `backend/tests/modules/fair_stand/test_image_optimize.py` | Byte sınırı/normalize var; F-001/F-002 açık. Mevcut testler yeni risklerin negatif kanıtı değil. |
| E-006 | SEC-09/10 — Stand/Core/CRM | Stand `.github/workflows/ci.yml` içinde Dependency audit (`npm run audit:deps`); CRM `.github/workflows/development-standard.yml` | **Düzeltme 2026-10-25:** Core `.github/workflows/ci.yml` (`0a4fe17`) içinde dependency audit adımı **yoktur**; ilk kayıttaki "Core aynı yol" ifadesi yanlıştı. Core lint adımı ruff yoksa atlıyor (CI log'unda "ruff not installed, skipping", E-012). Stand audit'i npm tarafını kapsar; Stand Python backend'i kapsamı doğrulanmadı. Required check ve branch protection doğrulanmadı. |
| E-007 | SEC-12 — CRM / operasyon | `scripts/server/backup/kyrox-backup.sh`, `kyrox-restore.sh`; `backend/tests/shared/test_database_backup_restore.py` | Kod/test var; bu turda yedek veya restore çalıştırılmadı. Canlı plan, erişim, harici kopya ve restore kanıtı açık. |
| E-008 | SEC-18/20 — CRM | `backend/app/main.py`: development/local/test CORS; genel exception handler | Kod var; canlı proxy/header ve validation hata içeriği kabulü açık. |

### Çalıştırılan hedefli kanıt E-009

- Sahip: Fair Stand; SEC-01 ve SEC-07'nin dar mevcut regresyonları.
- Ortam: Yerel Windows, backend `.venv`, SQLite/geçici dosya testleri; tarih 2026-10-25.
- Komut: `python -m pytest tests/modules/fair_stand/test_image_optimize.py tests/modules/fair_stand/test_projects_put_permissions.py -q` (backend dizini).
- Sonuç: **5 passed**, Starlette/httpx kullanımdan kaldırma uyarısı var.
- Sınır: 4 görüntü normalize/byte sınırı testi ve 1 create/update permission ayrımı. Permission testi `SelectiveAuthorization` kullanır; gerçek Core yetki kabulü değildir. Tüm güvenlik, SVG, kaynak tüketimi veya çapraz organization kabulü değildir. Yeni CI URL'si yok; yalnız yerel koşu.

### Çalıştırılan hedefli kanıt E-010 — CRM otomatik SEC sınıflama gate'i

- Sahip: Fair CRM; SEC-10 (gate'in kendisi) ve Dalga 5'in ilk adımı.
- Kaynak: base `9e9b20cb45b1059ad39252cc0d9c45e8370722a9`, head `9897fb7d0668d07d44f60ba415cab599a81c212f` (dal `feat/security-applicability-gate`), PR [kyrox-studio/kyrox-fair-crm#362](https://github.com/kyrox-studio/kyrox-fair-crm/pull/362), main merge commit `8f44468eafa636fc4d3c05898d5e5b194fd36125`. Dosyalar: `scripts/validate_feature_contracts.py` (`validate_security`, `_test_reference_error`, `material_change_errors`), `.kyrox/feature-contract.schema.json`, `.kyrox/features/security-applicability-gate.json`, `backend/tests/scripts/test_validate_feature_contracts_security.py`, `.github/workflows/development-standard.yml` (backend job tetik listesine validator ve şema eklendi; önceden yalnız validator değişince regresyon testleri koşmuyordu).
- Ortam: Yerel Windows, sistem Python 3.12.10; 2026-10-25.
- Komutlar ve sonuç (repo kökü):
  - `python -m pytest backend/tests/scripts/ -q` → **51 passed** (yeni dosya 26). Eksik/bilinmeyen kontrol, geçersiz status, gerekçesiz/reddedilen N/A, testsiz REQUIRED, bozuk/var olmayan/kapsam dışı test referansı, SEC-01/02 tutarsızlığı FAIL; geçerli kayıt PASS. Diff zorunluluğu geçici git deposunda: yeni contract'ta security yok → FAIL, material değişikliği kapsayan eski contract → FAIL, dokunulmayan eski contract → PASS, geçerli kayıt + material değişiklik → PASS.
  - `python scripts/validate_feature_contracts.py --contracts-only` → PASS (156 contract).
  - CLI uçtan uca, scratch klon: geçerli değişiklik `--base main~1` → exit 0; security bloğu silinmiş contract → exit 1; bozuk test referansı → exit 1.
  - `python scripts/backend_test_baseline.py` → PASS.
- Gerçek CI (GitHub Actions, ubuntu, Python 3.12), head `9897fb7`:
  - Pozitif — [run 40188775294](https://github.com/kyrox-studio/kyrox-fair-crm/actions/runs/40188775294): "Feature Contract / Applicability" `--base 9e9b20cb… --head 9897fb7…` → `DEVELOPMENT STANDARD GATE: PASS (156 valid feature contracts)`; "Backend Quality Check" → **2264 passed, 1 skipped** (yeni testler dahil); diğer tüm kontroller pass.
  - Negatif — geçici draft PR [kyrox-studio/kyrox-fair-crm#363](https://github.com/kyrox-studio/kyrox-fair-crm/pull/363), [run 40189153667](https://github.com/kyrox-studio/kyrox-fair-crm/actions/runs/40189153667): contract'tan `security` silindi → gate **FAIL** ("changed or material-covering contract requires a security.controls classification"). PR kapatıldı, dal silindi.
- Mock/bypass sınırı: Statik denetim; referanslanan testlerin geçtiği gate'te değil backend pytest job'ında kanıtlanır. Required check ve branch protection ayarı doğrulanmadı. Hiçbir ürün özelliğinin güvenli olduğu iddia edilmez.
- Durum: KOD VAR + CI kanıtı; Dalga 5'in CRM adımı. Kalan: required check/branch protection doğrulaması.

### Çalıştırılan hedefli kanıt E-011 — Stand change-contract SEC gate'i

- Sahip: Fair Stand; SEC-10 ve Dalga 5.
- Kaynak: base `ac18bc1a5d972095652f8a449fdab2bf66375dd3` (origin/main), head `02788684631d9d2f864826ce0817e2f70124f2ba` (dal `feat/security-sec-controls`, ayrı worktree; `feat/stand-mobilya-kilit-uyari` dalı ve onun commit'siz e2e değişikliklerine dokunulmadı), PR [kyrox-studio/kyrox-fair-stand#343](https://github.com/kyrox-studio/kyrox-fair-stand/pull/343). Dosyalar: `src/systemChangeContract.js` (`validateSecurityControls`, `securityTestReferences`, `securityTestReferenceError`), `scripts/verify-change-contract.mjs`, `test/systemChangeSecurityControls.test.js`, `SYSTEM_CHANGE_GATE.md`, `.github/change-contract.json` (bu değişikliğin kendi kaydı; SEC-10 REQUIRED).
- Ortam: Yerel Windows, Node 22 (`npm ci`); 2026-10-25.
- Komut/sonuç: `npm test` → **1532/1532**; `npm run syntax:check` → 0; yeni dosya 15 test (eksik/bilinmeyen kontrol, gerekçesiz/reddedilen N/A, testsiz REQUIRED, not-applicable+REQUIRED, bozuk/kapsam dışı/tanımsız Python referansı FAIL; CLI bozuk referans → exit 1, security bloğu yok → exit 1, geçerli → exit 0); `CHANGE_GATE_BASE=origin/main npm run contract:verify` → accepted.
- Gerçek CI negatif: geçici draft PR [kyrox-studio/kyrox-fair-stand#344](https://github.com/kyrox-studio/kyrox-fair-stand/pull/344), [job 121157742297](https://github.com/kyrox-studio/kyrox-fair-stand/actions/runs/40189656223/job/121157742297): bozuk SEC-10 referansı → "Security control test references do not resolve", exit 1. PR kapatıldı, dal silindi.
- Gerçek CI pozitif: [run 40189603070](https://github.com/kyrox-studio/kyrox-fair-stand/actions/runs/40189603070) — sonuç aşağıda.
- Sınır: `impact.security = not-applicable` beyanının doğruluğunu gate kanıtlamaz; yalnız path duvarının zorladığı dosyalarda yanlış beyan engellenir. Güvenlik iddiası yok.
- Not: Platform F-00x numaraları Stand'in kendi audit F-00x numaralarıyla aynı değildir (Stand F-004/005/006/009 kapalı gate bulgularıdır).

### Çalıştırılan hedefli kanıt E-012 — Core SEC kontrol kaydı (mevcut pytest/CI)

- Sahip: Core; SEC-01…22 proje düzeyi ilk sınıflama ve Dalga 5.
- Kaynak: base `0a4fe17d725fd540117f2d9721cbe51ba88199dd`, head `c58824d27362aefd93ec265a53e7f85485bd3928` (dal `feat/security-controls-registry`), PR [kyrox-studio/kyrox-core#43](https://github.com/kyrox-studio/kyrox-core/pull/43), main merge commit `3051d2e93593d6610c63090b734246c3eef79aef`. Dosyalar: `.kyrox/security-controls.json`, `backend/tests/test_security_controls_registry.py`. Yeni CI adımı yok; mevcut `pytest` adımı kaydı doğrular.
- Sınıflama: REQUIRED + test: SEC-01, 02, 03, 08, 13, 20, 22 (31 mevcut negatif/pozitif test). REQUIRED + açık **gap**: SEC-04 (secret scanning/anahtar rotasyonu), 05 (DB rol kapsamı), 06 (enjeksiyon negatifleri yok; kaynakta raw SQL/shell bulunmadı), 09 (Core CI'da dependency audit yok), 10 (workflow `permissions` yok, ruff sessizce atlanıyor), 11 (STARTTLS testi yok, lifecycle URL https zorunlu değil), 12 (Core restore kanıtı yok), 16 (Core kodunda login/reset rate limit/lockout yok; upstream proxy doğrulanmadı), 17 (güvenlik başlıkları uygulamada yok; proxy doğrulanmadı). N/A (kaynak taraması 2026-10-25): SEC-07 (upload yok), 14 (JSON + düz metin e-posta), 15/19 (cookie yok, bearer token), 18 (CORSMiddleware yok), 21 (dış hedefler yalnız deployment ayarından).
- Ortam: Yerel Windows, Core `.venv`; 2026-10-25. Komutlar: `pytest backend/tests/test_security_controls_registry.py` → **19 passed** (eksik/bilinmeyen kontrol, gerekçesiz/reddedilen N/A, gap'siz testsiz REQUIRED, belirsiz gap, bozuk referans, kayıtsız yeni modül, silinmiş yüzey FAIL; geçerli kayıt PASS); referanslı 31 test → passed; tam `pytest` → **460 passed**.
- Gerçek CI pozitif: [run 40189619392](https://github.com/kyrox-studio/kyrox-core/actions/runs/40189619392) "Backend CI" → **460 passed**, gap uyarısı log'da; aynı log'da "ruff not installed, skipping".
- Gerçek CI negatif: geçici draft PR [kyrox-studio/kyrox-core#44](https://github.com/kyrox-studio/kyrox-core/pull/44), [job 121157765534](https://github.com/kyrox-studio/kyrox-core/actions/runs/40189664088/job/121157765534): SEC-07 silindi → "Backend CI" **FAIL** (`AssertionError: controls missing: SEC-07`). PR kapatıldı, dal silindi.
- Sınır: Testler TestClient/yerel DB ile koşar; gerçek dağıtım, proxy ve canlı ortam kabulü değildir. Gap'li kontroller KABUL EDİLMEDİ durumundadır; ilgili teslimatlar bu kontroller için kabul alamaz.

Not: Stand yerel ana kopyası incelemede `feat/stand-mobilya-kilit-uyari` `83878db2` dalındaydı ve başka bir çalışmanın commit'siz e2e değişikliklerini taşıyordu; E-004/E-005/F-00x yeniden doğrulanırken SHA güncellenecek.

## 6. Açık bulgular ve ilk uygulama kuyruğu

| Kimlik | Kanıtlanan gözlem | Risk / henüz kanıtlanmayan etki | Sonraki iş ve kabul |
|---|---|---|---|
| F-001 / SEC-07/14 | Stand `image_optimize.py:optimize_uploaded_image` SVG MIME veya MIME'siz `<` ile başlayan içeriği SVG olarak değiştirmeden döndürüyor. `api/project_routes.py:download_project_asset` inline sunuyor. | P1 inceleme; aynı-origin aktif içerik riski. Tarayıcı/CSP/asset erişimi birlikte test edilmedi; çalışan XSS istismarı iddiası yok. | Stand sahibi: mevcut SVG kullanımını ve sunum yolunu izle; izole aktif içerik negatifleri. Sanitize/rasterize/izole sunum seçimi meşru logo ve görsel işlevleri korunarak kesinleştirilsin. |
| F-002 / SEC-07/16 | Stand `_open_raster_for_optimize` içinde `Image.MAX_IMAGE_PIXELS=None`; `opened.load()` küçültmeden önce çalışıyor. | P1 inceleme; byte limitine sığan çok büyük rasterın CPU/RAM tüketimi. Etkili kaynak üst sınırı veya istismar yük testi bu turda kanıtlanmadı. | Stand sahibi: açılım/piksel/bellek sınırı ve eşzamanlılık yaklaşımını çıkar; izole sınırlı testte reddi/limitli işlemeyi ve meşru yüksek çözünürlük regresyonunu kanıtla. |
| F-003 / SEC-07/16 | Stand `upload_project_asset` dosyayı `await file.read()` ile tam okuyor; uygulama byte sınırı daha sonra service katmanında. | P1 inceleme; proxy/multipart limitinin tüm ortamlarda kapsamı bilinmiyor. Tek başına sınırsız canlı upload kanıtı değildir. | Stand/operasyon: etkin upstream limiti ve uygulama okuma sınırını eşleştir; farklı giriş yollarında limit negatifleri. |
| F-004 / SEC-16 | Core `backend/app` içinde login/password-reset/signup için rate limit, lockout veya throttle kodu yok (kaynak taraması 2026-10-25, `0a4fe17`). | P1 inceleme; parola deneme/reset kötüye kullanımı. Upstream proxy limiti doğrulanmadı; çalışan istismar iddiası yok. | Core/operasyon: proxy limitini maskeli kanıtla veya uygulama sınırı için ürün kararı al; limit negatif testi. E-012 gap'i bu bulguya bağlı. |
| F-005 / SEC-09/10 | Core CI'da dependency audit adımı yok; lint adımı ruff kurulu değilse atlıyor (CI log, E-012); workflow `permissions` bloğu yok. | P1; bağımlılık riski ve lint kapısı izlenmiyor. | Core sahibi: mevcut CI'ya pip audit adımı ve ruff kurulumu; mevcut testleri gevşetmeden. Kapanış CI kanıtıyla. |

Bulgu kaydında düzeltme commit'i, ilgili negatif test, gerekli CI/runtime kanıtı ve kalan risk eklenmeden kapanış yapılmaz. Kod veya veri değiştirilmedi; bulgular bu başlangıç turunda yalnız kaydedildi.

## 7. Kaynaklar ve bakım

Başlangıç girdisi: kullanıcının 2026-10-25 tarihli `KYROX_SECURITY_CONTROLS_PRIORITY_DRAFT.md` taslağı; taslak ayrı bir bağlayıcı kaynak değildir. Bu belge mevcut onaylı standartlarla ilişkilendirilmiş ana güvenlik kaydıdır.

Dış doğrulama çerçevesi: [OWASP ASVS 5.0.0](https://github.com/OWASP/ASVS/tree/v5.0.0/5.0). ASVS eşlemesi henüz tamamlanmadı; OWASP uyumluluğu veya sertifikası iddia edilmez. Top 10 tek başına bu kontrol kataloğunun kabul ölçütü değildir.

Yeni güvenlik işi burada SEC/F/E kimliğiyle izlenir; proje yol haritası yalnız ilgili kayda link verir. Testler ve CI mekanizmaları sahip kod deposunda kalır. Büyük log/rapor kanıtı kopyalanmaz; commit'e bağlı test/CI/artefakt referansı verilir. Davranış değişince ilgili kanıt yeniden değerlendirilir; eski PASS süresiz güvenlik garantisi değildir.
