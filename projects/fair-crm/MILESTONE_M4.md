# M4 — FAIR CRM v1

- **Status:** Active
- **Current phase:** Active product delivery
- **Primary code repository:** fair-crm
- **Documentation:** [projects/fair-crm/](./)

## Goal

Ship the first production-ready version of FAIR CRM on KYROX Core identity (M2) and platform services (M3).

Application code lives in **fair-crm**. Documentation and decisions live in **kyrox-platform**. kyrox-core remains frozen except bug/security/performance fixes and CRM-driven reusable platform needs.

## Prerequisites (met)

- [x] M2 Identity Platform — kyrox-core v0.2.0 through v0.3.0
- [x] M3 Platform Services — kyrox-core v0.4.0
- [x] Core product-integration APIs used by FAIR CRM: authorization check, audit write, organization lifecycle ([INTEGRATION_WITH_CORE.md](integrations/INTEGRATION_WITH_CORE.md))

## Current state

Delivered capabilities: [PROJECT_STATUS.md](PROJECT_STATUS.md). Open work: [ROADMAP.md](ROADMAP.md).

## Scope

- Core CRM workflows and domain model
- Product UI integrated with Core auth and organization scoping
- Configuration and deployment for initial users
- Product-specific integrations not belonging in Core
- Data-integration pipeline (preview, duplicate detection, merge decisions)

## Dependencies

- kyrox-core (frozen except bug/security/performance fixes and CRM-driven reusable needs)
- [README.md](README.md)

## Success criteria

- FAIR CRM v1 deployable and usable by target initial users
- No reverse dependency from kyrox-core to fair-crm
- Status / roadmap / changelog kept current under this documentation hub
- CRM-driven Core gaps implemented in kyrox-core only when reusable across products ([ADR-0002](../../ecosystem/decisions/0002-core-product-separation.md))

## Related

- [PROJECT_STATUS.md](PROJECT_STATUS.md)
- [../../ecosystem/STATUS.md](../../ecosystem/STATUS.md)
- [../../ecosystem/ROADMAP.md](../../ecosystem/ROADMAP.md)
- [../../ecosystem/WORKFLOW.md](../../ecosystem/WORKFLOW.md)
