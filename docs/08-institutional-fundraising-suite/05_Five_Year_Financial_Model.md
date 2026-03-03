# 5-Year Financial Model

All figures in USD millions unless otherwise noted.

## 1. Key Modeling Assumptions

| Assumption | Y1 | Y2 | Y3 | Y4 | Y5 |
|---|---:|---:|---:|---:|---:|
| Ending customers | 45 | 140 | 340 | 700 | 1,200 |
| Average ACV ($k) | 82 | 96 | 117 | 132 | 153 |
| Gross margin | 74% | 77% | 80% | 82% | 84% |
| Logo churn | 10% | 9% | 8% | 7% | 6% |
| NRR | 112% | 116% | 120% | 122% | 123% |
| Blended CAC ($k) | 160 | 145 | 125 | 110 | 95 |

## 2. Revenue Build

| Revenue Line | Y1 | Y2 | Y3 | Y4 | Y5 |
|---|---:|---:|---:|---:|---:|
| Subscription revenue | 3.74 | 11.39 | 31.04 | 72.07 | 143.52 |
| Usage/API revenue | 0.82 | 2.48 | 6.77 | 15.71 | 31.28 |
| Services revenue | 0.24 | 0.73 | 1.99 | 4.62 | 9.20 |
| **Total revenue** | **4.80** | **14.60** | **39.80** | **92.40** | **184.00** |

Formula:

- `Subscription = Total x 78%`
- `Usage = Total x 17%`
- `Services = Total x 5%`

## 3. Cost Structure and Profitability

| P&L Line | Y1 | Y2 | Y3 | Y4 | Y5 |
|---|---:|---:|---:|---:|---:|
| Total revenue | 4.80 | 14.60 | 39.80 | 92.40 | 184.00 |
| Cost of revenue | 1.25 | 3.36 | 7.96 | 16.63 | 29.44 |
| **Gross profit** | **3.55** | **11.24** | **31.84** | **75.77** | **154.56** |
| R&D | 6.00 | 8.50 | 12.00 | 18.00 | 26.00 |
| Sales and marketing | 7.20 | 12.50 | 20.00 | 32.00 | 48.00 |
| G&A | 2.80 | 3.90 | 5.80 | 8.20 | 11.50 |
| **Total opex** | **16.00** | **24.90** | **37.80** | **58.20** | **85.50** |
| **EBITDA** | **(12.45)** | **(13.66)** | **(5.96)** | **17.57** | **69.06** |

## 4. Cash Flow and Burn

| Cash Flow Line | Y1 | Y2 | Y3 | Y4 | Y5 |
|---|---:|---:|---:|---:|---:|
| EBITDA | (12.45) | (13.66) | (5.96) | 17.57 | 69.06 |
| Capex | (0.80) | (1.20) | (2.00) | (3.50) | (5.00) |
| Working capital delta | (0.05) | (0.08) | (0.04) | (1.57) | (16.06) |
| Taxes | 0.00 | 0.00 | 0.00 | 0.00 | 0.00 |
| **Free cash flow** | **(13.30)** | **(14.94)** | **(8.00)** | **12.50** | **48.00** |
| Monthly burn (if negative) | 1.11 | 1.25 | 0.67 | 0.00 | 0.00 |

Runway logic:

- Starting cash assumed: $3.0M
- Raise: $25.0M
- Available cash: $28.0M
- Cumulative negative FCF before breakeven (Y1-Y3): $36.24M
- Required operating actions: phased hiring, spend gating, and working capital discipline to maintain >=24 months runway.

## 5. CAC, LTV, and Efficiency

| Metric | Y1 | Y2 | Y3 | Y4 | Y5 |
|---|---:|---:|---:|---:|---:|
| Blended CAC ($k) | 160 | 145 | 125 | 110 | 95 |
| ACV ($k) | 82 | 96 | 117 | 132 | 153 |
| Gross margin | 74% | 77% | 80% | 82% | 84% |
| Logo churn | 10% | 9% | 8% | 7% | 6% |
| LTV ($k) | 607 | 821 | 1,170 | 1,547 | 2,142 |
| LTV/CAC | 3.8x | 5.7x | 9.4x | 14.1x | 22.5x |
| CAC payback (months) | 31.7 | 23.6 | 16.0 | 12.2 | 8.9 |

Formula references:

- `LTV = (ACV x Gross Margin) / Logo Churn`
- `CAC Payback = CAC / ((ACV x Gross Margin)/12)`

## 6. Sensitivity Analysis

### Revenue sensitivity (Y5)

| Scenario | ACV vs Base | New Logo Adds vs Base | NRR | Y5 Revenue |
|---|---:|---:|---:|---:|
| Downside | -15% | -20% | 114% | 123.0 |
| Base | 0% | 0% | 123% | 184.0 |
| Upside | +12% | +15% | 128% | 251.0 |

### EBITDA sensitivity (Y5)

| Scenario | Gross Margin | S&M as % Revenue | Y5 EBITDA |
|---|---:|---:|---:|
| Downside | 79% | 29% | 32.0 |
| Base | 84% | 26% | 69.1 |
| Upside | 86% | 24% | 92.0 |

## 7. Investor Return Model (Series A)

Assume:

- Invested capital: $25.0M
- Initial ownership post-money: 25%
- Fully diluted ownership at exit after future dilution: 18%
- Holding period: 5 years

| Scenario | Exit EV | Investor Proceeds (18%) | MOIC | IRR |
|---|---:|---:|---:|---:|
| Downside | 600 | 108 | 4.3x | 33.8% |
| Base | 1,300 | 234 | 9.4x | 56.5% |
| Upside | 2,400 | 432 | 17.3x | 76.5% |

IRR formula:

`IRR = (MOIC^(1/5)) - 1`

## 8. Validation and Control Notes

- Model must be re-forecast monthly against pipeline and hiring realities.
- Board should track variance on revenue, CAC, payback, and gross margin as primary control levers.
- Any deviation >10% against plan should trigger scenario recast and spend reallocation.
