# Sovereign Portfolio Tech Stack And Access Matrix

Date: 2026-03-01
Generated UTC: 2026-03-01T19:44:09Z

## Summary

- Repositories covered: 24
- Credential sources: `.env.example`, `web/.env.example`, README/docs where present
- If credentials are not explicitly committed, repository uses ERP-IAM shared SSO profile
- Updated credential-mode normalization (IAM-only vs dev/demo fallback): `Documentation/SOVEREIGN_REPO_LINKS_AND_LOGINS_20260301.md`
- Healthcare frontend endpoints include multiple apps (`frontend/hims-portal`, `frontend/admin-dashboard`, `frontend/call-center-desktop`, `frontend/provider-portal`, `frontend/portal`)

## Repository Matrix

### ERP-AI

- Local Path: `/Users/AbiolaOgunsakin1/ERP/ERP-AI`
- Repository Link: `https://github.com/abiolaogu/ERP-AI.git`
- Tech Stack: Go, Node.js, Hasura, React, Refine, Ant Design, GraphQL Client
- Local Frontend Links:
  - `http://localhost:5180 (web)`
- Login Credentials:
  - `SSO via ERP-IAM (no repo-specific static credentials committed)`

### ERP-AIOps

- Local Path: `/Users/AbiolaOgunsakin1/ERP/ERP-AIOps`
- Repository Link: `https://github.com/abiolaogu/ERP-AIOps.git`
- Tech Stack: Go, Rust, Node.js, Hasura, Docker Compose, React, Refine, Ant Design, GraphQL Client
- Local Frontend Links:
  - `http://localhost:5179 (web)`
- Login Credentials:
  - `SSO via ERP-IAM (no repo-specific static credentials committed)`

### ERP-Assistant

- Local Path: `/Users/AbiolaOgunsakin1/ERP/ERP-Assistant`
- Repository Link: `https://github.com/abiolaogu/ERP-Assistant.git`
- Tech Stack: Go, Node.js, Hasura, Docker Compose, React, Next.js, Refine, Ant Design, GraphQL Client
- Local Frontend Links:
  - `http://localhost:5181 (web)`
  - `http://localhost:5173 (frontend/web)`
- Login Credentials:
  - `SSO via ERP-IAM (no repo-specific static credentials committed)`

### ERP-Autonomous-Coding

- Local Path: `/Users/AbiolaOgunsakin1/ERP/ERP-Autonomous-Coding`
- Repository Link: `https://github.com/abiolaogu/ERP-Autonomous-Coding.git`
- Tech Stack: Go, Node.js, Hasura, Docker Compose, React, Refine, Ant Design, GraphQL Client
- Local Frontend Links:
  - `http://localhost:5182 (web)`
- Login Credentials:
  - `SSO via ERP-IAM (no repo-specific static credentials committed)`

### ERP-BI

- Local Path: `/Users/AbiolaOgunsakin1/ERP/ERP-BI`
- Repository Link: `https://github.com/abiolaogu/ERP-BI.git`
- Tech Stack: Node.js, Hasura, React, Refine, Ant Design, GraphQL Client
- Local Frontend Links:
  - `http://localhost:5195 (web)`
- Login Credentials:
  - `SSO via ERP-IAM (no repo-specific static credentials committed)`

### ERP-BSS-OSS

- Local Path: `/Users/AbiolaOgunsakin1/ERP/ERP-BSS-OSS`
- Repository Link: `https://github.com/abiolaogu/ERP-BSS-OSS.git`
- Tech Stack: Rust, Node.js, Hasura, Docker Compose, React, Refine, Ant Design, GraphQL Client
- Local Frontend Links:
  - `http://localhost:5183 (web)`
- Login Credentials:
  - `KEYCLOAK_ADMIN=admin / KEYCLOAK_ADMIN_PASSWORD=admin (legacy local dev docs reference)`
  - `SSO via ERP-IAM (no repo-specific static credentials committed)`

### ERP-CRM

- Local Path: `/Users/AbiolaOgunsakin1/ERP/ERP-CRM`
- Repository Link: `https://github.com/abiolaogu/ERP-CRM.git`
- Tech Stack: Rust, Node.js, Hasura, Docker Compose, React, Refine, Ant Design, GraphQL Client
- Local Frontend Links:
  - `http://localhost:5186 (web)`
