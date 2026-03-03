# GitHub Native Factory Documentation

This repository is the documentation-as-code system for a multi-tenant SaaS Product delivered through a GitHub-native workflow. It connects business intent to architecture, implementation, operations, compliance, and design execution.

## Start Here

1. [Documentation Index](docs/README.md)
2. [Business Requirements (BRD)](docs/00-project-strategy/BRD.md)
3. [Product Requirements (PRD)](docs/00-project-strategy/PRD.md)
4. [Software Architecture Document (SAD)](docs/01-architecture/SAD.md)
5. [CI/CD and Environment Promotion](docs/03-quality-devops/CI-CD.md)
6. [Operations Manual](docs/04-user-ops/Operations-Manual.md)
7. [Project-Labeled Category-King Suites](docs/06-project-category-king/README.md)
8. [Centralized Project Source Packs](docs/07-project-source-docs/README.md)
9. [Institutional Fundraising Suite](docs/08-institutional-fundraising-suite/README.md)

## Quick Start (15 Minutes)

1. Read [Repo Onboarding](docs/02-developer-onboarding/Repo-README.md).
2. Set local prerequisites from [Environment Setup](docs/02-developer-onboarding/Environment-Setup.md).
3. Review [Contributing Guide](docs/02-developer-onboarding/CONTRIBUTING.md).
4. Open [OpenAPI Spec](docs/01-architecture/API/openapi.yaml).
5. Use the [PR template](.github/pull_request_template.md) for all changes.

## System Scope

The System documented here represents a realistic enterprise SaaS with:

- Multi-tenant web application, API, and Admin portal
- Tenant-aware Org and User management with RBAC
- Audit logging and observability by Environment
- Billing lifecycle and subscription management
- Notifications and external integrations
- Secure deployment on Harvester HCI + Rancher + Fleet + Coolify

## Governance

- Architectural decisions are recorded in [ADRs](docs/02-developer-onboarding/ADR/).
- Compliance evidence mapping is maintained in [Compliance Matrix](docs/00-project-strategy/Compliance-Regulatory-Matrix.md).
- Security reporting process is defined in [SECURITY.md](SECURITY.md).

## Project-Labeled Documentation Convention

- Strategy files: `<PROJECT>_Strategy_Category_King_Blueprint.md`
- Architecture files: `<PROJECT>_Architecture_Category_King_Blueprint.md`
- Figma prompt files: `<PROJECT>_Figma_Category_King_Prompts.md`
- Source packs: `<PROJECT>_Source_Pack/`
