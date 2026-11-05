# FAIR CRM — Integration with KYROX Core

**Status:** As-built (`kyrox-fair-crm` `main`)  
**Related:** [ARCHITECTURE.md](../architecture/ARCHITECTURE.md), [DECISIONS.md](../decisions/DECISIONS.md) ADR-007, Core contract [PRODUCT_INTEGRATION_GUIDE.md](../../kyrox-core/integrations/PRODUCT_INTEGRATION_GUIDE.md)

## 1. Integration model

KYROX Core and FAIR CRM are independent services. FAIR CRM uses Core only through public HTTP APIs ([Backend Architecture Standards §12](../../../standards/backend/BACKEND_ARCHITECTURE_STANDARDS.md#12-product-independence-rules)).

| Approach | Decision |
|----------|----------|
| Call Core public REST APIs | Yes |
| Import kyrox-core Python packages | No |
| Share SQLAlchemy session / database with Core | No |
| Mount Core routers inside FAIR CRM | No |
| Local JWT signature validation with the shared secret | Yes — validates tokens Core issued |

## 2. Runtime and configuration

| Component | Process (server-internal) | Database |
|-----------|---------------------------|----------|
| KYROX Core | `http://127.0.0.1:8000` | `kyrox_core` |
| FAIR CRM | `http://127.0.0.1:8001` | `fair_crm` |

| Setting | Purpose |
|---------|---------|
| `KYROX_CORE_BASE_URL` | Backend HTTP client → Core; not a browser URL |
| `JWT_SECRET_KEY`, `JWT_ALGORITHM` | Same values as Core; local access-token validation |
| `KYROX_CORE_PRODUCT_LIFECYCLE_TOKEN` | Product lifecycle / retained-audit service calls |
| `DATABASE_URL` | FAIR CRM database only |

`backend/requirements.txt` does not depend on kyrox-core. Browser paths: [DEV_RUNTIME.md](../ops/DEV_RUNTIME.md#browser--frontend-network-local--server).

## 3. Integration layer

`backend/app/integrations/kyrox_core/`:

| File | Role |
|------|------|
| `client.py` | `KyroxCoreHttpClient`; permission check (`check_permission`), audit write (`record_event`), Super Admin read |
| `auth.py` | Local JWT decode (`sub`, `email`, `sid`) |
| `ports.py` | `AuthContext` and ports used by application code |
| `lifecycle.py` | Organization lifecycle snapshot (fail-closed guard) |
| `tombstone.py` | Final same-suspension-episode organization delete |
| `audit_retention.py` | Retained audit evidence purge |
| `super_admin.py` | Super Admin reader |
| `dev_bypass.py` | Local development without Core (`FAIR_CRM_DEV_BYPASS_CORE`); never production authority |

## 4. Authentication

- **Browser → FAIR CRM bridge:** the SPA calls FAIR CRM `/api/v1/auth/login`, `/refresh`, `/logout`, `/signup`, `/activation/complete`, `/password/forgot`, `/password/reset`, `/password/change` (`modules/auth`). The bridge forwards to Core `/api/v1/auth/*`; Core issues and validates credentials. The refresh token is the host cookie `fair_crm_refresh_token` (path `/api/v1/auth`); the access token is kept in `fair-crm.auth.session` (localStorage).
- **Browser → Core directly** (`/kyrox-core/api/v1/...`): session identity and organization resolution (`auth/AuthContext.tsx`, `organizationResolver.ts`), organization, user and role management screens (`api/organizations.ts`, `api/userManagement.ts`, `api/roleManagement.ts`).
- **Token validation:** FAIR CRM decodes the JWT locally; claims used are `sub` (user), `email`, `sid` (session). The organization is not in the JWT; it comes from `X-Organization-Id`.

## 5. Backend → Core calls

All on `{KYROX_CORE_BASE_URL}/api/v1`, with the user's `Authorization` and `X-Organization-Id` unless noted:

| Purpose | Core endpoint |
|---------|---------------|
| Permission check (every protected route) | `POST /organizations/{organization_id}/authorization/check` |
| Audit event write (best-effort) | `POST /organizations/{organization_id}/audit-events` |
| Super Admin status | `GET /user-management/context` |
| Organization access check (dashboard) | `GET /organizations/{organization_id}/access/verify` |
| Organization lifecycle snapshot | `GET /organizations/{organization_id}/lifecycle-snapshot` (lifecycle token) |
| Terminal closure tombstone | `DELETE /organizations/{organization_id}` |
| Retained audit evidence purge | `POST /organizations/{organization_id}/retained-audit-evidence/purge` (lifecycle token) |

Core settings, jobs and notifications APIs are not used by FAIR CRM; product jobs run in FAIR CRM (`shared/background_jobs.py`, operations engine, `fair-crm-mail-worker`).

Permission codes and scope: [PERMISSION_SCOPE_GOVERNANCE.md](../PERMISSION_SCOPE_GOVERNANCE.md). Permissions live in Core `identity_permissions` and are added by Core migrations, never by FAIR CRM migrations.

## 6. Errors and audit

- Core HTTP errors map to FAIR CRM exceptions in the integration layer; Core internals are not exposed to clients.
- Audit writes are best-effort: a mutation does not fail only because the audit API is unavailable; the client logs a warning. Permission checks are required: a failure blocks the route.
- Unavailable or malformed lifecycle authority fails closed.

## 7. Data ownership and deploy order

FAIR CRM migrations never touch Core tables; Core migrations never create CRM tables; no cross-database FKs. Deploy Core and its migrations first, then FAIR CRM ([SERVER_DEPLOY.md](../ops/SERVER_DEPLOY.md)).

## 8. Must not

| Anti-pattern | Reason |
|--------------|--------|
| `import app.modules.identity...` from kyrox-core | Service boundary |
| Share `DATABASE_URL` with Core | Cross-repository DB coupling |
| Seed Core `identity_permissions` from FAIR CRM migrations | Wrong database and owner |
| Issue JWTs in FAIR CRM | Core owns auth |
| Call Core endpoints not in its public contract | Fragile coupling |

History: Sprint 1.0 integration design and Core API gaps CG-1…CG-6 (all closed): [INTEGRATION_WITH_CORE_SPRINT1_LEGACY.md](../../../archive/fair-crm/INTEGRATION_WITH_CORE_SPRINT1_LEGACY.md).