- Login Credentials:
  - `SSO via ERP-IAM (no repo-specific static credentials committed)`

### ERP-Church-Management

- Local Path: `/Users/AbiolaOgunsakin1/ERP/ERP-Church-Management`
- Repository Link: `https://github.com/abiolaogu/ERP-Church-Management.git`
- Tech Stack: Go, Node.js, Hasura, Docker Compose, React, Next.js, Refine, Ant Design, GraphQL Client
- Local Frontend Links:
  - `http://localhost:5184 (web)`
  - `http://localhost:5173 (frontend/web)`
- Login Credentials:
  - `SSO via ERP-IAM (no repo-specific static credentials committed)`

### ERP-Commerce

- Local Path: `/Users/AbiolaOgunsakin1/ERP/ERP-Commerce`
- Repository Link: `https://github.com/abiolaogu/ERP-Commerce.git`
- Tech Stack: Go, Node.js, Hasura, React, Refine, Ant Design, GraphQL Client
- Local Frontend Links:
  - `http://localhost:5185 (web)`
- Login Credentials:
  - `SSO via ERP-IAM (no repo-specific static credentials committed)`

### ERP-DBaaS

- Local Path: `/Users/AbiolaOgunsakin1/ERP/ERP-DBaaS`
- Repository Link: `https://github.com/abiolaogu/ERP-DBaaS.git`
- Tech Stack: Go, Node.js, Hasura, Kubernetes, React, Refine, Ant Design, GraphQL Client
- Local Frontend Links:
  - `http://localhost:5178 (web)`
- Login Credentials:
  - `SSO via ERP-IAM (no repo-specific static credentials committed)`

### ERP-Finance

- Local Path: `/Users/AbiolaOgunsakin1/ERP/ERP-Finance`
- Repository Link: `https://github.com/abiolaogu/ERP-Finance.git`
- Tech Stack: Go, Node.js, Hasura, React, Refine, Ant Design, GraphQL Client
- Local Frontend Links:
  - `http://localhost:5187 (web)`
- Login Credentials:
  - `SSO via ERP-IAM (no repo-specific static credentials committed)`

### ERP-HCM

- Local Path: `/Users/AbiolaOgunsakin1/ERP/ERP-HCM`
- Repository Link: `https://github.com/abiolaogu/ERP-HCM.git`
- Tech Stack: Go, Node.js, Hasura, React, Refine, Ant Design, GraphQL Client
- Local Frontend Links:
  - `http://localhost:3012 (web)`
- Login Credentials:
  - `SSO via ERP-IAM (no repo-specific static credentials committed)`

### ERP-Healthcare

- Local Path: `/Users/AbiolaOgunsakin1/ERP/ERP-Healthcare`
- Repository Link: `https://github.com/abiolaogu/ERP-Healthcare.git`
- Tech Stack: Go, Hasura, Docker Compose, Kubernetes
- Local Frontend Links:
  - `No dedicated web app detected`
- Login Credentials:
  - `SSO via ERP-IAM (no repo-specific static credentials committed)`

### ERP-IAM

- Local Path: `/Users/AbiolaOgunsakin1/ERP/ERP-IAM`
- Repository Link: `https://github.com/abiolaogu/ERP-IAM.git`
- Tech Stack: Go, Node.js, Hasura, React, Refine, Ant Design, GraphQL Client
- Local Frontend Links:
  - `http://localhost:3011 (web)`
- Login Credentials:
  - `SSO via ERP-IAM (no repo-specific static credentials committed)`

### ERP-Marketing

- Local Path: `/Users/AbiolaOgunsakin1/ERP/ERP-Marketing`
- Repository Link: `https://github.com/abiolaogu/ERP-Marketing.git`
- Tech Stack: Rust, Node.js, Hasura, Docker Compose, React, Refine, Ant Design, GraphQL Client
- Local Frontend Links:
  - `http://localhost:5188 (web)`
- Login Credentials:
  - `SSO via ERP-IAM (no repo-specific static credentials committed)`

### ERP-Observability

- Local Path: `/Users/AbiolaOgunsakin1/ERP/ERP-Observability`
- Repository Link: `https://github.com/abiolaogu/ERP-Observability.git`
- Tech Stack: Go, Node.js, Hasura, Kubernetes, React, Refine, Ant Design, GraphQL Client
- Local Frontend Links:
  - `http://localhost:5177 (web)`
