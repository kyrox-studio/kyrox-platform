# Altı kapsamın odakları

Her kapsam uygulanabilir tüm SEC-01…22 kontrollerini eşleştirir; aşağıdakiler asgari odaklardır. Önceki denetimin bulgularını (SECURITY_PROGRAM §6) okuyup yalnız yeni bulgu veya durum değişikliği raporla.

## 1. Core — kimlik, oturum, yetki
- Route sayımı ve gruplama (kimliksiz, bearer, Super Admin, permission, servis token).
- Normal kullanıcı ve OrganizationAdmin ile: yabancı organization kaynağına doğrudan ID, yetki yükseltme (kendine/başkasına rol veya permission), header/rol adıyla istisna kazanma.
- Oturum yaşam döngüsü: çıkış, parola değişimi/sıfırlama, yönetici pasifleştirme/silme, org askıya alma sonrası eski access/refresh reddi; refresh tekrar kullanımı (production benzeri commit/rollback ile test et).
- Platform Super Admin istisnasını ADR-0003 ile ayrı değerlendir (aktif SA bypass, pasif/silinmiş SA, son aktif SA).
- Servis token'ları: tanımsız/kısa/varsayılan değerle fail-closed mu.

## 2. CRM — organization izolasyonu
- Tüm route'lar × permission dependency × organization filtresi × tenant isolation registry.
- Doğrudan ID, alt kaynak, toplu işlem, export/import, dosya/ek, job/run, mail worker, webhook.
- Raw SQL / kullanıcı girdili sort / subprocess. Dev bypass ve `APP_ENV` varsayılanları.

## 3. Stand — erişim ve dosya
- Proje/revizyon/asset/export/paylaşım, admin/katalog uçları; Core oturumunun tüketimi (önbellek süresi).
- Dosya yükleme: MIME yanıltma, SVG aktif içerik, piksel/bellek sınırı, tam okuma; izole ve süre/bellek sınırlı testlerle.
- Frontend render yolları (innerHTML, SVG, URL).

## 4. DB ve sunucu (salt okunur)
- Dinleyen portlar, güvenlik duvarı, nginx (limit_req, başlıklar, body limit, yollar), systemd `User=`, `.env` izinleri, sertifikalar, SSH.
- PostgreSQL rolleri ve hangi servisin hangi rolle bağlandığı (parola değil), tetikleyiciler, `pg_hba`.
- Yedek: izin, şifreleme, uzak kopya yetkisi, sahipsiz restore volume'ları; deploy ve seed betiklerinin her deploy'da ne değiştirdiği.

## 5. Tarayıcı ve dış bağlantılar
- XSS (sandbox'sız iframe/srcDoc, innerHTML, şablon autoescape), cookie bayrakları, CSRF, CORS, güvenlik başlıkları, açık OpenAPI.
- SSRF: kullanıcı kontrollü URL'ye istek atan yollar; IP/yönlendirme kontrolü (izole yerel dinleyiciyle).
- Rate limit: login/reset/signup/upload/export (yerel/izole).

## 6. Tedarik zinciri ve toparlanma
- Her ekosistemde bağımlılık taraması: `npm audit`, ayrı venv'de `pip-audit`; canlı venv sürümleri ile requirements alt sınırlarını karşılaştır.
- Workflow `permissions`, action sabitleme, `pull_request_target`, script injection; `gh api` ile branch protection, secret scanning, Dependabot (yalnız okuma).
- Git geçmişinde sır taraması (değeri maskele).
- Yedek/restore betiklerinin izole Docker'da sentetik veriyle denenmesi; RPO/RTO kararının varlığı.
