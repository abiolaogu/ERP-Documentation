# Company Profile and Model Assumptions

This fundraising suite is built to be institution-ready with explicit assumptions and formula transparency.

## Input Fields (Replace with Actuals)

- `COMPANY_DESCRIPTION`: [INSERT COMPANY DESCRIPTION HERE]
- `TARGET_MARKET`: [INSERT TARGET MARKET]
- `BUSINESS_MODEL`: [INSERT BUSINESS MODEL]
- `FUNDING_ASK`: [INSERT FUNDING ASK]
- `GEOGRAPHY`: [INSERT GEOGRAPHY]

## Illustrative Base Case Used in This Suite

If input fields are not populated, this suite defaults to the following model case for calculation integrity.

| Field | Base Case |
|---|---|
| Company | AI-native multi-tenant workflow operating platform for regulated enterprises |
| Product scope | Web application + API + admin control plane |
| Primary buyer | COO, CIO, VP Operations, Business Unit GM |
| Initial segment | Mid-market and enterprise organizations (500-10,000 employees) |
| Business model | Subscription (platform seats + workflow modules) + usage-based API + implementation services |
| Funding ask | $25.0M Series A |
| Geography | US (primary), UK and DACH (expansion) |

## Core Operating Assumptions

| Metric | Y1 | Y2 | Y3 | Y4 | Y5 |
|---|---:|---:|---:|---:|---:|
| Ending customers (logos) | 45 | 140 | 340 | 700 | 1,200 |
| Average ACV ($k) | 82 | 96 | 117 | 132 | 153 |
| Net revenue retention | 112% | 116% | 120% | 122% | 123% |
| Gross margin | 74% | 77% | 80% | 82% | 84% |
| Logo churn | 10% | 9% | 8% | 7% | 6% |
| Blended CAC ($k) | 160 | 145 | 125 | 110 | 95 |
| Sales cycle (days) | 145 | 132 | 120 | 112 | 104 |

## Formula Conventions

- `Revenue = Subscription + Usage + Services`
- `Gross Profit = Revenue x Gross Margin`
- `EBITDA = Gross Profit - Operating Expenses`
- `Operating Burn (monthly) = max(0, -Free Cash Flow / 12)`
- `CAC Payback (months) = CAC / Monthly Gross Profit per New Customer`
- `LTV = (ACV x Gross Margin) / Logo Churn`
- `LTV/CAC = LTV / CAC`
- `Investor MOIC = Exit Proceeds / Invested Capital`
- `Investor IRR = (MOIC^(1/holding years)) - 1`

## Revenue Mix Assumption

| Stream | Share | Rationale |
|---|---:|---|
| Subscription | 78% | Primary recurring revenue engine and valuation anchor |
| Usage/API | 17% | Scales with customer process volume and integrations |
| Services | 5% | Controlled implementation layer to accelerate deployment |

## Cost Structure Assumption

| Cost Category | Structural Driver |
|---|---|
| Cost of revenue | Cloud, support, security operations, customer success delivery |
| R&D | Product engineering, data/AI platform, quality and platform reliability |
| Sales and marketing | Pipeline generation, field sales, partnerships, sales enablement |
| G&A | Finance, legal, compliance, leadership, admin systems |

## Governance Notes

- All tables are designed for institutional review and can be exported to Excel directly.
- Assumptions are intentionally conservative on margin expansion and aggressive on execution discipline.
- Any changes in ACV, CAC, or churn should cascade through sections 03, 04, 05, and 11.
