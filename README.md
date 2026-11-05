# KYROX Platform

**KYROX Platform is the single documentation and knowledge hub for the entire KYROX ecosystem.**

Human-readable rules, standards, architecture decisions, project documentation, status, roadmap and AI operating instructions live here. Application repositories (`kyrox-core`, `kyrox-fair-crm`, and future products) are implementation and verification repositories: code, tests, migrations, CI and machine-readable runtime/quality contracts may live there, but human/AI documentation must not be duplicated there.

## First read — humans and AI agents

When you are given the KYROX ecosystem, **start with this repository** and follow the reading order in [AGENTS.md](AGENTS.md). Where each kind of knowledge belongs, the platform doctrine and precedence rules: [ecosystem/DOCUMENT_GOVERNANCE.md](ecosystem/DOCUMENT_GOVERNANCE.md).

## Repositories

| Repository | Role |
|------------|------|
| [kyrox-platform](.) | **All human/AI documentation**, shared standards, project docs, ADRs, roadmap, status |
| [kyrox-core](https://github.com/kyrox-studio/kyrox-core) | Shared SaaS backend implementation, tests, migrations and CI |
| [kyrox-fair-crm](https://github.com/kyrox-studio/kyrox-fair-crm) | Fair CRM implementation, tests, migrations, CI and machine-readable feature contracts |
| [kyrox-fair-stand](https://github.com/kyrox-studio/kyrox-fair-stand) | Fair Stand konfigüratörü, katalog API'si ve `fair_stand` veritabanı |
| [kyrox-brand](https://github.com/kyrox-studio/kyrox-brand) | Marka varlıkları: logo dosyaları ve renk token'ları |
| [kyrox-web](https://github.com/kyrox-studio/kyrox-web) | kyrox.studio WordPress teması ve sunucu kurulumu |

## Knowledge layout

`standards/`, `projects/<product>/`, `ecosystem/` and `archive/` are defined in the classification table of [DOCUMENT_GOVERNANCE.md](ecosystem/DOCUMENT_GOVERNANCE.md#classification-rule).

## Web ve ürünler için yayına hazırlık

[Web ve Ürünler — Yayına Hazırlık](ecosystem/WEB_LAUNCH_CHECKLIST.md): Kyrox web sitesi, Fair-CRM ve Fair-Stand için basitten zora, tek tek ilerlenen kontrol ve iş listesi.

[Tarayıcı Depolaması ve Dış Servisler](ecosystem/WEB_STORAGE_AND_SERVICES.md): Canlı anonim ziyaret kanıtı, oturum/depolama kayıtları ve gizlilik metinlerine temel olacak servis envanteri.

## Important shared standards

- [CRUD & UI Authorization Standard](standards/ui/CRUD_UI_AUTHORIZATION_STANDARD.md) — reusable CRUD permission semantics and UI visibility/action rules.
- [Backend Architecture Standards](standards/backend/BACKEND_ARCHITECTURE_STANDARDS.md)

For the complete index, use [standards/README.md](standards/README.md).
