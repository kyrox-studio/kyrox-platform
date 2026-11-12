# KYROX Core — Project Status

Living status for KYROX Core. Ecosystem summary: [../../ecosystem/STATUS.md](../../ecosystem/STATUS.md). Future work: [ROADMAP.md](ROADMAP.md).

| Field | Value |
|-------|-------|
| Last verified | **2026-10-30** |
| Repository mode | Stable platform baseline; fixes and reusable product-driven changes continue |
| Active ecosystem milestone | M4 — FAIR CRM v1 |
| Migration head in `main` | `20261027_0078_login_throttles` (`main` `3cf08c8`) |
| Last recorded deploy | `cac2f9f` (2026-11-12, #55), production head `20261027_0078` |

Notification `in_app` system channel and Super Admin inbox: Core PR #42 (2026-10-13), migration `20261012_0073`.

## Current capability state

| Area | Status |
|------|--------|
| Identity and authentication | Implemented baseline plus database-backed sign-in throttle/lockout (`0078`), absolute session expiry (`0077`), refresh-token reuse detection, shared 12–255 character `PasswordPolicy`, one-time identity action tokens, user-wide session/refresh invalidation, server-side access-token session enforcement, controlled public signup/atomic first-user bootstrap, single-use activation/set-password, Core-owned forgot/reset password and authenticated self-service password change; final P0.2 Core adversarial certification is complete |
| Authorization | Implemented; normal organization permission checks require an `active` organization, so `pending_activation` organizations are non-operational until successful activation |
| Organization and user management | Implemented; normal users use direct single-organization ownership via `identity_users.organization_id`; public signup creates a `pending_activation` organization and inactive first `OrganizationAdmin`, CORE-06 activates both after a valid one-time token/password-set flow, and existing Platform Super Admin organization/user creation remains available |
| Membership / invitation model | Removed by migration `20260817_0057_remove_memberships`; not a current Core capability |
| Roles and permissions | Implemented and evolved through later migrations |
| Audit | Implemented; activation completion records secret-safe `identity.activation.complete`, password reset records `identity.password.reset`, and authenticated password change records `identity.password.change` evidence |
| Settings | Implemented |
| Background jobs | Implemented; internal jobs support platform scope where required by Core identity notifications while existing organization-scoped behavior remains available |
| Notifications | Implemented baseline plus Core-owned production SMTP identity-email capability, platform-scoped recipients/jobs, identity templates, deterministic platform idempotency and redacted delivery logging; generic `in_app` system channel with idempotent delivery and Super Admin inbox (Core PR #42) |
| Product authorization integration | Implemented |
| FAIR CRM permission catalog support | Implemented, including quotation, cost-catalog, mail-send-operation, and Fair Stand admin/projects/settings/items permissions (`20260919_0066`–`20260922_0069`, lifecycle `active`) |
| File storage | Planned |
| Caching | Demand-driven candidate |
| Observability | Demand-driven candidate |

## P0.2 identity / onboarding and lifecycle records

Delivery history (CORE-01…CORE-09, PRs, CI runs, test counts): [P0_2_IDENTITY_ONBOARDING_IMPLEMENTATION.md](../../ecosystem/P0_2_IDENTITY_ONBOARDING_IMPLEMENTATION.md). Lifecycle (OL-05…OL-10): [SAAS_ROADMAP.md](../../ecosystem/SAAS_ROADMAP.md) P0.2.

## Organization lifecycle baseline

Current Core organization lifecycle behavior relevant to SaaS P0.2:

- existing operator-driven organization creation remains Platform Super Admin only and creates an `ACTIVE` organization,
- public signup creates a separate explicit `PENDING_ACTIVATION` organization and inactive first user; CORE-06 activation moves both to `ACTIVE` only after successful one-time token validation and password set,
- normal users are assigned directly to one organization and receive database-backed roles,
- the first public-signup user receives the existing protected `OrganizationAdmin` role and is not a Super Admin,
- the legacy `Owner` role is removed; `OrganizationAdmin` remains the protected organization administrative role,
- organization suspend/delete are SYSTEM-scope and are not assignable to organization roles,
- normal authorization requires an active organization; `PENDING_ACTIVATION`, `SUSPENDED` and `ARCHIVED` organizations are non-operational for normal permission-protected paths,
- Core organization delete is a soft-delete (`deleted_at`), not cross-product data deletion,
- Platform Super Admin reactivation: `POST /organizations/{organization_id}/reactivate` (`SUSPENDED → ACTIVE`, SYSTEM `identity.organizations.reactivate`),
- product lifecycle contracts (product lifecycle token): `GET /organizations/{organization_id}/lifecycle-snapshot`, terminal tombstone `DELETE /organizations/{organization_id}`, `POST /organizations/{organization_id}/retained-audit-evidence/purge`; `GET /organizations/{organization_id}/access/verify`,
- cross-repository product job/provider/data-retention effects are not implied by Core organization state changes.

The cross-repository lifecycle contract is [ADR-0006](../../ecosystem/decisions/0006-organization-lifecycle-and-onboarding.md) (Proposed overall; OL-01…OL-10 required-export path accepted).

## Layering (2026-11-12)

Role and user management queries moved from `api/` to `infrastructure/role_management` and `infrastructure/user_management`; `AuthorizationContext` lives in `application/authorization/context.py` (re-exported by `api/authorization/context.py`). `backend/tests/test_api_layer_has_no_orm.py` enforces the layer imports. No behavior or schema change.

## Migration status

Current head: `20261027_0078_login_throttles`. Recent: `0073` in-app notifications, `0075` permission lock keeps role grants (ADR-0005), `0076` roles.assign grants, `0077` session absolute expiry, `0078` login throttle. Fair Stand admin/project/settings/item permissions: `0066`–`0069`; preview permissions dropped in `0072`.

The code repository is the implementation source for complete migration history. This document records the verified current head and capability-level state only.

## Integration contract

Products integrate with Core through public HTTP APIs. Product domain implementation stays in the product repository.

Canonical contract: [integrations/PRODUCT_INTEGRATION_GUIDE.md](integrations/PRODUCT_INTEGRATION_GUIDE.md).

## Update protocol

When Core changes materially:

1. Update this file with current capability truth and migration head when useful.
2. Update [CHANGELOG.md](CHANGELOG.md) for delivered history.
3. Refresh the summary in [../../ecosystem/STATUS.md](../../ecosystem/STATUS.md).
4. Keep exact test counts and commit SHAs out of permanent standards and roadmap documents; implementation evidence may be recorded in project status/tracker history.