- Login Credentials:
  - `SSO via ERP-IAM (no repo-specific static credentials committed)`

### ERP-Platform

- Local Path: `/Users/AbiolaOgunsakin1/ERP/ERP-Platform`
- Repository Link: `https://github.com/abiolaogu/ERP-Platform.git`
- Tech Stack: Node.js, Hasura, React, Next.js, Refine, Ant Design, GraphQL Client
- Local Frontend Links:
  - `http://localhost:3010 (web)`
  - `http://localhost:5173 (apps/web)`
- Login Credentials:
  - `SSO via ERP-IAM (no repo-specific static credentials committed)`

### ERP-Projects

- Local Path: `/Users/AbiolaOgunsakin1/ERP/ERP-Projects`
- Repository Link: `https://github.com/abiolaogu/ERP-Projects.git`
- Tech Stack: Go, Node.js, Hasura, React, Refine, Ant Design, GraphQL Client
- Local Frontend Links:
  - `http://localhost:5189 (web)`
- Login Credentials:
  - `SSO via ERP-IAM (no repo-specific static credentials committed)`

### ERP-SCM

- Local Path: `/Users/AbiolaOgunsakin1/ERP/ERP-SCM`
- Repository Link: `https://github.com/abiolaogu/ERP-SCM.git`
- Tech Stack: Node.js, Hasura, Docker Compose, React, Refine, Ant Design, GraphQL Client
- Local Frontend Links:
  - `http://localhost:5190 (web)`
  - `http://localhost:5173 (frontend)`
- Login Credentials:
  - `SSO via ERP-IAM (no repo-specific static credentials committed)`

### ERP-School-Management

- Local Path: `/Users/AbiolaOgunsakin1/ERP/ERP-School-Management`
- Repository Link: `https://github.com/abiolaogu/ERP-School-Management.git`
- Tech Stack: Go, Node.js, Hasura, Docker Compose, Kubernetes, React, Next.js
- Local Frontend Links:
  - `http://localhost:5173 (apps/web)`
- Login Credentials:
  - `SSO via ERP-IAM (no repo-specific static credentials committed)`

### ERP-Web-Hosting

- Local Path: `/Users/AbiolaOgunsakin1/ERP/ERP-Web-Hosting`
- Repository Link: `https://github.com/abiolaogu/ERP-Web-Hosting.git`
- Tech Stack: Python, Node.js, Hasura, React, Refine, Ant Design, GraphQL Client
- Local Frontend Links:
  - `http://localhost:5187 (web)`
- Login Credentials:
  - `SSO via ERP-IAM (no repo-specific static credentials committed)`

### ERP-Workspace

- Local Path: `/Users/AbiolaOgunsakin1/ERP/ERP-Workspace`
- Repository Link: `https://github.com/abiolaogu/ERP-Workspace.git`
- Tech Stack: Go, Node.js, Hasura, React, Refine, Ant Design, GraphQL Client
- Local Frontend Links:
  - `http://localhost:5191 (web)`
- Login Credentials:
  - `SSO via ERP-IAM (no repo-specific static credentials committed)`

### ERP-eCommerce

- Local Path: `/Users/AbiolaOgunsakin1/ERP/ERP-eCommerce`
- Repository Link: `https://github.com/abiolaogu/ERP-eCommerce.git`
- Tech Stack: Node.js, Hasura, Docker Compose, React, Refine, Ant Design, GraphQL Client
- Local Frontend Links:
  - `http://localhost:5192 (web)`
- Login Credentials:
  - `SSO via ERP-IAM (no repo-specific static credentials committed)`

### ERP-iPaaS

- Local Path: `/Users/AbiolaOgunsakin1/ERP/ERP-iPaaS`
- Repository Link: `https://github.com/abiolaogu/ERP-iPaaS.git`
- Tech Stack: Node.js, Hasura, Docker Compose, Kubernetes, React, Refine, Ant Design, GraphQL Client
- Local Frontend Links:
  - `http://localhost:5193 (web)`
- Login Credentials:
  - `SSO via ERP-IAM (no repo-specific static credentials committed)`
