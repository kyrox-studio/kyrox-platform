# AGENTS.md — KYROX knowledge entry

This file is the mandatory entry point for AI agents and a concise routing guide for humans working anywhere in the KYROX ecosystem.

## Fundamental rule

The platform doctrine is defined in [ecosystem/DOCUMENT_GOVERNANCE.md](ecosystem/DOCUMENT_GOVERNANCE.md#platform-doctrine). Do not treat documentation, chat history, local conventions or tool-specific rule files in `kyrox-fair-crm` / `kyrox-core` / `kyrox-fair-stand` as independent sources of truth for ecosystem ownership. Human-readable ecosystem rules belong here.

## Required reading order

1. [ecosystem/DOCUMENT_GOVERNANCE.md](ecosystem/DOCUMENT_GOVERNANCE.md)
2. [standards/README.md](standards/README.md)
3. [ecosystem/WORKFLOW.md](ecosystem/WORKFLOW.md)
4. For every material product/platform delivery, read [ecosystem/SAAS_ROADMAP.md](ecosystem/SAAS_ROADMAP.md) and apply the SaaS-impact classification from the shared Feature Delivery / Applicability standards.
5. Open the owning project entry:
   - Fair CRM → [projects/fair-crm/README.md](projects/fair-crm/README.md)
   - Fair Stand → [projects/fair-stand/README.md](projects/fair-stand/README.md)
   - Core → [projects/kyrox-core/README.md](projects/kyrox-core/README.md)
6. If the task changes current state or planned work, also read:
   - [ecosystem/STATUS.md](ecosystem/STATUS.md)
   - [ecosystem/ROADMAP.md](ecosystem/ROADMAP.md)

## Ortak güvenlik girişi

Yeni proje ve mevcut değişiklikler için [Güvenlik Kuralları, Yol Haritası ve Kanıt Kaydı](standards/security/SECURITY_PROGRAM.md) okunur. Güvenlik kontrolü/bulgusu burada tek kayıtla izlenir; uygulanabilir kural çakışması, eksik test veya kanıt teslimattan önce bildirilir. Otomatik denetimin mevcut sınırı aynı belgede açıklanır. Uçtan uca güvenlik ve karar ↔ kod uyum denetimi için tekrar çalıştırılabilir prosedür: [kyrox-security-audit skill'i](.claude/skills/kyrox-security-audit/SKILL.md).

## How to decide what to read

- **Reusable behavior across products** → `standards/` first.
- **Fair CRM-only behavior/domain/UI** → `projects/fair-crm/` after applicable shared standards.
- **Fair Stand Item/Category/configurator ownership** → `projects/fair-stand/` after applicable shared standards and [ADR-0007](ecosystem/decisions/0007-fair-stand-product-ownership.md).
- **Core-only architecture/API behavior** → `projects/kyrox-core/` after applicable shared standards.
- **Cross-repo strategy/workflow/ownership** → `ecosystem/`.
- **Historical material** → `archive/`; never use it as a live rule unless a canonical document explicitly says so.

For CRUD, permission-driven UI visibility, route/action gating and backend authorization, read [standards/ui/CRUD_UI_AUTHORIZATION_STANDARD.md](standards/ui/CRUD_UI_AUTHORIZATION_STANDARD.md) before any product-specific UI document.

For material feature/platform work, SaaS readiness is not a separate optional cleanup phase: classify organization isolation, permission scope, Core/product ownership, entitlement/usage/lifecycle/security/job/data-lifecycle impact before implementation and prove every affected dimension before DONE.

## Precedence and hard rules

Precedence, anti-duplication and placement rules: [ecosystem/DOCUMENT_GOVERNANCE.md](ecosystem/DOCUMENT_GOVERNANCE.md#precedence-and-overlap). This repository contains the human/AI knowledge base; do not add application code here.
