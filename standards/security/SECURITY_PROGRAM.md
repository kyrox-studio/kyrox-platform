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
| 1 | SEC-01/02/03: kimlik, yetki, organization sınırı | Core + tüketici servislerde gerçek kimlikle kendi/yabancı kaynak, iptal edilmiş oturum ve sistem aktörü matrisi; eksik akışların listesi | E-013 denetimi: Core gerçek zincirle negatif testli; CRM/Stand izolasyonu mock Core ile sızıntısız; gerçek Core rol matrisi ve F-C1…F-C10 açık (§6) |
| 2 | SEC-04/05/06/07/08/14/21: sır, DB, girdi, dosya ve dış URL | Somut yüzey envanteri, negatifler ve bulgu düzeltmeleri; meşru dosya/entegrasyon regresyonları geçer | E-013: envanter ve bulgular §6; F-001…F-003 izole doğrulandı; F-C7 düzeltmesi PR'da |
| 3 | SEC-09/10/11/12/13: pipeline, bağımlılık, TLS, yedek, audit | Tarama kapsamı ve zorunluluk kanıtı; ayrı restore ortamında başarı; maskelenmiş canlı yapılandırma kontrolü | E-013: izole restore sentetik veriyle başarılı; canlı yapılandırma maskeli incelendi; F-V5…F-V7, F-G1…F-G3 açık |
| 4 | SEC-15/16/17/18/19/20/22: tarayıcı, kötüye kullanım, hata ve işler | Uygulanabilir koşulların negatifleri; proxy ve uygulama birlikte doğrulanır | E-013: CORS canlıda kapalı; F-004, F-V8, F-B3, F-C10/F-R4/F-S4 açık; SEC-22 kısmen doğrulanmadı |
| 5 | Yeni proje/değişiklik için otomatik uygunluk denetimi | Sahip repo mevcut contract/verify/CI akışında eksik uygulanabilir kontrol/test/kanıt için başarısız olur; yanlış N/A ve yeni yüzey regresyon testleri var | KISMİ — CRM main'de, CI pozitif/negatif kanıtlı (E-010); Core (E-012) ve Stand (E-011) main'de, CI pozitif/negatif kanıtlı. Required check/branch protection doğrulanmadı |

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
| Fair Stand | `.github/change-contract.json` + `scripts/verify-change-contract.mjs` (CI "Change contract gate") | `impact.security = affected` → `security.controls` zorunlu, aynı REQUIRED/N/A kuralları; `not-applicable` iken REQUIRED yasak. Path duvarı `backend/` ve güvenlik dosyalarında `security`'yi zaten zorunlu kılar. | **CI'da çalışıyor, main'de** — E-011 |
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
| E-006 | SEC-09/10 — Stand/Core/CRM | Stand `.github/workflows/ci.yml` içinde Dependency audit (`npm run audit:deps`); CRM `.github/workflows/development-standard.yml` | **Düzeltme 2026-10-25:** Core `.github/workflows/ci.yml` (`0a4fe17`) içinde dependency audit adımı **yoktur**; ilk kayıttaki "Core aynı yol" ifadesi yanlıştı. Core lint adımı ruff yoksa atlıyor (CI log'unda "ruff not installed, skipping", E-012). Stand audit'i npm tarafını kapsar; Stand Python backend'i kapsamı doğrulanmadı. Required check ve branch protection doğrulanmadı. **Düzeltme 2 (E-013):** CRM `development-standard.yml` içinde npm/pip audit adımı **yoktur**; Python bağımlılık taraması hiçbir depoda CI'da yok (F-G3). |
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
- Gerçek CI pozitif: [run 40189603070](https://github.com/kyrox-studio/kyrox-fair-stand/actions/runs/40189603070) head `0278868` → success. #342 main'e girince `.github/change-contract.json` çakıştı; main birleştirildi (head `bc28de5c0d553616cec00d043ca1410c59960906`, yerelde gate accepted, `npm test` 1545/1545) ve [run 40191723793](https://github.com/kyrox-studio/kyrox-fair-stand/actions/runs/40191723793) → **success**: "Change contract accepted: security-sec-controls", Catalog API pytest 200 passed, E2E 244 passed. Main merge commit `06f205bfa261fced43627ccceef5416ebd79b52f`.
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

## 6. Uçtan uca denetim — 2026-10-25 (E-013)

### 6.0 Kapsam, ortam ve kanıt türleri

- **Kaynak SHA (origin/main):** Core `3051d2e93593d6610c63090b734246c3eef79aef`, CRM `8f44468eafa636fc4d3c05898d5e5b194fd36125`, Stand `ac18bc1a5d972095652f8a449fdab2bf66375dd3`, Web `22e763f56ec5828fe6c3a24d312b13c3839f82cf`. İnceleme yalnız ayrık worktree'lerde yapıldı; başka oturumların dallarına dokunulmadı.
- **Canlı sürüm (salt okunur):** Core `0a4fe17`, CRM `9e9b20cb` (main'in bir önceki commit'i), Stand `ac18bc1a`. Canlıda yalnız yapılandırma/metadata okundu, sırlar yalnız sha256 ilk 8 karakteriyle karşılaştırıldı; DB satırı okunmadı, hiçbir şey değiştirilmedi, rate-limit/yük/upload/restore testi canlıda yapılmadı. Tekil GET/HEAD/OPTIONS istekleri: ana sayfa, sağlık, `/kyrox-core/docs`, yabancı origin preflight.
- **İzole test:** yerel Windows, SQLite/TestClient; Core'da ayrıca geçici Postgres; restore tatbikatı ayrı Docker konteynerlerinde sentetik veriyle (konteynerler silindi).
- **Kanıt türleri her bulguda ayrıdır:** **Kod** = kaynak gözlemi · **İzole** = yerel/izole çalıştırılmış negatif test veya istismar · **Canlı** = canlı yapılandırma/metadata gözlemi. "Canlıda doğrulandı" yalnız canlıda doğrudan gözlenen yapılandırma için kullanılır; canlıda istismar denenmedi.
- **Test sınırı:** Mevcut CRM (1561+34 izolasyon/permission testi) ve Stand (65) testleri Core yetki kararını mock'lar. Core testleri (460) gerçek login/JWT/session zinciriyle koşar. Denetim negatif test betikleri depoya eklenmedi; kalıcı regresyon testi her düzeltme dalında eklenir (aşağıda "eksik doğrulama").

### 6.1 Yüzey envanteri

| Yüzey | Sayım | Kimlik / sınır mekanizması |
|---|---|---|
| Core API | 61 route + 4 docs/openapi | Kimliksiz 8 (auth/recovery), yalnız bearer 3, Super Admin 15, org erişimi + permission'sız 3, `require_permission` 29, servis token'ı 3 (lifecycle 2, notification 1). Davet route'u yok. |
| CRM API | 264 route | Public 12 (health, 9 auth, lifecycle internal, MailerSend webhook); permission dependency'de 137, endpoint/use-case içinde 115. Org bağlamı `X-Organization-Id`; her korumalı istekte Core `authorization/check`. Cookie yalnız `/auth/refresh` ve `/auth/logout`. |
| Stand API | 67 HTTP + 1 WS + `/health` | Projeler 11, cost-items 4, group-templates 5, live share 2 + WS, katalog 2 (Core `access/verify`), admin 43 (global tablolar, system-scope atanamaz permission). |
| Dosya yüzeyleri | CRM teklif logosu; Stand proje asset'leri, export | Upload/indirme route'ları ve sunucu dizinleri |
| Dış URL (sunucu çıkışı) | CRM website zenginleştirme/scraper, MailerSend, SMTP, Core lifecycle/notification | `httpx` istemcileri |
| Arka plan işleri | Core jobs/notification worker; CRM mail worker, operations, organization-closure | DB/job tabloları |
| Veritabanı | `kyrox-db` (PostgreSQL): `kyrox_core`, `fair_crm`, `fair_stand`; WordPress MariaDB | Rol/grant, pg_hba |
| Sunucu/proxy | nginx (fuar.kyrox.studio + kyrox.studio), 4 systemd servisi, docker, SSH, ufw, yedek (yerel + Drive) | Canlı salt okunur |
| Tedarik zinciri | 4 depo, workflow'lar, npm/pip bağımlılıkları, GitHub ayarları | `gh api` salt okunur |

### 6.2 Kontrol matrisi (yüzey × SEC)

Durumlar §3'tedir. "TEST GEÇTİ (izole)" kabul değildir; "CANLI" yalnız yapılandırma gözlemidir.

| SEC | Core | CRM | Stand | Sunucu / DB / CI |
|---|---|---|---|---|
| 01 | BULGU AÇIK (F-C1, F-C2, F-C9); internal uçlar kullanıcı JWT'sini ret: TEST GEÇTİ (izole) | KOD VAR; Core reddederken 239/252 route 403 TEST GEÇTİ (izole, mock Core); kalanlar F-R3 | KOD VAR; admin 43 route system-scope; TEST GEÇTİ (izole, mock) | — |
| 02 | Org A→B kullanıcı/rol/org: TEST GEÇTİ (izole, gerçek zincir) | 16 kaynakta 113 yabancı ID çağrısı, sızıntı 0: TEST GEÇTİ (izole, mock Core); F-R3, R-CRM-1 açık | 10 işlem iki yön 404: TEST GEÇTİ (izole, mock); F-S4 | — |
| 03 | BULGU AÇIK (F-C3, F-C4, F-C4b, F-C6) | KOD VAR (fail-closed Core kesintisinde 403, izole); oturum iptali yalnız Core çağrısında | BULGU AÇIK (F-S5, 10 sn önbellek) | — |
| 04 | BULGU AÇIK (F-C5, F-C7, F-C8) | BULGU AÇIK (F-R4 log'a gövde; varsayılan sırlar) | BULGU AÇIK (F-V3) | BULGU AÇIK (F-V2, F-V3, F-G1) |
| 05 | — | — | — | BULGU AÇIK (F-V1) |
| 06 | KOD VAR (raw SQL/shell yok) | KOD VAR (sort whitelist 15 test geçti); R-CRM-1 raw SQL | KOD VAR | — |
| 07 | N/A (upload yok) | BULGU AÇIK (F-R1) | BULGU AÇIK (F-001, F-002, F-003) | nginx 256m: CANLI |
| 08 | KOD VAR | KOD VAR; F-R1 | BULGU AÇIK (F-S6) | — |
| 09 | BULGU AÇIK (F-G2, F-005) | BULGU AÇIK (F-G2; npm 0 açık) | BULGU AÇIK (F-G2 Pillow) | — |
| 10 | BULGU AÇIK (F-005) | BULGU AÇIK (F-G3) | KOD VAR (E-011 PR'da) | BULGU AÇIK (F-G3) |
| 11 | DOĞRULANMADI (STARTTLS) | DOĞRULANMADI | DOĞRULANMADI | HTTPS yönlendirme, TLS≥1.2, sertifika geçerli: CANLI |
| 12 | — | KOD VAR; izole restore TEST GEÇTİ (E-013) | — | BULGU AÇIK (F-V5, F-V6, F-V7) |
| 13 | KOD VAR; R-3 | KOD VAR (audit best-effort) | DOĞRULANMADI | BULGU AÇIK (DB/auditd log yok, F-V9) |
| 14 | N/A | BULGU AÇIK (F-B1, F-B4) | BULGU AÇIK (F-001); frontend innerHTML 25 kullanım güvenli (izole) | — |
| 15 | N/A (bearer) | KOD VAR (HttpOnly/Secure/SameSite=lax, izole prod ayarı; canlı Set-Cookie görülmedi) | N/A (bearer) | — |
| 16 | BULGU AÇIK (F-004) | BULGU AÇIK (F-004) | BULGU AÇIK (F-002, F-003) | nginx `limit_req` yok: CANLI (F-004) |
| 17 | BULGU AÇIK (F-C11) | BULGU AÇIK (F-V8) | BULGU AÇIK (F-V8) | Başlık yok: CANLI (F-V8) |
| 18 | N/A (CORS yok) | Prod'da CORS yok, yabancı origin 405: CANLI | Prod'da CORS yok: CANLI | — |
| 19 | N/A (bearer) | BULGU AÇIK (F-B3 logout); refresh başlıksız 403: TEST GEÇTİ (izole) | N/A (bearer) | — |
| 20 | BULGU AÇIK (F-C10, F-C11) | BULGU AÇIK (F-R4) | BULGU AÇIK (F-S4) | — |
| 21 | N/A (dış hedef ayardan) | BULGU AÇIK (F-B2) | N/A (sunucu çıkışı yok; TURN yapılandırılmamış) | — |
| 22 | KOD VAR (mevcut testler) | DOĞRULANMADI (job/run yabancı ID testi eksik) | BULGU AÇIK (F-S7 live share kaydı silinmiyor) | — |

Bu matrisle hiçbir projeye veya kontrole PASS/KABUL verilmez.

### 6.3 Bulgular

Kimlik önekleri: **F-C** Core, **F-R** CRM, **F-S** Stand, **F-B** tarayıcı/dış bağlantı, **F-V** sunucu/DB, **F-G** tedarik zinciri/CI. F-001…F-005 önceki kayıtlardır. Risk P0–P2. Her satırda "Eksik" alanı kapanış için gereken kanıttır.

#### Öncelik 1 — servis kimliği

| Kimlik / SEC | Akış | Kod | İzole kanıt | Canlı yapılandırma | Risk / durum | Eksik |
|---|---|---|---|---|---|---|
| **F-C7** SEC-04/01 | Core `GET /organizations/{id}/lifecycle-snapshot`, `POST /organizations/{id}/retained-audit-evidence/purge`; çağıran: CRM `integrations/kyrox_core/lifecycle.py`, `audit_retention.py` (organization-closure: terminal closure, artifact cleanup, evidence retention) | `product_lifecycle_routes.py:require_product_lifecycle_credential` ayar yoksa `config.py` public varsayılanını karşılaştırıyordu; CRM `config.py:kyrox_core_product_lifecycle_token` aynı public varsayılanı gönderiyor | Public değerle snapshot 200 (Core `3051d2e`, TestClient) | Core ve CRM süreç ortamında ve `.env`'lerde lifecycle anahtarı **yok** → her iki taraf public değerle çalışıyor (CANLI yapılandırma). Core `/kyrox-core/` nginx üzerinden dışa açık. nginx access log'unda bu yollara çağrı yok; CRM çağrıları iç adresten gidebileceği için bu, çağrı olmadığının kanıtı değil. | **P1 — KABUL EDİLDİ (canlı), E-014 (§6.7)**: token canlıda ayarlandı, #45 merge `91f9d41` ve deploy edildi; eski varsayılan/yanlış/eksik 401, geçerli kabul. [kyrox-studio/kyrox-core#45](https://github.com/kyrox-studio/kyrox-core/pull/45) head `9446556`: varsayılan yok; tanımsız/kısa/`dev-insecure` → 503, yanlış → 401; 9 yeni test main'de FAIL, dalda PASS; tam suite 469 passed (yerel) ve CI [run 40193613457](https://github.com/kyrox-studio/kyrox-core/actions/runs/40193613457) 469 passed (lint adımı yine "ruff not installed", F-005). | CRM tarafında public varsayılanı göndermeyi reddetme (ayrı CRM işi); token tanımsızken 503 canlıda denenmedi (CI'da kanıtlı) |

#### Core — kimlik, oturum, yetki (SHA `3051d2e`; izole = gerçek login/JWT/session zinciri, SQLite)

| Kimlik / SEC | Akış | Kod | İzole kanıt | Canlı | Risk | Eksik |
|---|---|---|---|---|---|---|
| F-C3 SEC-03 | Refresh rotasyonu | `refresh_session.py:RefreshSessionUseCase.execute` reuse'da iptal edip hata fırlatıyor; `db/session.py:get_db` rollback → iptal kalıcı değil (org-suspended dalı da aynı) | Eski refresh 401 ama session aktif, `reuse_detected_at` boş, yeni çift çalışıyor; güvenli beklenti testi FAIL | — | P1 — **CANLIDA DEPLOY (E-016)**: kod + CI testi; canlı davranış testi veri değiştireceği için yapılmadı | Canlıda refresh tekrar kullanımı senaryosu (izole ortamda Postgres ile tekrar) |
| F-C4 SEC-03 | Yönetici parola değişimi/pasifleştirme/silme | `user_management/routes.py:update_user/delete_user` `RevokeAllUserSessionsUseCase` çağırmıyor | Parola değişiminden sonra eski access ve refresh çalışıyor; pasif kullanıcıya `user-management/context` ve `GET /organizations` 200 | — | P1 — **CANLIDA DEPLOY (E-016)**: kod + CI testi; canlı davranış testi veri değiştireceği için yapılmadı | Canlıda pasifleştirme sonrası token düşmesi (kontrollü test hesabıyla) |
| F-C4b SEC-03/01 | Super Admin istisnası | `guards.py:is_super_admin`, `authorization_service.check_permission` bayrağı status/deleted bakmadan okuyor | Pasif+silinmiş Super Admin'in eski token'ı `/system/settings` ve yabancı org listesi 200 | — | P1 — **CANLIDA DEPLOY (E-016)** #46 (oturum iptaliyle; aktif Super Admin bypass semantiği değişmedi). **Karar (kullanıcı, 2026-10-25):** kullanıcı pasife alındığı anda token düşer; Super Admin dahil. `PRODUCT_INTEGRATION_GUIDE` §5'teki "account authorization state" ifadesi ADR-0003'e uyacak şekilde düzeltildi | Canlıda pasif Super Admin token düşmesi (kontrollü test hesabıyla) |
| F-C1 SEC-01 | Kullanıcı yönetimi | `update_user` korumalı rol (OrganizationAdmin) taşıyan hedefin parola/e-posta/status'ünü `identity.users.update` ile değiştirmeye izin veriyor | Editor, OrgAdmin parolasını değiştirip onunla giriş yaptı ve pasifleştirdi | — | P1 | **Ürün kararı** (§6.5) |
| F-C2 SEC-01 | Rol yönetimi | `update_organization_role_permissions` kendi rolüne yetki eklemeye izin veriyor; `identity.roles.assign` hiçbir route'ta istenmiyor | Editor rolüne `users.delete` ekleyip OrgAdmin'i sildi; `roles.assign`'sız rol atadı | — | P1 | **Ürün kararı** |
| F-C5 SEC-04 | Yapılandırma | `JWT_SECRET_KEY`, `CORE_IDENTITY_ACTION_TOKEN_SECRET_KEY` public varsayılanlı; `ENVIRONMENT` hiçbir yerde okunmuyor; başlangıç guard'ı yok; access TTL 15 gün (ADR 15 dk) | Varsayılan sırlar testle gösterildi | Core `ENVIRONMENT=development` (davranış değiştirmiyor); JWT anahtarı `.env`'de tanımlı (CANLI, anahtar adı) | P1 | Canlı JWT ve action-token değerlerinin public varsayılandan farklı olduğu hash ile doğrulanmadı |
| F-C8 SEC-04 | JWT paylaşımı | HS256, `iss/aud/kid` yok; session süresiz | Anahtarla görülen bir `sid`'den 10 yıllık Super Admin token'ı 200; claim eklemek yetki vermiyor | HS256 anahtarı Core/CRM/Stand'de aynı (CANLI, hash) | P1 | **Mimari karar** (ayrı anahtar / asimetrik + JWKS) |
| F-C9 SEC-01 | Son Super Admin (ADR-0003, kullanıcı kararıyla teyit 2026-10-25) | **Düzeltme:** ilk denetim notu ("kodda koruma yok") eksikti. Migration `20260814_0042`/`0045` PostgreSQL tetikleyicisi `trg_identity_users_protect_last_super_admin` (advisory lock ile) son aktif Super Admin'in pasif/yetkisiz/silinmesini engelliyor. API katmanında kontrol yoktu; tetikleyici hatası 500 dönüyordu | Denetim testi SQLite'ta koştu (tetikleyici yok), bu yüzden kaldırma başarılı göründü. #46: API 409 + 3 test (main'de FAIL, dalda PASS) | Canlı DB'de tetikleyici var (CANLI metadata) | P2 — **CANLIDA DEPLOY (E-016)**: API 409 kodu yüklü, canlı tetikleyici etkin (	genabled=O), aktif Super Admin sayısı ≥1 | Canlıda deneme yapılmadı (başarısızlık son Super Admin'i kapatırdı); Postgres'te tetikleyici + API birlikte izole test |
| F-C10 SEC-20 | JWT çözme | `decode_access_token` eksik `exp`'te KeyError | Geçerli imzalı eksik claim → 500 (erişim açılmıyor) | — | P2 — **KABUL EDİLDİ (canlı), E-016**: deploy öncesi 6 eksik-claim durumu canlıda 500, sonrası 401 | — |
| F-C11 SEC-17/20 | OpenAPI | `create_app` docs/openapi kapatmıyor | 200 | `/kyrox-core/docs`, `openapi.json` dışa 200 (CANLI) | P2 | — |
| F-C6 SEC-03/16 | Login/signup | Status kontrolü parola doğrulamasından önce | Pasif 403 / bilinmeyen 401; signup 409 | — | P2 | Zamanlama farkı ölçülmedi |
| F-004 SEC-16 | Login/reset/forgot/change | Rate limit/lockout kodu yok | 25–30 yanlış login sonrası doğru parola 200; forgot 20×202 | nginx'te `limit_req` yok (CANLI) → "upstream sınır" ihtimali çürüdü | P1 | **Ürün kararı** (limit/lockout dengesi) |

#### CRM (SHA `8f44468e`; izole = mock Core + CRM test JWT)

| Kimlik / SEC | Akış | Kod | İzole kanıt | Canlı | Risk | Eksik |
|---|---|---|---|---|---|---|
| F-B1 SEC-14 | Teklif şablonu önizleme | `frontend/src/pages/QuoteEditorPage.tsx` iframe `srcDoc` sandbox'sız; `quotes/api/routes.py:_render` `source_code`'u sanitize etmeden döndürüyor; access token localStorage'da | Render çıktısında localStorage okuyan script aynen döndü | — | P1 | Gerçek tarayıcı kanıtı; şablon düzenleme yetkisinin kimlerde olduğu |
| F-B2 SEC-21 | Website zenginleştirme / scraper | `website_http_fetcher`, `website_validator` IP kontrolü yok, `follow_redirects=True`; scraper yalnız şema kontrolü | 127.0.0.1 içeriği çekildi, başka porta 302 izlendi; 169.254.169.254, [::1], 10.x, `0.0.0.0:5432` filtreden geçti | — | P1 | Playwright/scraper yolunda izole tekrar |
| F-R1 SEC-07/08 | Teklif logosu yükleme | `quote_templates/api/routes.py` yalnız istemci Content-Type'ına bakıyor; tam okuma sonra boyut | `image/png` etiketli HTML ve script'li SVG 200 | Uygulama `nosniff` göndermiyor (CANLI başlık) | P1 | — |
| F-B3 SEC-19 | `/auth/logout` | CSRF başlığı istemiyor | Yabancı Origin, gövdesiz istek 204, Core oturumu iptal | SameSite=lax; `*.kyrox.studio` (WordPress dahil) aynı site | P2 | — |
| F-B4 SEC-14 | E-posta şablonları | Jinja `autoescape=False` (sandbox SSTI'yi engelliyor) | Değişkenle HTML/link enjeksiyonu | — | P2 | — |
| F-R4 SEC-20/04 | Hata işleyici | `main.py` model-validator hatası 500; log'a tam gövde | 500 (`ValueError not JSON serializable`); JSON olmayan gövde 500 | — | P2 | SMTP/provider parolasının log'a düştüğünün canlı kanıtı yok (log okunmadı) |
| F-R3 SEC-02/20 | `compare-import`, `PATCH /email-accounts/{id}`, operations `start/retry/retry-failed` | Permission'dan önce kayıt araması | Yabancı UUID varlığı 404/403 farkından öğreniliyor; veri dönmüyor | — | P2 | — |
| F-R2 SEC-01/03 | Dev bypass | `APP_ENV` varsayılanı `development`; `APP_ENV` tanımsız + `FAIR_CRM_DEV_BYPASS_CORE=true` iken token'sız istek kabul | Kanıtlandı; production+bypass → başlamıyor | Canlı `APP_ENV=production`, `FAIR_CRM_DEV_BYPASS_CORE=false` (CANLI, `.env` ve süreç ortamı) | P2 (canlıda etkisiz) | Varsayılanı güvenli yapan düzeltme |
| F-R5 SEC-02/16 | MailerSend webhook | İmzadan önce kayıt araması ve Core çağrısı | Kod incelemesi | — | P2 | İzole negatif |

#### Stand (SHA `ac18bc1a`; izole = mock yetki + yerel testler, 22 yeni test)

| Kimlik / SEC | Akış | Kod | İzole kanıt | Canlı | Risk | Eksik |
|---|---|---|---|---|---|---|
| F-001 SEC-07/14 | Asset yükleme/indirme | `image_optimize.py:optimize_uploaded_image` SVG'yi değiştirmeden saklıyor; `download_project_asset` `image/svg+xml` + `inline` | Script'li SVG aynen `inline` döndü (svg+xml, boş MIME, charset varyantı) | CSP ve nosniff yok (CANLI); fuar.kyrox.studio CRM ile aynı origin | **P1 — doğrulandı (izole)** | Gerçek tarayıcıda "yeni sekmede aç" istismarı; doğrudan URL Bearer istediği için 401 |
| F-002 SEC-07/16 | Görsel optimize | `MAX_IMAGE_PIXELS=None`, `load()` küçültmeden önce; decode async route'ta olay döngüsünü blokluyor | 439 KB 20000×20000 PNG 500 MB tavanında MemoryError; 48 KB → 83 MB tepe | Servis root, `MemoryMax` yok, ~1,35 GB boş RAM (CANLI) | **P1 — doğrulandı (izole)** | Pillow 13 ile tekrar (F-G2) |
| F-003 SEC-07/16 | Upload okuma | `await file.read()` tam okuma, sınır sonra | 20 MB tam okunup 400 | nginx `client_max_body_size 256m` (CANLI) | **P1 — doğrulandı (izole)** | — |
| F-S4 SEC-02/20 | `POST /projects` | IntegrityError yakalanmıyor | Yabancı proje id'siyle 500 (varlık ifşası); PUT doğru 404 | — | P2 | — |
| F-S5 SEC-03 | Core oturumu | Olumlu izin önbelleği 10 sn | İptalden sonra 10 sn 200; Core 401 → 401, Core hata → 503 | — | P2 | exp'siz JWT Stand'de kabul; Core'un son kararı doğrulanmadı |
| F-S6 SEC-08 | Proje payload | Sunucu boyut sınırı yok; `max_image_upload_mb` üst sınırsız; `customerId` CRM'de doğrulanmıyor | 3 MB payload 201 | — | P2 | — |
| F-S7 SEC-22/16 | Live share | Süresi dolan kayıtlar silinmiyor; TURN tanımlanırsa credential kimliksiz izleyiciye gider | Kod + yerel | TURN tanımlı değil (CANLI) | P2 | — |

#### Sunucu, DB ve yapılandırma (canlı salt okunur)

| Kimlik / SEC | Kod / şablon | Canlı yapılandırma | Risk | Eksik |
|---|---|---|---|---|
| **F-V1** SEC-05 | Compose/bootstrap tek `postgres` rolü; `bootstrap-server.sh` uzak SUPERUSER rolü ve `ufw allow 15432` oluşturabiliyor | Core, CRM, Stand, migration ve yedek aynı `postgres` rolü (SUPERUSER+BYPASSRLS, tek giriş rolü); 5 URL'de aynı parola; CRM üç DB'nin superuser URL'sini taşıyor. 15432 kuralı ve uzak rol şu an yok. 5432 yalnız 127.0.0.1 | **P0** | Rol ayrımı için mimari plan (§6.6) |
| F-V2 SEC-04 | Deploy şablonu `DEPLOY_SERVICE_USER` root'a düşüyor | 4 servis `User=root`, sertleştirme yok; Stand `project-assets` 0777 | P1 | — |
| F-V3 SEC-04 | — | Core/CRM/Stand backend `.env` 0644; HS256 anahtarı 3 serviste aynı. **E-015:** Core ve CRM `.env` 0600 root:root, nobody okuyamıyor (canlı doğrulandı) | P1 — Core/CRM kısmı düzeltildi | Stand `.env` izni; HS256 ayrımı (F-C8) |
| F-V4 SEC-04/11 | — | SSH `PermitRootLogin yes`, `PasswordAuthentication yes`, fail2ban yok; 24 saatte 11 başarısız deneme; son 7 gün tüm başarılı girişler publickey | P1 | — |
| F-V5 SEC-12/04 | `kyrox-backup.sh` şifrelemesiz | Drive'a şifresiz: 3 PG + MariaDB dump, tüm `.env`'ler, 4 TLS privkey; rclone `scope=drive` (tam Drive), değişmez kopya yok | P1 | — |
| F-V6 SEC-12 | `kyrox-restore.sh` `docker rm -f` `-v`'siz | 16.10 tatbikatından 6 sahipsiz volume (~3,1 GB); içerik okunmadı, ilişki zaman/dizin yapısından | P1 | Volume içeriğinin silinmeden önce sahibince doğrulanması |
| F-V7 SEC-12 | Tatbikat yalnız tablo sayısı karşılaştırıyor | RPO/RTO kararı yok, tatbikat zamanlanmamış (son 16.10, elle) | P1 | **Karar** |
| F-V8 SEC-17 | Uygulama ve nginx şablonunda başlık yok | HSTS, CSP, nosniff, frame-ancestors, Referrer-Policy yok; `Server: nginx/1.24.0` | P2 | Meşru gömme/paylaşma akışlarıyla uyum testi |
| F-V9 SEC-13 | — | PostgreSQL bağlantı/DDL logu ve auditd yok | P2 | — |
| F-V10 | — | Kernel yeniden başlatma bekliyor (142 çalışıyor, 147 kurulu); eski `kyrox-postgres-dev` + 1,1 GB volume | P2 | Silme kararı kullanıcıda |

Olumlu canlı gözlemler: 5432/8000/8001/8002/8090 yalnız 127.0.0.1; ufw/iptables DROP; HTTP→HTTPS 301; TLS 1.0/1.1 ret; sertifikalar 20.12.2026 ve 13.01.2027'ye kadar; son yedek 25.10 03:30, Drive kopyası `rclone check` ile doğrulandı; uvicorn yalnız 127.0.0.1'e XFF güveniyor; WordPress `.env` 0600, wp-login `limit_req`, xmlrpc 403.

#### Tedarik zinciri, CI ve toparlanma

| Kimlik / SEC | Kod / CI | İzole / tarama | Canlı / GitHub | Risk | Eksik |
|---|---|---|---|---|---|
| F-G2 SEC-09 | Sürüm kilidi yok; deploy `pip install -r` `--upgrade`'siz (`common.sh`) → canlı eski kalıyor, CI farklı sürüm test ediyor | `pip-audit` canlı venv: Core/CRM PyJWT 2.13.0 (15 kayıt, 1 kritik, 6 yüksek; düzeltme 2.14/2.15 minor); Stand Pillow 12.3.0 (37 kayıt, 4 kritik, 22 yüksek; düzeltme 13.0 **major**); certifi kritik (3 venv). Taze kurulum 0 açık | Canlı venv sürümleri | P1 | Pillow major için onay; kilit dosyası kararı |
| F-G1 SEC-04/10 | — | Git geçmişi regex taraması: gerçek sır yok (51 aday tek tek; gitleaks/trufflehog kurulmadı); canlı dist'te sır yok | 4 public depoda secret scanning ve push protection kapalı; Core/CRM/Stand Dependabot alerts kapalı; org 2FA zorunlu değil | P1 | Ayar değişikliği kullanıcı onayıyla |
| F-G3 SEC-10 | Python bağımlılık taraması hiçbir CI'da yok; CRM CI'da npm audit yok (**E-006'daki "CRM development-standard.yml audit" ifadesi yanlış**); `prod-path-e2e.yml` `head_ref` shell'e giriyor; action'lar SHA ile sabit değil; çoğu workflow'da `permissions` yok (repo varsayılanı read) | — | Web main ruleset'inde PR/zorunlu kontrol yok | P1/P2 | Branch protection/required check tüm depolarda |
| F-005 SEC-09/10 | Core'da ruff atlanıyor, dependency audit yok | Bugünkü CI log'unda yine "ruff not installed" | — | P1 | — |

Olumlu: gerçek `kyrox-backup.sh` + `kyrox-restore.sh --deneme` izole Docker'da sentetik veri ve Core şemasıyla tam restore: şema, 6.274 satır ve grant birebir; bozuk dump sha256 ve `pg_restore` ile yakalandı. Canlı son yedeğin sha256'sı OK, `pg_restore --list` üç dump'ı okuyor. CRM npm açıkları kapandı (#359/#360); Stand npm 0.

### 6.4 Doğrulanmayı bekleyen riskler

| Kimlik | Risk | Neden doğrulanmadı | Nasıl doğrulanır |
|---|---|---|---|
| R-1 SEC-01 | Canlı Core DB'de `trg_identity_users_default_owner_super_admin` var (CANLI metadata): `dev@example.com` ile eklenen satır Super Admin olur | Core API yollarında rol atama tetikleyicisi satırı engelledi; CRM/Stand seed yolları canlıda incelenmedi | Seed/bootstrap yollarının canlıda çalışıp çalışmadığı; tetikleyicinin kaldırılması (karar) |
| R-2 SEC-03 | Access TTL 15 gün, session süresiz, her login önceki oturumları kapatıyor (ADR: 15 dk, çok cihaz) | Ürün kararı | Karar sonrası test |
| R-3 SEC-13 | Core `audit-events` permission istemiyor; ip/user_agent istemciden | Ürünlerin çağrı biçimi doğrulanmadı | Çağıran envanteri |
| R-CRM-1 SEC-02 | `customer_field_grouping.py:_load_fair_metadata` org filtresiz tüm tenant katılım satırlarını okuyor | Sızıntı görülmedi; iki org'lu Postgres testi gerekiyor | İzole Postgres negatif testi |
| R-CRM-2 SEC-22 | Job/run ID'leri için yabancı-ID testi eksik | Zaman | İzole negatif |
| R-ROL | Normal kullanıcı / OrganizationAdmin rol matrisi CRM ve Stand'de gerçek Core ile koşulmadı | Testler mock | Core + ürün birlikte izole ortamda |
| R-XSS | F-001, F-B1 gerçek tarayıcıda denenmedi | Zaman | Playwright, izole |
| R-EXP | Stand exp'siz JWT kabul ediyor; Core son karar | Core ile birlikte denenmedi | Gerçek Core zinciri |
| R-B | SMTP host ile port tarama, export ZIP belleği, WS mesaj boyutu, upload/export limitleri | Zaman | İzole |

### 6.5 Kapsam dışı

- Canlıda istismar, rate-limit, yük, upload, restore ve DB satır okuma — kural gereği yapılmadı.
- Docker imajları, OS paketleri, WordPress eklentileri, rclone 1.60.1 için zafiyet tarayıcısı — çalıştırılmadı.
- Canlı veriyle ayrı ortamda restore — canlıdan veri çekilmedi; yalnız sentetik veri ve canlı yedeğin metadata'sı.
- kyrox-web (WordPress) yalnız yapılandırma düzeyinde; uygulama içi güvenlik testi yok.
- OWASP ASVS eşlemesi — yapılmadı.

### 6.6 Öncelikli düzeltme sırası ve kararlar

**Kod (her biri ayrı dal, mevcut testler gevşetilmeden, regresyon testiyle):**
1. F-C7 lifecycle fail-closed — Core [kyrox-studio/kyrox-core#45](https://github.com/kyrox-studio/kyrox-core/pull/45) merge ve canlıda (E-014). Kalan: CRM'in public varsayılanı göndermeyi reddetmesi.
2. Core oturum (güvenli kapsam): F-C3, F-C4, F-C4b, F-C9, F-C10 — [kyrox-studio/kyrox-core#46](https://github.com/kyrox-studio/kyrox-core/pull/46) merge `da11f2c`, canlıda (E-016); head `9b4a08a`: 17 yeni test (production benzeri commit/rollback) main'de 16 FAIL, dalda PASS; F-C9 için API 409 eklendi; tam suite 486 passed (yerel, main birleştirilmiş). Kalan: F-C11 (canlı adım 4 veya ayar).
3. CRM: F-B1 (sandbox + sanitize), F-B2 (IP/yönlendirme allowlist), F-R1, F-B3, F-B4, F-R4, F-R3, F-R2 varsayılanı.
4. Stand: F-001 (attachment + nosniff + CSP sandbox veya rasterize — meşru logo/görsel işlevini koruyarak), F-002 (piksel sınırı + thread'de decode), F-003 (akışlı okuma sınırı), F-S4, F-S5.
5. CI: F-G3/F-005 (pip-audit, ruff kurulumu, `permissions`, `head_ref` env üzerinden), PyJWT yükseltmesi (minor).

**Ürün / mimari kararı gerekenler (kod başlamadan karar):** F-C1 (korumalı rollü kullanıcıyı kim düzenler), F-C2 (sahip olunmayan yetkiyi verme yasağı ve `roles.assign`), F-C8 (servis başına anahtar veya asimetrik imza + JWKS), F-004 (limit/lockout politikası), R-2 (TTL ve çok cihaz), F-V7 (RPO/RTO), Pillow 13 (major), F-V1 (DB rol ayrımı planı), R-1 (dev tetikleyicisinin kaldırılması).

**Canlı yapılandırma — aşamalı, her adım ayrı onayla, tek seferde toplu uygulanmaz:**

| Adım | Değişiklik | Ön koşul | Doğrulama | Geri dönüş |
|---|---|---|---|---|
| 1 | Core `CORE_PRODUCT_LIFECYCLE_TOKEN` ve CRM `KYROX_CORE_PRODUCT_LIFECYCLE_TOKEN` aynı güçlü değer; `.env` yedeği; iki servis restart | Yok (mevcut kodla uyumlu: iki taraf aynı değeri kullanır) | Sağlık 200; CRM→Core lifecycle çağrısı (iç adres) 200; public değerle 401 | **UYGULANDI (E-014).** Varsayılana dönülmez; sorun olursa token iki tarafta birlikte yeniden üretilir |
| 2 | PR #45 deploy | Adım 1 | Token'sız/yanlış 401, doğru 200 | **UYGULANDI (E-014).** Önceki commit'e deploy (token kalır) |
| 3 | `.env` dosyaları 0600, sahibi servis kullanıcısı; `project-assets` 0750 | Servis kullanıcısı kararı (F-V2 ile birlikte) | Servisler başlar, upload çalışır | **Kısmen UYGULANDI (E-015):** Core/CRM `.env` 0600 root:root (servisler root); Stand `.env` ve `project-assets` açık. Geri dönüş: `chmod` önceki değer (önerilmez) |
| 4 | nginx: `/kyrox-core/docs`, `openapi.json`, `redoc` kapat; `server_tokens off`; nosniff + Referrer-Policy | `nginx -t` | curl -I başlıklar; docs 404 | Yapılandırma yedeği + `nginx -s reload` |
| 5 | nginx `limit_req` (login/reset/forgot/signup) | F-004 kararı (eşikler) | İzole önce; canlıda yalnız başlık/yapılandırma kontrolü | Yedek + reload |
| 6 | SSH: `PasswordAuthentication no`, ardından `PermitRootLogin prohibit-password`; fail2ban | Anahtarla ikinci oturum açık tutulur; konsol erişimi teyitli | Yeni oturumla anahtar girişi | Açık oturumdan `sshd_config` yedeği + reload |
| 7 | Servisleri root olmayan kullanıcıya taşıma + systemd sertleştirme + `MemoryMax` | Dosya sahiplikleri planı | Servis sağlığı, upload/export | Unit drop-in sil + daemon-reload |
| 8 | JWT anahtarı ayrımı | F-C8 kararı ve kod desteği | Tüm oturumlar yeniden giriş ister (bildirim gerekli) | Eski anahtara dönüş |
| 9 | DB rolleri: servis başına ayrı runtime rolü, ayrı migration ve yedek rolü; superuser yalnız bakım | F-V1 planı, izole provası | Her servis kendi DB'sine sınırlı; migration ayrı rolle | Eski URL'lere dönüş (rol silinmeden önce) |
| 10 | Yedek şifreleme (rclone crypt veya age), Drive yetkisi `drive.file`, değişmez/ayrı kopya; restore betiğine `-v` | F-V7 kararı; anahtar saklama yeri | İzole restore şifreli yedekten | Eski remote paralel tutulur |
| 11 | 6 sahipsiz restore volume ve eski `kyrox-postgres-dev` | Sahibin silme onayı | — | Silme geri alınamaz; önce yedek |

Adımlar 6, 8, 9, 10 erişim/kimlik/veri kaybı riski taşıdığı için birbirinden ayrı pencerelerde uygulanır.

Adım 1–2 2026-10-25'te kullanıcı onayıyla uygulandı (E-014). Geri dönüş kuralı: sabit varsayılan değere **dönülmez**; `.env` yedekleri token içermediği için geri yüklenmez. Sorun olursa token iki tarafta birlikte yeniden üretilir.

Kapanış için: düzeltme commit'i, kalıcı negatif test, CI kanıtı ve gerektiğinde canlı doğrulama aynı kayda bağlanır.

### 6.7 Çalıştırılan canlı değişiklik ve kanıt E-014 — lifecycle token (F-C7)

- **Kapsam:** Yalnız Core `CORE_PRODUCT_LIFECYCLE_TOKEN` ve CRM `KYROX_CORE_PRODUCT_LIFECYCLE_TOKEN` eklendi, `kyrox-core`, `fair-crm-backend` ve `fair-crm-mail-worker` sırayla yeniden başlatıldı, Core [kyrox-studio/kyrox-core#45](https://github.com/kyrox-studio/kyrox-core/pull/45) deploy edildi. nginx, systemd unit, dosya izinleri, diğer sırlar, Stand ve CRM kodu değişmedi. Toplu `deploy-all.sh` kullanılmadı (nginx/systemd/diğer repo/dev seed adımları içerdiği için).
- **Token:** Sunucuda `secrets.token_urlsafe(48)` ile üretildi, 64 karakter, iki `.env`'e doğrudan yazıldı. Değer hiçbir çıktıya, komut argümanına, belgeye veya log'a yazılmadı. Kanıt yalnız: uzunluk 64, iki dosyadaki değerin sha256 ilk 8 karakteri eşit, her dosyada anahtar bir kez. Geçerli token doğrulamada `/dev/shm`'deki 0600 geçici header dosyasından verildi ve silindi.
- **Yedek:** `.env` yedekleri 0600: Core yedeği `/root/kyrox-env-backups/` (0700) altında. Core çalışma kopyasında izlenmeyen dosya bıraktığı için depo dışına taşındı. CRM yedeği CRM `backend/` altında (git ignore).
- **Doğrulama yöntemi:** Sunucu içinden `127.0.0.1:8000`, her koşuda yeni rastgele (var olmayan) organization UUID'si. Snapshot salt okunur. Purge geçerli token ile **çağrılmadı**; ret durumları işleyiciye ulaşmadan 401 döner. Veri okunmadı, silinmedi.

| Kontrol | Önce (Core `0a4fe17`, token yok) | Token sonrası (Core `0a4fe17`) | #45 deploy sonrası (Core `91f9d41`) |
|---|---|---|---|
| snapshot, header yok | 401 | 401 | 401 |
| snapshot, yanlış token | 401 | 401 | 401 |
| snapshot, eski sabit varsayılan | **404 (kabul edildi)** | 401 | 401 |
| snapshot, geçerli token | — | 404 (kimlik geçti, org yok) | 404 |
| purge, header yok / yanlış | 401 / 401 | 401 / 401 | 401 / 401 |
| purge, eski sabit varsayılan | **409 (kabul edildi, işleyiciye ulaştı)** | 401 | 401 |
| CRM istemcisi (`KyroxCoreLifecycleClient`) → Core snapshot | 404, token uzunluk 46, `dev-insecure` | 404, uzunluk 64, `dev-insecure` değil | 404, uzunluk 64 |
| Dış yol `https://fuar.kyrox.studio/kyrox-core/…/lifecycle-snapshot`, eski varsayılan / header yok | — | — | 401 / 401 |
| Servis sağlığı | — | Core health 200; 4 servis active | Core health 200, active |

- **"Önce" sütunundaki purge 409'u:** Açığın canlıda var olduğunun doğrudan kanıtıdır. İstek eski varsayılan token'la yetki kontrolünü geçip purge işleyicisine ulaştı ve rastgele, var olmayan organization için ön koşul hatası (409) döndü. Hiçbir kayıt etkilenmedi. Bu ölçüm, sonrası kontrolüyle aynı betikle alındı; gelecekte varsayılan token'ın kabul edildiği durumlarda purge isteği gönderilmemeli.
- **Deploy:** `/opt/kyrox/kyrox-core` temiz çalışma kopyasında `0a4fe17` → `91f9d41` fast-forward (8 dosya: kod, test, `.env.example`, SEC kaydı; bağımlılık veya migration değişikliği yok), yalnız `kyrox-core` restart. Merge commit `91f9d415d7bb86ea07dd431478ab76e78b0c94b0`, PR head `9446556` CI [run 40193613457](https://github.com/kyrox-studio/kyrox-core/actions/runs/40193613457) 469 passed.
- **Durum:** F-C7 → **KABUL EDİLDİ (canlı)**: kod (fail-closed), negatif testler (CI), canlı yapılandırma ve canlı ret/kabul doğrulaması aynı kayıtta.
- **Canlıda doğrulanmayanlar:** Token tanımsızken 503 davranışı canlıda denenmedi (token kaldırılmadı); CI testleriyle kanıtlı. CRM organization-closure akışının gerçek bir kapanışla uçtan uca çalışması denenmedi (veri değiştirir).
- **Kalan risk (E-014 anında):** Token, 0644 izinli `.env` dosyalarına yazıldı; bu nedenle E-015'te izinler düzeltildi ve token yenilendi. CRM token tanımsızsa public varsayılanı göndermeye hazır kod taşıyordu (Core artık reddediyor); CRM tarafı düzeltmesi E-015 sonrası ayrı PR.

### 6.8 Çalıştırılan canlı değişiklik ve kanıt E-015 — Core/CRM `.env` izinleri ve lifecycle token rotasyonu (F-V3 kısmi, F-C7)

- **Kapsam:** Yalnız Core ve CRM backend `.env` sahiplik/izinleri, iki `.env` yedeğinin yeri ve lifecycle token'ın iki tarafta birlikte yenilenmesi. Diğer sırlar (JWT, DB, SMTP vb.) değiştirilmedi. Stand `.env`'i kapsam dışı (izni değişmedi, F-V3 açık).
- **İnceleme (salt okunur):**
  - Login kabuğu olan tek yerel kullanıcı `root`.
  - `kyrox-core`, `fair-crm-backend`, `fair-crm-mail-worker` ve `fair-stand` servislerinin dördü de `User=root`. `.env`'leri yalnız bu üç Core/CRM birimi `EnvironmentFile` olarak okuyor.
  - Yedek betiği root ile çalışıyor. `docker` grubunda yalnız root var.
  - `/var/backups/kyrox` 0700, arşivler 0600 root:root, `/etc/fair-crm/*` 0600: zaten doğru.
  - Deploy betiği mevcut `.env`'yi korur ve yerinde yazar (`copy_env_if_missing`, `replace_or_append_env_key`); izni geri açan adım yok.
  - CRM frontend `.env`/`.env.production` yalnız derleme zamanı `VITE_*` anahtarları içeriyor (anahtar adları kontrol edildi); build'e gömülü ve sunucu sırrı değil, kapsam dışı.
- **Hedef:** Servisler root çalıştığı için gerekli okuyucu yalnız root: `.env` → `0600 root:root`. Servis kullanıcısı değişikliği F-V2 kararına bağlı, yapılmadı.
- **Değişiklik:**
  - Core ve CRM `backend/.env` için `chown root:root` ve `chmod 600` uygulandı.
  - CRM `.env` yedeği uygulama dizininden `/root/kyrox-env-backups/` (0700) altına taşındı. Core yedeği zaten oradaydı.
  - Yedeklerin ikisi de 0600 ve token içermiyor (token öncesi kopyalar).
  - Git çalışma kopyaları temiz: Core 0, CRM 0 değişiklik.
- **Yetkisiz yerel kullanıcı testi** (`runuser -u nobody`: `test -r` ve 0 bayt `head -c 0` açma denemesi; içerik okunmadı):

| Dosya | Önce | Sonra |
|---|---|---|
| Core `backend/.env` | 644 root:root — nobody **okunabilir / açıldı** | 600 root:root — okunamaz / reddedildi |
| CRM `backend/.env` | 644 root:root — nobody **okunabilir / açıldı** | 600 root:root — okunamaz / reddedildi |
| Core yedeği (`/root/kyrox-env-backups/`) | 600 — okunamaz / reddedildi | 600 — okunamaz / reddedildi (dizin 0700) |
| CRM yedeği | 600 (CRM `backend/` altında) — okunamaz | 600, `/root/kyrox-env-backups/` altında — okunamaz |

- **Rotasyon nedeni:** E-014 token'ı yaklaşık bir saat boyunca 0644 dosyada duruyordu.
- **Rotasyon yöntemi:**
  - Yeni değer sunucuda `secrets.token_urlsafe(48)` ile üretildi (64 karakter). İki dosyadaki tek satır yerinde değiştirildi; sahiplik ve izin korundu, rotasyon sonrası yine 0600 root:root.
  - Kanıt yalnız sha256 ilk 8 karakteri karşılaştırmasıdır: yeni değer iki tarafta eşit ve eskisinden farklı.
  - Eski ve yeni header'lar `/dev/shm`'deki 0600 geçici dosyalardan verildi ve silindi.
  - Servisler sırayla yeniden başlatıldı. Core health 200; dört servis active.
- **Rotasyon doğrulaması:** Sunucu içi, rastgele var olmayan organization UUID'si. Purge geçerli token ile çağrılmadı.

| Kontrol | Sonuç |
|---|---|
| snapshot: eksik / yanlış / sabit varsayılan | 401 / 401 / 401 |
| snapshot: **eski (E-014) token** | **401** |
| snapshot: yeni token | 404 (kimlik geçti, org yok) |
| purge: eksik / yanlış / sabit varsayılan / eski token | 401 / 401 / 401 / 401 |
| CRM istemcisi → Core snapshot | 404; CRM token uzunluğu 64, `dev-insecure` değil |

- **Durum:** F-C7 KABUL EDİLDİ (canlı) korunuyor. F-V3 → Core ve CRM için düzeltildi (canlı doğrulandı); **Stand `.env` 0644 ve HS256 anahtar paylaşımı açık**.
- **Kalan / doğrulanmayan:**
  - Drive'daki şifresiz yedekler (F-V5) eski `.env` kopyalarını içeriyor. Bu kopyalarda lifecycle token yok, ama diğer sırlar var.
  - Servislerin root olmayan kullanıcıya taşınması (F-V2) yapılmadı.
  - Diğer sırların rotasyonu bu işlemde yapılmadı.

### 6.9 Çalıştırılan canlı değişiklik ve kanıt E-016 — Core oturum ve son Super Admin düzeltmeleri (#46)

- **Karar dayanağı:** Kullanıcı 2026-10-25'te teyit etti: kullanıcı pasife alındığı anda token düşer, Super Admin dahil; son Super Admin kendini silemez veya kapatamaz (ADR-0003). `PRODUCT_INTEGRATION_GUIDE` §5 bu karara göre düzeltildi.
- **Kaynak:** [kyrox-studio/kyrox-core#46](https://github.com/kyrox-studio/kyrox-core/pull/46), head `9b4a08ad8930d6696b768a0f1e8929e7ee028360`.
  - CI: [run 40198905185](https://github.com/kyrox-studio/kyrox-core/actions/runs/40198905185) Backend CI success.
  - Merge commit: `da11f2cb79877bc692efa9923301686bca493597`.
  - Yerel: 486 passed; yeni 17 test main'de 16 FAIL.
- **Deploy:** Yalnız Core.
  - `/opt/kyrox/kyrox-core` temiz çalışma kopyasında `91f9d41` → `da11f2c` fast-forward. Değişen 7 dosya kod, test ve SEC kaydı; bağımlılık veya migration değişikliği yok.
  - Yalnız `kyrox-core` yeniden başlatıldı. Health 200; `kyrox-core`, `fair-crm-backend`, `fair-crm-mail-worker`, `fair-stand` active.
  - nginx, systemd, `.env`, diğer depolar değişmedi.
- **Doğrulama yöntemi (veri değiştirmeden):**
  - Sunucu içinden `127.0.0.1:8000`.
  - Sahte token'lar sunucuda, mevcut imza anahtarıyla, rastgele ve var olmayan kullanıcı/oturum kimlikleriyle imzalandı. Anahtar ve token değerleri yazdırılmadı. Hiçbir oturum açılmadı, kullanıcı değiştirilmedi.

| Kontrol | Önce (`91f9d41`) | Sonra (`da11f2c`) |
|---|---|---|
| Geçerli imzalı token, `exp` / `iat` / `sid` / `jti` / `email` / `sub` eksik (F-C10) | **500** (6/6) | 401 (6/6) |
| Geçerli imzalı, tüm claim'ler, var olmayan oturum | 401 | 401 |
| Geçerli imzalı, bozuk `sub` | 401 | 401 |
| Yanlış anahtarla imzalı | 401 | 401 |
| `/system/settings` (Super Admin ucu), var olmayan oturum | 401 | 401 |
| `/auth/refresh`, bilinmeyen token | 401 | 401 |
| `kyrox-core` log, restart sonrası Traceback/ERROR | (önce ölçümündeki 500'ler log'da) | 0 / 0 |
| Lifecycle regresyonu (E-014/E-015 tablosu) | — | aynı: varsayılan/yanlış/eksik 401, geçerli 404, CRM istemcisi 404 |

- **Çalışan kodda yüklü korumalar (dosya araması):**
  - Refresh reddinden önce iptal kalıcılığı (`_persist_revocation_before_rejecting` 2 dal; iki composition root'ta `persist_revocation=db.commit`).
  - `update_user` / `delete_user` oturum iptali (2).
  - Son Super Admin API kontrolü (2).
  - Claim zorunluluğu (1).
  - Süreç başlangıcı dosya zamanıyla aynı (16:52:12).
- **DB durumu:** `trg_identity_users_protect_last_super_admin` etkin (`tgenabled=O`); aktif ve silinmemiş Super Admin sayısı ≥ 1 (sayı yazılmadı).
- **Durum:**
  - F-C10 → **KABUL EDİLDİ (canlı)**.
  - F-C3, F-C4, F-C4b, F-C9 → **CANLIDA DEPLOY**: kod, CI testi ve yüklü kod doğrulandı. Davranış testleri canlıda yapılmadı, çünkü refresh tekrar kullanımı, kullanıcı pasifleştirme ve son Super Admin'i kaldırma denemesi veri değiştirir; sonuncusu başarısız olsaydı platform yöneticisiz kalırdı.
- **Kabul için kalan:** İzole Postgres ortamında Core + tetikleyici ile bu dört senaryo; veya kullanıcı onayıyla canlıda ayrılmış bir test hesabıyla pasifleştirme → token 401 (son Super Admin senaryosu canlıda denenmez).
- **Davranış değişikliği:** Yönetici bir kullanıcının parolasını, e-postasını, durumunu ya da Super Admin bayrağını değiştirdiğinde veya kullanıcıyı sildiğinde, o kullanıcı tüm cihazlardan çıkış yapar.

## 7. Kaynaklar ve bakım

Başlangıç girdisi: kullanıcının 2026-10-25 tarihli `KYROX_SECURITY_CONTROLS_PRIORITY_DRAFT.md` taslağı; taslak ayrı bir bağlayıcı kaynak değildir. Bu belge mevcut onaylı standartlarla ilişkilendirilmiş ana güvenlik kaydıdır.

Dış doğrulama çerçevesi: [OWASP ASVS 5.0.0](https://github.com/OWASP/ASVS/tree/v5.0.0/5.0). ASVS eşlemesi henüz tamamlanmadı; OWASP uyumluluğu veya sertifikası iddia edilmez. Top 10 tek başına bu kontrol kataloğunun kabul ölçütü değildir.

Yeni güvenlik işi burada SEC/F/E kimliğiyle izlenir; proje yol haritası yalnız ilgili kayda link verir. Testler ve CI mekanizmaları sahip kod deposunda kalır. Büyük log/rapor kanıtı kopyalanmaz; commit'e bağlı test/CI/artefakt referansı verilir. Davranış değişince ilgili kanıt yeniden değerlendirilir; eski PASS süresiz güvenlik garantisi değildir.
