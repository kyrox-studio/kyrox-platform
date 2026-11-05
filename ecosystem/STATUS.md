# KYROX Ecosystem Status

Single source of truth for **cross-repository current state**. Detailed product/Core status lives in the project status files linked below. Do not duplicate sprint histories, test-count histories or roadmap queues here.

| Field | Value |
|-------|-------|
| Active milestone | **M4 — FAIR CRM v1** |
| Core policy | Frozen for speculative product work; bug/security/performance fixes and approved reusable product-driven platform needs are allowed |
| Documentation hub | `kyrox-platform` |
| Implementation repos | `kyrox-core`, `kyrox-fair-crm`, `kyrox-fair-stand`; marka varlıkları: `kyrox-brand`; web sitesi: `kyrox-web` |
| Last ecosystem sync | **2026-10-29** |

## SaaS readiness

| Workstream | State | Canonical record |
|---|---|---|
| P0.1 tenant isolation | **DONE** (2026-08-26). New organization-owned behavior still ships with cross-organization evidence | [P0_1_TENANT_ISOLATION_CERTIFICATION](../archive/fair-crm/backlog/P0_1_TENANT_ISOLATION_CERTIFICATION.md) |
| P0.2 identity / SaaS onboarding | **DONE** (2026-08-29). Organization is the account boundary; FAIR CRM is a thin consumer of Core identity APIs and owns no credential authority | [P0_2_IDENTITY_ONBOARDING_IMPLEMENTATION.md](P0_2_IDENTITY_ONBOARDING_IMPLEMENTATION.md) |
| OL-07 suspension / reactivation | **DONE** (2026-09-07) | [P0_2_OL_07_IMPLEMENTATION.md](P0_2_OL_07_IMPLEMENTATION.md) |
| OL-08 required-export closure | **DONE / certified** (2026-09-11) through package integrity, credential disposition, artifact/product cleanup, Core tombstone and 12-month retained-evidence expiry. `not_required` export has no accepted policy authority and fails closed | [P0_2_OL_08_IMPLEMENTATION.md](P0_2_OL_08_IMPLEMENTATION.md), [P0_2_OL_08_B_EXPORT_DECISION.md](P0_2_OL_08_B_EXPORT_DECISION.md) |
| OL-09 timing / OL-10 backup ageing | Accepted contracts; obligations used by the closure path are runtime-covered | [P0_2_OL_09_RETENTION_GRACE_DECISION_READINESS.md](P0_2_OL_09_RETENTION_GRACE_DECISION_READINESS.md), [P0_2_OL_10_RUNTIME_ACCEPTANCE.md](P0_2_OL_10_RUNTIME_ACCEPTANCE.md) |

[ADR-0006](decisions/0006-organization-lifecycle-and-onboarding.md) remains Proposed overall. No additional OL08 lifecycle runtime is sequenced or authorized; a new case needs an explicit policy/product decision.

## KYROX Core

Canonical detail: [projects/kyrox-core/PROJECT_STATUS.md](../projects/kyrox-core/PROJECT_STATUS.md)  
Active planning: [projects/kyrox-core/ROADMAP.md](../projects/kyrox-core/ROADMAP.md)

Core remains the reusable, product-agnostic SaaS backend. Identity, authentication, authorization, organization/user/role governance and shared platform services are implemented in Core. Products consume Core through public contracts; product domain logic does not belong in Core.

The approved P0.2 identity/onboarding primitives are complete. Core owns canonical organization/identity lifecycle and SYSTEM-scoped organization suspend/reactivate/delete authority. Product-specific closure progress/export/provider/data/artifact behavior remains outside Core. For current-model closure, Core additionally owns the final tombstone timestamp and Core audit-retention execution; FAIR CRM consumes those accepted public/internal product-lifecycle contracts without taking Core identity/audit ownership.

Core PR #42 (2026-10-13) added a generic `in_app` system notification channel and Super Admin inbox, requested by the FAIR CRM TOBB workflow. Core `main` migration head: `20261027_0078_login_throttles`.

## FAIR CRM

Canonical detail: [projects/fair-crm/PROJECT_STATUS.md](../projects/fair-crm/PROJECT_STATUS.md)  
Current work queue: [projects/fair-crm/ROADMAP.md](../projects/fair-crm/ROADMAP.md)

FAIR CRM remains the active M4 product. Its P0.1 tenant-isolation certification, P0.2 identity bridge, OL-07 suspension/reactivation runtime and the current-model required-export OL08 closure path are complete within their accepted boundaries. FAIR CRM `main` reaches migration `0098_tobb_manual_date_revision`; the closure path itself ends at `0083_terminal_closure`, and OL08-F adds post-terminal retention runtime without a new FAIR migration.

The TOBB System Fair review and controlled synchronization workflow (ADR-038 amendment + ADR-040) merged on 2026-10-13 through FAIR CRM PR #342 and is in the last recorded deploy (`122fdf9f`, 2026-10-28).

The current documentation/quality focus is to keep Platform as the single human/AI knowledge source and to ensure permission-controlled UI surfaces consistently follow effective permissions and the shared CRUD/UI authorization standard.

## Fair Stand

Canonical detail: [projects/fair-stand/PROJECT_STATUS.md](../projects/fair-stand/PROJECT_STATUS.md)  
Work queue: [projects/fair-stand/ROADMAP.md](../projects/fair-stand/ROADMAP.md)

Fair Stand is an independent FastAPI service (`:8002`) with its own PostgreSQL `fair_stand` database; the configurator UI is hosted inside the FAIR CRM frontend ([ADR-0007](decisions/0007-fair-stand-product-ownership.md), [ADR-0008](decisions/0008-fair-stand-independent-service.md)). `main` migration head: `0090_carpet_anthracite`.

## Milestones

| Milestone | Status | Canonical document |
|-----------|--------|--------------------|
| M1 Foundation | Completed / historical | [archive/milestones/M1_FOUNDATION.md](../archive/milestones/M1_FOUNDATION.md) |
| M2 Identity | Completed / historical | [archive/milestones/M2_IDENTITY.md](../archive/milestones/M2_IDENTITY.md) |
| M3 Platform Services | Completed / historical | [archive/milestones/M3_PLATFORM_SERVICES.md](../archive/milestones/M3_PLATFORM_SERVICES.md) |
| M4 FAIR CRM v1 | **Active** | [projects/fair-crm/MILESTONE_M4.md](../projects/fair-crm/MILESTONE_M4.md) |

## Repository boundaries

| Repository | Owner scope |
|------------|-------------|
| `kyrox-platform` | All human/AI documentation, shared standards, project docs, ADRs, status and roadmap |
| `kyrox-core` | Reusable SaaS platform implementation, tests, migrations and CI |
| `kyrox-fair-crm` | FAIR CRM product implementation, tests, migrations, CI and machine-readable contracts; hosts the Fair Stand configurator UI, Admin screens, and same-origin proxy |
| `kyrox-fair-stand` | Fair Stand FastAPI (`:8002`) and PostgreSQL `fair_stand`; configurator runtime |
| `kyrox-brand` | Logo dosyaları ve marka renk token'ları; kurallar `standards/brand/` altında |
| `kyrox-web` | kyrox.studio: özel WordPress teması ve sunucu kurulumu; içerik WordPress panelinde |

See [REPOSITORY_STRATEGY.md](REPOSITORY_STRATEGY.md) and [DOCUMENT_GOVERNANCE.md](DOCUMENT_GOVERNANCE.md).
