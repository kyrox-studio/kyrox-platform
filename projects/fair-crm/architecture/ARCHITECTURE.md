# FAIR CRM Architecture

**Status:** As-built (`kyrox-fair-crm` `main`)  
**Live status:** [PROJECT_STATUS.md](../PROJECT_STATUS.md)  
**Related:** [INTEGRATION_WITH_CORE.md](../integrations/INTEGRATION_WITH_CORE.md), [DECISIONS.md](../decisions/DECISIONS.md), [Backend Architecture Standards](../../../standards/backend/BACKEND_ARCHITECTURE_STANDARDS.md)

## 1. Two services, two databases, one token

FAIR CRM is an independent FastAPI product service (`:8001`) with its own PostgreSQL database (`fair_crm`). KYROX Core is a separate platform service (`:8000`, database `kyrox_core`). They talk only through Core's public HTTP APIs. Inside, FAIR CRM is a modular monolith.

| Decision | Choice | ADR |
|----------|--------|-----|
| Core relationship | Independent platform service; public API integration | ADR-007 |
| Product deployment | Independent FastAPI service | ADR-008 |
| Core Python imports | Forbidden | ADR-007 |
| Database | Separate product database; no shared session, pool or FK to Core tables | ADR-007 |
| Tenancy | `organization_id` (UUID) from Core; logical key, not an FK | ADR-002 |
| Language | English backend/API/DB; Turkish UI | ADR-006 |

Browser paths are same-origin: Core at `/kyrox-core/api/v1/...`, FAIR CRM at `/api/v1/...`, Fair Stand API at `/api/v1/fair-stand/...` (proxied to `:8002`). Vite (local) and nginx (server) proxy them ([DEV_RUNTIME.md](../ops/DEV_RUNTIME.md), [NGINX_SSL_CONFIGURATION.md](../ops/NGINX_SSL_CONFIGURATION.md)).

## 2. Repository layout

```text
kyrox-fair-crm/
  backend/
    app/
      main.py
      api/v1/router.py          # mounts module routers under /api/v1
      core/                     # config, exceptions, logging, pagination, request timing, performance monitoring
      db/                       # session, base, alembic version compatibility
      integrations/kyrox_core/  # Core HTTP client and ports: auth (JWT decode), client, lifecycle, tombstone,
                                # audit_retention, super_admin, dev_bypass (local only), ports
      shared/                   # cross-module primitives (background jobs, consent, email, secret encryption,
                                # queued/running work lifecycle, canonical import, database backup, …)
      modules/<module>/         # domain / application / infrastructure / api
    alembic/                    # product schema only
    scripts/                    # backend tooling (e.g. diagnose_crm_performance.py)
    tests/
  frontend/                     # React + TypeScript (Vite); hosts the Fair Stand configurator via @fair-stand
  scripts/                      # dev (PowerShell), server (deploy/backup/nginx/systemd), maintenance (CI governance),
                                # e2e_validation.py, seed_core_dev_identity.py, validate_feature_contracts.py, quality_check.py
  docker-compose.yml            # local PostgreSQL
  .kyrox/                       # machine-readable feature contracts, schemas, backend test baseline
```

Modules (`backend/app/modules/`): `activities`, `auth`, `contacts`, `cost_catalog`, `customers`, `dashboard`, `data_integration`, `email_accounts`, `email_delivery`, `email_webhooks`, `fair_emails`, `fairs`, `imports`, `mail_send_operations`, `mail_templates`, `operations`, `organization_closure`, `participations`, `quote_templates`, `quotes`, `scraper`, `smtp`, `system_admin`, `template_contents`, `todos`.

Rules: no import of Core Python modules; no Core routers mounted in the CRM app; no shared `DATABASE_URL`; Core access only through `app/integrations/kyrox_core/`. Modules talk through application services; shared primitives live in `backend/app/shared/`. Table naming: [CONSTITUTION.md §4](../CONSTITUTION.md#4-language-and-naming).

## 3. Layers (per module)

| Layer | Responsibility | Example (`customers`) |
|-------|----------------|------------------------|
| domain | Entities, value objects, ports | `Customer`, `CustomerRepository` |
| application | Use cases | `CreateCustomerUseCase`, `ArchiveCustomerUseCase` |
| infrastructure | SQLAlchemy models and repositories | `SqlAlchemyCustomerRepository` |
| api | FastAPI routes, schemas, dependency wiring | `DELETE /api/v1/customers/{customer_id}` |

Dependency direction: `api → application → domain`; infrastructure implements domain ports; Core integration sits behind ports in `integrations/kyrox_core/`.

## 4. Request pipeline

```text
HTTP request → FAIR CRM
  → JWT decoded locally with the shared JWT_SECRET_KEY (integrations/kyrox_core/auth.py)
  → X-Organization-Id read; Super Admin status from Core GET /api/v1/user-management/context
  → permission check: Core POST /api/v1/organizations/{organization_id}/authorization/check
  → use case → repository (fair_crm DB, organization-scoped)
  → audit event: Core POST /api/v1/organizations/{organization_id}/audit-events
  → response
```

Every use case takes `organization_id` from the validated auth context, never from the request body. Organization lifecycle (suspension, tombstone, audit retention) also comes from Core through `lifecycle.py`, `tombstone.py`, `audit_retention.py`.

## 5. Testing

| Layer | Approach |
|-------|----------|
| Domain / application | Unit tests with fake repositories and fake Core ports |
| Infrastructure | Integration tests against the product DB |
| Integrations | Mocked Core HTTP; real Core in prod-path E2E ([CI_PROD_PATH_E2E.md](../testing/CI_PROD_PATH_E2E.md)) |
| API | `TestClient`; tenant isolation registry ([TENANT_ISOLATION_SYSTEM_TEST_STANDARD.md](../../../standards/quality/TENANT_ISOLATION_SYSTEM_TEST_STANDARD.md)) |

`python scripts/quality_check.py` before a PR. CI: [GITHUB_WORKFLOWS.md](../testing/GITHUB_WORKFLOWS.md).

History: Sprint 1.0 architecture design: [ARCHITECTURE_SPRINT1_LEGACY.md](../../../archive/fair-crm/ARCHITECTURE_SPRINT1_LEGACY.md); legacy `fuar-crm` reference: [FUAR_CRM_REFERENCE_ANALYSIS.md](../../../archive/fair-crm/FUAR_CRM_REFERENCE_ANALYSIS.md).
