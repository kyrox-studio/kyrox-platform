# Ortam sürümleri — yerel, CI, sunucu

**Kural (kullanıcı kararı 2026-11-13):** yerel, CI ve sunucu A'dan Z'ye aynı sürümle çalışır. Esas canlı sunucudur. Sürüm depoda dosyayla kilitlenir; CI ve yerel kurulum oradan okur; `/deploy` fark görürse durur. Sürüm tahmin edilmez, ölçülür.

## Ölçüm — 2026-11-13

Kaynak: `ssh kyrox` (sunucu), `.github/workflows/*.yml` (CI), yerel komutlar (Windows geliştirme makinesi).

| Bileşen | Sunucu (esas) | GitHub CI | Yerel | Durum |
|---|---|---|---|---|
| İşletim sistemi | Ubuntu 24.04.5 LTS | `ubuntu-latest` (sabit değil) | Windows 11 | CI sabitlenecek |
| Node | 22.23.3 | `22` → o günkü en son 22.x (fair-stand, fair-crm, web) | **24.18.0** | Farklı |
| npm | 10.9.9 | Node ile gelen | 11.16.0 | Farklı |
| Python | 3.12.3 (`/usr/bin/python3`, venv'ler) | `3.12` → o günkü en son 3.12.x | 3.12.10 | Ara sürüm farklı |
| Python paketleri | venv'de ne kuruluysa | `pip install -r requirements.txt` (hepsi `>=`) | venv'de ne kuruluysa | **Kilitsiz** — örnek: fair-stand SQLAlchemy sunucuda 2.0.54, yerelde 2.1.4; FastAPI 0.141.1 / 0.143.1 |
| Postgres | 18.6 (Docker `postgres:18-alpine`) | core: `postgres:16-alpine`; fair-crm prod-path: `postgres:16` | 18.4 (`postgres:18-alpine`) | **CI 16** |
| nginx | 1.24.0 (Ubuntu) | yok | yok | Yalnız sunucu |
| Vite | CRM 8.3.8, Stand 8.3.5 | `package-lock.json` ile aynı | aynı (lock) | Kilitli |
| three.js | 0.184.0 | lock | lock | Kilitli |
| Playwright | CRM ön yüz 1.65.0 (lock), CRM venv 1.62.0; Stand E2E `@playwright/test@1.62.1` (`e2e:deps`) | aynı | aynı | Kilitli, ama üç ayrı sürüm |
| Docker | 29.1.3 | — | Docker Desktop | — |
| WordPress / MariaDB | `wordpress:7-php8.4-apache`, `mariadb:11.8` | — | aynı imaj adları | Etiket ara sürüme kilitli değil |

Servisler (sunucu): `kyrox-core`, `fair-crm-backend`, `fair-crm-mail-worker`, `fair-stand` systemd servisi olarak venv Python'uyla (uvicorn) çalışır; Docker'da yalnız `kyrox-db` (Postgres), `kyrox-web`, `kyrox-web-db` vardır.

## Kaymanın kaynakları

1. **`requirements.txt` hepsi `>=`:** her ortam kurulduğu günün en yeni paketini alır.
2. **Deploy kendi kendine yama yapar:** `deploy-all.sh` → `patch_python_dependencies` (`pip-audit --patch`) sunucudaki paketleri depoya yazmadan yükseltir.
3. **CI sürümleri sabit değil:** `ubuntu-latest`, `node-version: 22`, `python-version: "3.12"`.
4. **Postgres CI'da 16:** üretim 18.
5. **Yerel Node 24.**

## Kilitleme planı

| Bileşen | Kilit | Okuyan |
|---|---|---|
| Node | `.nvmrc` = `22.23.3`, `package.json` `engines` + `volta` (npm 10.9.9) | CI `node-version-file`, yerel Volta, deploy kontrolü |
| Python | `.python-version` = `3.12.3` | CI `python-version-file`, yerel, deploy kontrolü |
| Python paketleri | `requirements.lock.txt` (sunucu `pip freeze`, `==`) | CI, deploy (`pip install -r requirements.lock.txt`), yerel venv |
| Postgres | `postgres:18.6-alpine` | CI service imajı, `docker-compose.yml` (yerel ve sunucu), deploy kontrolü |
| WordPress / MariaDB / WP-CLI | `wordpress:7.1.4-php8.4-apache`, `mariadb:11.8.9`, `wordpress:cli-2.12.0-php8.4` | `kyrox-web/sunucu/docker-compose.yml` |
| İşletim sistemi | `runs-on: ubuntu-24.04` | CI |
| npm paketleri | `package-lock.json` + `npm ci` (zaten) | — |

Paket yükseltme ve güvenlik yaması: lock dosyasında PR ile yapılır; sunucuya deploy ile gider.

## Kararlar (kullanıcı, 2026-11-13)

- **Güvenlik yaması:** deploy sunucuda paket yükseltmez. `pip-audit` (yalnız rapor) açık bulursa deploy durur; yama `requirements.lock.txt`'de PR ile yapılır, CI'dan geçer, sonra deploy edilir. Eski karar (SECURITY_PROGRAM F-G2 / E-022: deploy aynı-major yamayı kendisi uygular) bununla değişti.
- **Deploy sürüm kontrolü:** `deploy-all.sh` → `check_runtime_versions` (Node = `.nvmrc`, `python3` = `.python-version`), `check_postgres_version` (container = `docker-compose.yml` etiketi). Fark varsa deploy durur.
- **Yerel makine eşitlenir:** Node Volta ile (`package.json` → `volta` alanı; başka projeler Volta varsayılanı 24.18.0'da kalır), Python 3.12.3 `uv` ile (`uv venv --python 3.12.3`, `uv pip install -r requirements.lock.txt`), Postgres container `postgres:18.6-alpine`.

## Kilit dosyası ayrıntıları

- Kilit, sunucudaki venv'in `pip freeze`'idir (`pip`, `setuptools`, `wheel` hariç).
- Linux'a özgü paket platform koşuluyla yazılır: `uvloop==… ; sys_platform != "win32" …` (uvicorn'un kendi koşulu).
- Windows'ta sistem saat dilimi verisi olmadığı için CRM kilidinde `tzdata==… ; sys_platform == "win32"` vardır (sunucuya kurulmaz).
- İşletim sistemi tek farktır: yerel Windows, sunucu ve CI Ubuntu 24.04. Tam eşitlik gereken test (prod-path E2E) CI'da Ubuntu'da koşar.

## Paket ekleme / yükseltme

1. `requirements.txt` (insan listesi) ve `requirements.lock.txt` aynı PR'da değişir.
2. Kilit, temiz bir 3.12.3 venv'de `pip install -r requirements.txt` + `pip freeze` ile üretilir; platform koşullu satırlar korunur.
3. CI kilitten kurar ve test eder; deploy aynı kilidi sunucuya kurar.
