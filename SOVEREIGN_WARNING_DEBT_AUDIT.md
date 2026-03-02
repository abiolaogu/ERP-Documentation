# Sovereign Warning Debt Audit

Date: 2026-03-01T17:00:54Z
Source log: `/tmp/sovereign_ga_all_phase11_warning_debt_elimination.txt`

## Warning Matrix

| Repository | Router Future | AntD Deprecated | List Deprecated | React act Warning | localstorage-file Flag | Chunk >500k | Circular Chunks |
| --- | --- | --- | --- | --- | --- | --- | --- |
| ERP-Observability | 0 | 0 | 0 | 0 | 0 | 0 | 0 |
| ERP-Projects | 0 | 0 | 0 | 0 | 0 | 0 | 0 |
| ERP-AI | 0 | 0 | 0 | 0 | 0 | 0 | 0 |
| ERP-CRM | 0 | 0 | 0 | 0 | 0 | 0 | 0 |
| ERP-Web-Hosting | 0 | 0 | 0 | 0 | 0 | 0 | 0 |
| ERP-BSS-OSS | 0 | 0 | 0 | 0 | 1 | 0 | 0 |
| ERP-Church-Management | 0 | 0 | 0 | 0 | 0 | 0 | 0 |
| ERP-Finance | 0 | 0 | 0 | 0 | 0 | 0 | 0 |
| ERP-Platform | 0 | 0 | 0 | 0 | 0 | 0 | 0 |
| ERP-Assistant | 0 | 0 | 0 | 0 | 0 | 0 | 0 |
| ERP-SCM | 0 | 0 | 0 | 0 | 0 | 0 | 0 |
| ERP-Commerce | 0 | 0 | 0 | 0 | 0 | 0 | 0 |
| ERP-DBaaS | 0 | 0 | 0 | 0 | 0 | 0 | 0 |
| ERP-iPaaS | 0 | 0 | 0 | 0 | 1 | 0 | 0 |
| ERP-Marketing | 0 | 0 | 0 | 0 | 1 | 0 | 0 |
| ERP-BI | 0 | 0 | 0 | 0 | 0 | 0 | 0 |
| ERP-Autonomous-Coding | 0 | 0 | 0 | 0 | 0 | 0 | 0 |
| ERP-Workspace | 0 | 0 | 0 | 0 | 0 | 0 | 0 |
| ERP-eCommerce | 0 | 0 | 0 | 0 | 1 | 0 | 0 |
| ERP-AIOps | 0 | 0 | 0 | 0 | 0 | 0 | 0 |
| ERP-IAM | 0 | 0 | 0 | 0 | 0 | 0 | 0 |
| ERP-HCM | 0 | 0 | 0 | 0 | 0 | 0 | 0 |

## Totals

- Repositories observed in log: 22
- Router future warnings: 0
- AntD deprecation warnings: 0
- List deprecation warnings: 0
- React act warnings: 0
- localstorage-file flag warnings: 4
- Chunk >500k warnings: 0
- Circular chunk warnings: 0

## Notes

- `localstorage-file` warnings are environment/runtime flag noise and not application code failures.
- Chunk-size and circular-chunk warnings are build optimization debt; they are non-blocking for GA hard gates.
