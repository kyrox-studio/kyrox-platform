# FAIR CRM — Dev Restart Guide (auth / RBAC)

**Amaç:** Yerelde Core + Fair CRM kimlik ve yetki hattını gerçek auth ile (bypass kapalı) çalışır hale getirmek ve doğrulamak.
Ön koşullar, kurulum (`setup-dev.ps1`), servis başlatma/durdurma, portlar, migration'lar ve restore sonrası adımlar: [DEV_RUNTIME.md](DEV_RUNTIME.md).

---

## 1 — Dev identity seed (Core RBAC)

`dev-start.ps1` / `reset-dev.ps1` seed'i kendisi çalıştırır (`Invoke-DevCoreIdentitySeed`); `DEV_USER_PASSWORD` ortamda veya `backend/.env` içinde yoksa durur. Elle:

```powershell
cd kyrox-fair-crm
$env:DEV_USER_PASSWORD = "<dev-password>"
python scripts/seed_core_dev_identity.py
```

Seed **yalnız ekler**: organizasyon ve kullanıcı yoksa oluşturur; var olan kullanıcıyı (şifre, e-posta, durum) hiç değiştirmez. Kullanıcı `dev@example.com`, rolü korunan `organization_admin`; organizasyon `FAIR_CRM_DEV_ORGANIZATION_ID` (varsayılan `00000000-0000-4000-8000-000000000010`). Rol matrisi `scripts/fair_crm_role_matrix.py` (`ROLE_MATRIX_VERSION = 25`).

Başarılıysa `scripts/.dev_state.json` yazılır:

```powershell
Get-Content scripts\.dev_state.json | ConvertFrom-Json | Select-Object organization_id, role_matrix_version
```

## 2 — Backend `.env`

```env
APP_ENV=development
FAIR_CRM_DEV_BYPASS_CORE=false
JWT_SECRET_KEY=<Core ile aynı değer>
KYROX_CORE_BASE_URL=http://127.0.0.1:8000
DATABASE_URL=postgresql+psycopg2://postgres:postgres@localhost:5432/fair_crm
```

Gerçek RBAC ve prod-path test için bypass kapalı olmalı. Core `backend/.env` içindeki `JWT_SECRET_KEY` Fair CRM ile aynı olmalı. `KYROX_CORE_BASE_URL` backend → Core sunucu içi HTTP içindir. Frontend `.env` ve tarayıcı yolları: [DEV_RUNTIME.md](DEV_RUNTIME.md#browser--frontend-network-local--server).

## 3 — Bypass kapalı mı?

```powershell
curl -H "Authorization: Bearer dev-bypass" -H "X-Organization-Id: 00000000-0000-4000-8000-000000000010" http://127.0.0.1:8001/api/v1/customers
```

Beklenen **401** veya **403**. 200 dönerse bypass açıktır: `.env` → `false`, `reset-dev.ps1`.

## 4 — Prod-path E2E gate

```powershell
$env:FAIR_CRM_DEV_BYPASS_CORE = "false"
python scripts/e2e_validation.py --prod-path
```

Bütün adımlar `[PASS]` ise auth/RBAC hattı sağlıklı. Rapor: `scripts/e2e_validation_report.json`. Adımlar ve CI karşılığı: [CI_PROD_PATH_E2E.md](../testing/CI_PROD_PATH_E2E.md).

## 5 — Hızlı pytest (servis gerekmez)

```powershell
cd kyrox-fair-crm\backend
python -m pytest tests/modules/test_endpoint_permission_enforcement.py tests/modules/test_role_matrix_authorization.py -q
```

## Sık sorunlar

| Sorun | Çözüm |
|-------|-------|
| E2E adım **0b** fail | Fair CRM bypass açık; `.env` → `false`, `reset-dev.ps1` |
| E2E adım **4/5** fail | Core (8000) veya Fair (8001) kapalı |
| 403 her yerde | `python scripts/seed_core_dev_identity.py` |
| Giriş şifresi tutmuyor | Seed var olan kullanıcının şifresini değiştirmez; kullanıcı daha önce başka şifreyle oluşturulduysa o şifre geçerlidir |
| JWT invalid | `JWT_SECRET_KEY` Core ↔ Fair CRM eşleşmesi |
| kyrox-core bulunamadı | Sibling clone veya `KYROX_CORE_ROOT=C:\path\to\kyrox-core` |

Core login denemesi:

```powershell
curl -X POST http://127.0.0.1:8000/api/v1/auth/login -H "Content-Type: application/json" -d "{\"email\":\"dev@example.com\",\"password\":\"$env:DEV_USER_PASSWORD\"}"
```

## İlgili

- [PERMISSION_SCOPE_GOVERNANCE.md](../PERMISSION_SCOPE_GOVERNANCE.md) ve [INTEGRATION_WITH_CORE.md](../integrations/INTEGRATION_WITH_CORE.md) — RBAC ayrıntısı
- [DEV_RUNTIME.md](DEV_RUNTIME.md) — betikler ve çalışma ortamı
