# KYROX Core Roadmap

KYROX Core is the reusable SaaS backend platform for KYROX products. This document is **Core's single work list**: every open, deferred or candidate Core item is one row here (verified against Core `main` `1224fae` on 2026-10-28). Current version, migration state, CI state and capability truth belong in [PROJECT_STATUS.md](PROJECT_STATUS.md).

## Guiding rule

Core remains product-agnostic. Product domain behavior stays in product repositories. Products integrate with Core through documented public HTTP APIs and contracts.

The cross-repository [KYROX SaaS Readiness Roadmap](../../ecosystem/SAAS_ROADMAP.md) defines SaaS sequencing and ownership guardrails. It does not automatically activate Core candidates: billing/subscription, generic metering, observability or other reusable SaaS infrastructure still requires a proven product need and the applicable ownership/planning/ADR decision.

During FAIR CRM M4, Core is frozen against speculative platform development. Allowed work is limited to:

- bug fixes,
- security fixes,
- performance/reliability fixes,
- reusable platform needs proven by a real product requirement.

A reusable-looking capability must have an explicit ownership decision before migration or implementation. Do not move FAIR CRM business semantics into Core merely to make code reusable.

## Existing platform baseline

The existing Core baseline covers identity/authentication, organization/user/role governance, authorization, audit, settings, background jobs, notifications and product authorization integration. See [PROJECT_STATUS.md](PROJECT_STATUS.md) for current implementation truth.

## Active product-driven platform work

None. The in-app notification channel (Core #42) and the P0.2 identity / onboarding slice (CORE-01…09) are delivered and live; history: [CHANGELOG.md](CHANGELOG.md), [P0_2_IDENTITY_ONBOARDING_IMPLEMENTATION.md](../../ecosystem/P0_2_IDENTITY_ONBOARDING_IMPLEMENTATION.md).

## Candidates (not scheduled)

A candidate becomes active work only through the KYROX planning/ADR flow or a documented reusable product need. None of these exists in Core code today.

| Candidate | Note |
|---|---|
| Email verification for every account path | Required by [ADR-0003](../../ecosystem/decisions/0003-identity-security-strategy.md); today only the public signup → activation path proves the address |
| MFA | — |
| Password-reset and signup rate limiting | Login throttling / temporary lockout is delivered (kyrox-core#50, SECURITY_PROGRAM E-026) |
| Device / session management UX and policy hardening | — |
| File storage service | — |
| Webhooks / event bus | — |
| Billing and subscription hooks | — |
| Metrics / OpenTelemetry / distributed tracing | — |
| Caching | Only for a measured need |
| Advanced platform administration and impersonation policies | — |
| Release / operational automation | — |

## Ownership decisions pending

Do not move or duplicate code until an explicit ownership decision is made from the actual current implementation ([ADR-0002](../../ecosystem/decisions/0002-core-product-separation.md); historical proposal: [PLATFORM_OWNERSHIP_REVIEW_2026-07-23](../../archive/ecosystem/PLATFORM_OWNERSHIP_REVIEW_2026-07-23.md)).

| Area | Question |
|---|---|
| Generic email delivery | Should provider-account abstraction, generic delivery execution, retry/rate-limit mechanics and provider-neutral delivery status move to Core? FAIR CRM recipient selection, templates/context, exclusions and activity history stay in FAIR CRM |
| Generic Operation Engine | Are start/cancel/pause/resume/retry/schedule primitives and generic run/job state product-independent enough for Core? FAIR CRM automation types, handlers, source selection and domain results stay in FAIR CRM |

## Product-driven platform extraction

When FAIR CRM or a future product develops infrastructure that appears reusable:

1. Separate generic lifecycle/infrastructure from product business meaning.
2. Decide ownership: Core / product / provider-handler.
3. Document the public contract and migration impact.
4. Move only the generic capability; keep domain orchestration in the product.
5. Verify the real product path through public Core APIs.

## Related

- [PROJECT_STATUS.md](PROJECT_STATUS.md)
- [integrations/PRODUCT_INTEGRATION_GUIDE.md](integrations/PRODUCT_INTEGRATION_GUIDE.md)
- [../../ecosystem/P0_2_IDENTITY_ONBOARDING_IMPLEMENTATION.md](../../ecosystem/P0_2_IDENTITY_ONBOARDING_IMPLEMENTATION.md)
- [../../ecosystem/SAAS_ROADMAP.md](../../ecosystem/SAAS_ROADMAP.md)
- [../../ecosystem/ROADMAP.md](../../ecosystem/ROADMAP.md)
- [../../ecosystem/WORKFLOW.md](../../ecosystem/WORKFLOW.md)
- [../../ecosystem/decisions/0002-core-product-separation.md](../../ecosystem/decisions/0002-core-product-separation.md)
- [../../ecosystem/decisions/0006-organization-lifecycle-and-onboarding.md](../../ecosystem/decisions/0006-organization-lifecycle-and-onboarding.md)
