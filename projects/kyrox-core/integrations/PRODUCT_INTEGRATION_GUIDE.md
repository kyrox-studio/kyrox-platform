# KYROX Core — Product Integration Guide

**Version:** v0.4.0+ (product integration APIs)  
**Audience:** Product teams (e.g. FAIR CRM) integrating with KYROX Core as an **independent platform service**

---

## 1. Overview

### Local implementation contract: system in-app notifications (2026-10-12)

Approved for the TOBB product requirement; local acceptance passed and production deployment is pending. Core reuses `platform_notifications` with channel `in_app`; persistence delivers the inbox entry without SMTP or a notification dispatch job.

- `POST /api/v1/system/notifications/in-app`: active JWT/session and Core DB Super Admin flag required; organization headers are unnecessary. Body: `recipient` (`super_admin` broadcast or user UUID), `subject`, `body`, optional generic `variables`, required `idempotency_key`.
- `GET /api/v1/system/notifications/in-app?limit=50`: same Super Admin guard; returns only platform-scoped in-app messages addressed to `super_admin` or the authenticated user. Limit 1–100, newest first. Response includes id, subject, body, variables and created_at.
- `POST /api/v1/internal/notifications/in-app`: the same delivery payload, authenticated by `X-Kyrox-Notification-Token`. Core must explicitly configure `CORE_PRODUCT_NOTIFICATION_TOKEN` (minimum 32 characters); product workers configure the same secret as `KYROX_CORE_PRODUCT_NOTIFICATION_TOKEN`. Missing configuration fails closed with 503, wrong/missing credentials with 401. The credential is purpose-separated from lifecycle reads, suspension signals, user tokens and provider secrets, and is never persisted in job payloads or logs.

Delivery is idempotent within system scope. Replaying a key with changed content or navigation variables returns 409. Inbox access cannot be granted to OrganizationAdmin through role permissions. Products keep domain run/review history in their own database and send only generic content/navigation metadata through HTTP. No cross-repository imports, shared sessions, foreign keys, second notification store or external notification are permitted.

KYROX Core is a **standalone backend platform service**. Product applications are **separate services** that integrate through **Core public HTTP APIs** under `/api/v1`.

Products must **not**:

- Import kyrox-core Python packages
- Share a database or SQLAlchemy session with Core
- Mount Core routers inside the product application

Products **must**:

- Authenticate users through Core
- Pass organization context on org-scoped calls
- Use Core APIs for platform capabilities (audit, settings, jobs, notifications)
- Store product domain data in the product's own database

---

## 2. Runtime topology

```text
Client  ──►  KYROX Core (:8000)     auth, orgs, RBAC, platform services
Client  ──►  Product app (:8001)    product domain APIs

Product app ──HTTP──► KYROX Core    permission check, audit write, settings, jobs, notifications
```

Each service has its own database. Product tables reference Core `organization_id` as an opaque UUID — no cross-database foreign keys.

---

## 3. Authentication

### Login

```http
POST /api/v1/auth/login
Content-Type: application/json

{"email": "user@example.com", "password": "..."}
```

Response includes `access_token` and `refresh_token`.

### Refresh and logout

| Method | Path |
|--------|------|
| `POST` | `/api/v1/auth/refresh` |
| `POST` | `/api/v1/auth/logout` |

### JWT access token

Core issues signed JWT access tokens. Claims include:

| Claim | Meaning |
|-------|---------|
| `sub` | User id (UUID) |
| `email` | User email |
| `sid` | Session id (UUID) |
| `exp`, `iat`, `jti` | Expiry and token metadata |

**Organization is not embedded in the JWT.** Tenant context is supplied per request.

### Product-side validation

Product services validate access tokens locally using the same `JWT_SECRET_KEY` and `JWT_ALGORITHM` configured on Core (environment alignment — not a code import).

Reject expired or invalid tokens with `401 Unauthorized`. Local decoding cannot see a revoked server-side session; Core rejects it on the next Core call (every Core-protected route requires the `sid` session to be active), so products must authorize protected operations through `POST /organizations/{organization_id}/authorization/check`.

---

## 4. Organization context

Org-scoped Core and product routes require:

```http
Authorization: Bearer <access_token>
X-Organization-Id: <organization_uuid>
```

Rules:

- Path `{organization_id}` must match `X-Organization-Id` or Core returns `400 Organization scope mismatch`.
- The user must have an **active organization membership** (or be a platform super-admin) for org-scoped integration endpoints.

---

## 5. Authorization (RBAC)

### Platform Super Admin

Platform Super Admin is not a role. It is determined only by the authenticated user's `identity_users.is_super_admin` database flag. After authentication, Core returns an authorization allow result before RBAC, permission lookup, membership, organization scope, or ownership enforcement. This also covers new or unseeded permission codes. Authentication and token validity are never bypassed ([ADR-0003](../../../ecosystem/decisions/0003-identity-security-strategy.md#platform-super-admin-invariant)): when any user, including a Super Admin, is deactivated, deleted or loses the Super Admin flag, all of that user's sessions are revoked and existing tokens stop working immediately. The last active, non-deleted Super Admin cannot be deactivated, demoted or deleted, including by themselves (API 409, database trigger as concurrency guard).

