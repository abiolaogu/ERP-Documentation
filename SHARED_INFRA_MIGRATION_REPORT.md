# Shared Infra Migration Report

## Completion Summary

- Root project compose files migrated to shared infra contract: `12/12`
- Workik config initialized in ERP repositories: `24/24`
- Legacy `AIDDShell` wrappers/components: `0` references, `0` files remaining
- Hardcoded module-local GraphQL defaults (`191xx`, `4000/graphql`, `8080/graphql`): `0` matches in non-doc code/config scan
- Root compose local DB/IAM/observability infra services (`postgres`, `hasura`, `keycloak`, `prometheus`, `grafana`, etc.): `0` remaining

## Rewritten Root Compose Files

- `ERP-AIOps/docker-compose.yaml`
- `ERP-Assistant/docker-compose.yml`
- `ERP-Autonomous-Coding/docker-compose.yml`
- `ERP-BSS-OSS/docker-compose.yml`
- `ERP-CRM/docker-compose.yml`
- `ERP-Church-Management/docker-compose.yml`
- `ERP-Healthcare/docker-compose.yml`
- `ERP-Marketing/docker-compose.yml`
- `ERP-SCM/docker-compose.yml`
- `ERP-School-Management/docker-compose.yml`
- `ERP-eCommerce/docker-compose.yml`
- `ERP-iPaaS/docker-compose.yml`

## Shared Infra Defaults Applied

- Hasura GraphQL: `http://localhost:8090/v1/graphql`
- Hasura WebSocket: `ws://localhost:8090/v1/graphql`
- Shared DB URL: `${ERP_SHARED_DB_URL:-postgres://erp_admin:changeme_in_production@localhost:5432/erp_shared?sslmode=disable}`
- Shared IAM URL: `${ERP_SHARED_IAM_URL:-http://localhost:8081}`
- Shared OTLP Endpoint: `${ERP_SHARED_OTLP_ENDPOINT:-http://localhost:4317}`

## Workik Config Coverage

- `ERP-AI/.workik/config.yml`
- `ERP-AIOps/.workik/config.yml`
- `ERP-Assistant/.workik/config.yml`
- `ERP-Autonomous-Coding/.workik/config.yml`
- `ERP-BI/.workik/config.yml`
- `ERP-BSS-OSS/.workik/config.yml`
- `ERP-CRM/.workik/config.yml`
- `ERP-Church-Management/.workik/config.yml`
- `ERP-Commerce/.workik/config.yml`
- `ERP-DBaaS/.workik/config.yml`
- `ERP-Finance/.workik/config.yml`
- `ERP-HCM/.workik/config.yml`
- `ERP-Healthcare/.workik/config.yml`
- `ERP-IAM/.workik/config.yml`
- `ERP-Marketing/.workik/config.yml`
- `ERP-Observability/.workik/config.yml`
- `ERP-Platform/.workik/config.yml`
- `ERP-Projects/.workik/config.yml`
- `ERP-SCM/.workik/config.yml`
- `ERP-School-Management/.workik/config.yml`
- `ERP-Web-Hosting/.workik/config.yml`
- `ERP-Workspace/.workik/config.yml`
- `ERP-eCommerce/.workik/config.yml`
- `ERP-iPaaS/.workik/config.yml`