Products must honor this Core authorization result and must not add local role, membership, scope, ownership, hide, disable, or CRUD checks that can deny a Platform Super Admin. For all other users, products must remain fail-closed and enforce the normal database-backed authorization result. Only an active Platform Super Admin may change the flag, and no role—including `OrganizationAdmin`—may receive the same bypass.

The canonical invariant and lifecycle protections are defined in [ADR-0003: Identity security strategy](../../../ecosystem/decisions/0003-identity-security-strategy.md#platform-super-admin-invariant).

### Role templates and permission lifecycle

`OrganizationAdmin`, `ReadUser`, `CreateUpdateUser`, and `FullUser` are database-backed system definitions and never authorization bypasses. `OrganizationAdmin` is a protected role assignable in every organization. The other three are templates from which Super Admin creates organization-specific, editable roles. Products must authorize from Core's permission decision, never from a role name or template type.

Super Admin may explicitly synchronize existing derived roles from a changed template. Core also owns platform-wide permission lifecycle: **locked** permissions stay on roles but authorize nothing for non-Super-Admin actors until unlocked; **inactive** permissions are removed from all system, derived, and custom roles and cannot be reassigned by an organization administrator. Super Admin remains unaffected because ADR-0003's flag-based bypass runs before permission lifecycle enforcement.

The canonical model, propagation scopes, preview/audit requirements, and permission states are defined in [ADR-0005: Role template and permission governance](../../../ecosystem/decisions/0005-role-template-and-permission-governance.md).

### Permission code format

Dot-separated, lowercase, minimum three segments:

```text
{module}.{resource}.{verb}
```

Examples:

| Code | Owner |
|------|-------|
| `audit.logs.read` | Core platform |
| `settings.platform.read` | Core platform |
| `fair_crm.customers.read` | Product (registered in Core RBAC) |

### Permission modules

Core platform modules: `audit`, `core`, `identity`, `jobs`, `notifications`, `settings`.

Product modules use the same naming rules (e.g. `fair_crm`). Product permission codes are stored in Core's RBAC tables and assigned to roles — products do not reimplement RBAC.

### Product permissions

Product permission codes are seeded by Core migrations (FAIR CRM customers first in `20260701_0025`; later modules in their own migrations) and assigned to organization roles in Core. Products check them through the authorization check API below. FAIR CRM catalog and scope: [PERMISSION_SCOPE_GOVERNANCE.md](../../fair-crm/PERMISSION_SCOPE_GOVERNANCE.md).

### Check permission (product integration API)

Products call Core to verify whether the **current user** has a permission in the **current organization**:

```http
POST /api/v1/organizations/{organization_id}/authorization/check
Authorization: Bearer <access_token>
X-Organization-Id: <organization_uuid>
Content-Type: application/json

{"permission_code": "fair_crm.customers.read"}
```

Response:

```json
{"allowed": true, "permission_code": "fair_crm.customers.read"}
```

Notes:

- Requires authentication. Normal users also require active organization membership; Platform Super Admin bypasses membership and organization-scope authorization.
- Does **not** require the permission being checked — use this before enforcing product route access.
- Returns `allowed: false` when the user lacks the permission (HTTP 200, not 403).
- Returns `allowed: true` for a Platform Super Admin even when the requested permission code has not been registered or seeded.

Protected Core routes use `require_permission` internally and return `403` when denied.

---

## 6. Organizations, users and lifecycle

Core has no membership/invitation model (removed in `20260817_0057_remove_memberships`); a normal user belongs to one organization.

| Operation | API |
|-----------|-----|
| List / create organization (create: Super Admin) | `GET`, `POST /api/v1/organizations` |
| Get / update / delete organization | `GET`, `PATCH`, `DELETE /api/v1/organizations/{organization_id}` |
| Suspend / reactivate (SYSTEM) | `POST /api/v1/organizations/{organization_id}/suspend`, `.../reactivate` |
| Organization access check | `GET /api/v1/organizations/{organization_id}/access/verify` |
| Lifecycle snapshot (product lifecycle token) | `GET /api/v1/organizations/{organization_id}/lifecycle-snapshot` |
| Retained audit evidence purge (product lifecycle token) | `POST /api/v1/organizations/{organization_id}/retained-audit-evidence/purge` |
| Users | `GET /api/v1/organizations/{organization_id}/users`, `.../users/deleted`, `POST .../users/manual`, `GET`/`PATCH`/`DELETE .../users/{user_id}`, `POST .../users/{user_id}/restore` |
| Caller context (incl. Super Admin) | `GET /api/v1/user-management/context` |
| Roles | `GET`/`POST .../roles`, `GET`/`PATCH`/`DELETE .../roles/{role_id}`, `PUT .../roles/{role_id}/permissions`, `GET .../managed-roles`, `GET .../role-permissions` |
| Permission catalog and lifecycle | `GET /api/v1/permissions`, `POST /api/v1/permissions/{permission_id}/lifecycle` (+ `/preview`) |
| Role templates | `GET /api/v1/role-templates`, `PATCH .../{role_id}`, `POST .../{role_id}/derive`, `.../sync` (+ `/preview`) |

Products consume these APIs; they do not reimplement organization or user lifecycle.

---

## 7. Audit

### Write (product integration API)

Products append org-scoped audit events:

```http
POST /api/v1/organizations/{organization_id}/audit-events
Authorization: Bearer <access_token>
X-Organization-Id: <organization_uuid>
Content-Type: application/json

{
  "action": "fair_crm.customer.created",
  "resource_type": "customer",
  "resource_id": "550e8400-e29b-41d4-a716-446655440000",
  "new_values": {"display_name": "Example Co"},
  "metadata": {"source": "api"}
}
```

Response: `201 Created` with the recorded audit event.

Rules:

- **Append-only** — no update or delete APIs.
- Requires authentication and active organization membership.
- `action` must follow `module.resource.action` naming (e.g. `fair_crm.customer.created`).
- `user_id` and `session_id` are taken from the JWT — callers cannot impersonate another user.
- Do not store passwords, tokens, or secrets in `old_values`, `new_values`, or `metadata`.

### Query

```http
GET /api/v1/organizations/{organization_id}/audit-logs
```

Requires permission `audit.logs.read`. Supports cursor pagination and filters (`action`, `action_prefix`, `resource_type`, etc.).

**Product integration note:** Products such as FAIR CRM may treat audit **writes** as best-effort in early sprints — CRM operations should not fail solely because the audit API is unavailable. Products should log audit write failures clearly at warning level.

---

## 8. Settings

Org-scoped key/value settings (opaque JSON):

| Method | Path |
|--------|------|
| `GET` | `/organizations/{id}/settings` |
| `GET` | `/organizations/{id}/settings/{key}` |
| `PUT` | `/organizations/{id}/settings/{key}` |
| `DELETE` | `/organizations/{id}/settings/{key}` |

Permissions: `settings.platform.read`, `settings.platform.update`.

Product keys use a product namespace:

```text
fair_crm.customers.default_status
fair_crm.import.preview_required
```

Core stores values without interpreting product schemas.

System settings (`/api/v1/system/settings/...`) require platform super-admin.

---

## 9. Background jobs

| Method | Path | Permission |
|--------|------|------------|
| `POST` | `/organizations/{id}/jobs` | `jobs.platform.execute` |
| `GET` | `/jobs/{id}` | `jobs.platform.read` |

Enqueue example:

```json
{
  "job_type": "fair_crm.import.process_batch",
  "payload": {"batch_id": "..."}
}
```

Product job **handlers** execute in the product service (or product worker). Core stores job metadata and dispatch status.

---

## 10. Notifications

| Method | Path | Permission |
|--------|------|------------|
| `POST` | `/organizations/{id}/notifications/send` | `notifications.platform.execute` |
| `GET` | `/notifications/{id}` | `notifications.platform.read` |

Products pass rendered subject/body or template metadata. Core dispatches asynchronously via the jobs platform.

---

## 11. Error handling

Core returns standard HTTP status codes:

| Code | Meaning |
|------|---------|
| `400` | Validation error or organization scope mismatch |
| `401` | Missing or invalid token |
| `403` | Permission denied (protected Core routes) or inactive membership |
| `404` | Resource not found |
| `422` | Request schema validation failed |

Error body shape: `{"detail": "..."}`.

---

## 12. Product integration checklist

1. Configure `KYROX_CORE_BASE_URL` and matching JWT validation settings.
2. Direct clients to Core for login; reuse access token on product API.
3. Send `X-Organization-Id` on every org-scoped product request.
4. Call `POST .../authorization/check` before enforcing product permissions.
5. Register product permission codes in Core RBAC (migrations or admin process).
6. Emit audit events via `POST .../audit-events` after successful product mutations.
7. Use settings/jobs/notifications APIs instead of reimplementing platform logic.
8. Keep product domain data in the product database only.

---

## 13. Health

```http
GET /api/v1/health
```

Use for load balancer and local development smoke checks.

---

## 14. Versioning

- Current API version prefix: `/api/v1`
- Core is not released by tag today (last tag `v0.4.0`): `deploy-all.sh` pulls Core `main`, and FAIR CRM prod-path CI checks out the Core branch with the same name as the PR branch when it exists, otherwise `main`.
- Breaking API changes require a new version prefix and migration notes.

---

## 15. Related documents

- [KYROX Core Architecture](../README.md)
- [Backend Architecture Standards](../../../standards/backend/BACKEND_ARCHITECTURE_STANDARDS.md)
- [Platform Services Design](../../../archive/kyrox-core/designs/PLATFORM_SERVICES_DESIGN.md)
- [Identity Platform Design](../../../archive/kyrox-core/designs/IDENTITY_PLATFORM_DESIGN.md)
- kyrox-platform [ADR-0004 Audit service strategy](../../../ecosystem/decisions/0004-audit-service-strategy.md)
