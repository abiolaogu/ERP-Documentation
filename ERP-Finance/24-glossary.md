# ERP-Finance Glossary

## Document Information

| Field | Value |
|-------|-------|
| Module | ERP-Finance |
| Document Type | Glossary |
| Version | 1.0.0 |
| Last Updated | 2026-02-23 |

## Financial Terms

| Term | Definition |
|------|-----------|
| **Accounts Payable (AP)** | Amounts owed by the organization to vendors/suppliers for goods or services received but not yet paid. |
| **Accounts Receivable (AR)** | Amounts owed to the organization by customers for goods or services delivered but not yet collected. |
| **Accrual Accounting** | Accounting method where revenue and expenses are recorded when earned/incurred, regardless of cash movement. |
| **Aging Report** | Report showing outstanding invoices grouped by age buckets (current, 30, 60, 90, 120+ days). |
| **ARR (Annual Recurring Revenue)** | Annualized value of recurring subscription revenue. ARR = MRR x 12. |
| **ASC 606** | US GAAP standard for revenue recognition from contracts with customers. |
| **Balance Sheet** | Financial statement showing assets, liabilities, and equity at a point in time. |
| **Chart of Accounts (COA)** | Complete listing of all accounts used to classify financial transactions. |
| **Consolidation** | Combining financial statements of multiple entities into one group report with elimination entries. |
| **Credit Note** | Document reducing the amount owed by a customer, typically issued for returns or adjustments. |
| **Debit/Credit** | Double-entry bookkeeping system: debits increase assets/expenses, credits increase liabilities/equity/revenue. |
| **Depreciation** | Systematic allocation of an asset's cost over its useful life. |
| **Double-Declining Balance** | Accelerated depreciation method using twice the straight-line rate applied to book value. |
| **Dunning** | The process of sending systematic reminders to customers about overdue payments. |
| **EBITDA** | Earnings Before Interest, Taxes, Depreciation, and Amortization. |
| **Fiscal Period** | A defined accounting period (month, quarter, year) for financial reporting. |
| **FX (Foreign Exchange)** | Currency conversion between different monetary units. |
| **GAAP** | Generally Accepted Accounting Principles -- standard US accounting framework. |
| **General Ledger (GL)** | The master record of all financial transactions, serving as the single source of truth. |
| **Gross Margin** | Revenue minus Cost of Goods Sold, expressed as percentage of revenue. |
| **IFRS** | International Financial Reporting Standards -- global accounting framework. |
| **IFRS 15** | International standard for revenue recognition (equivalent to ASC 606). |
| **Immutable Ledger** | A ledger where posted entries cannot be modified or deleted; corrections require reversing entries. |
| **Income Statement** | Financial statement showing revenue, expenses, and profit/loss over a period. |
| **Invoice** | Formal document requesting payment for goods/services delivered. |
| **Journal Entry** | A record of a financial transaction with balanced debit and credit lines. |
| **MRR (Monthly Recurring Revenue)** | Total recurring revenue normalized to a monthly value. |
| **Net Profit Margin** | Net income divided by total revenue, expressed as percentage. |
| **Opening Balance** | The starting balance of an account at the beginning of an accounting period. |
| **Overage** | Usage exceeding the included allowance in a subscription plan, subject to additional charges. |
| **Per-Diem** | Daily allowance for expenses (meals, lodging) during business travel. |
| **Period Close** | The process of finalizing all transactions for an accounting period and locking it against further changes. |
| **Proration** | Calculating partial charges when a subscription changes mid-billing-cycle. |
| **Retained Earnings** | Cumulative net income minus dividends, representing reinvested profits. |
| **Revenue Recognition** | The process of recording revenue in the accounting period when it is earned. |
| **Reversing Entry** | A journal entry that offsets a previously posted entry, used for corrections. |
| **ROA (Return on Assets)** | Net income divided by total assets, measuring asset efficiency. |
| **ROE (Return on Equity)** | Net income divided by shareholders' equity, measuring equity returns. |
| **Salvage Value** | Estimated residual value of an asset at the end of its useful life. |
| **Straight-Line Depreciation** | Equal depreciation amount each period: (Cost - Salvage) / Useful Life. |
| **Sub-Ledger** | Detailed ledger for a specific account type (AP sub-ledger, AR sub-ledger). |
| **Sum-of-Years Digits** | Accelerated depreciation using fraction of remaining useful life over sum of all years. |
| **Three-Way Matching** | Verification that a purchase order, goods receipt, and vendor invoice all agree. |
| **Trial Balance** | Report listing all accounts with their debit or credit balances; total debits must equal total credits. |
| **Units of Production** | Depreciation based on actual usage rather than time elapsed. |
| **Useful Life** | The estimated period over which an asset will provide economic benefit. |
| **Variance Analysis** | Comparison of actual financial results to budgeted amounts. |
| **VAT (Value Added Tax)** | Consumption tax levied on the value added at each stage of production/distribution. |
| **WHT (Withholding Tax)** | Tax deducted at source before payment to the recipient. |
| **Zero-Based Budgeting** | Budgeting method requiring justification of every expense from zero, not from prior year. |

## Technical Terms

| Term | Definition |
|------|-----------|
| **AIDD** | AI-Driven Development -- methodology where AI assists in development with guardrails. |
| **Axum** | Rust web framework used by the billing and payments engines. |
| **CloudEvents** | Specification for describing event data in a common, interoperable format. |
| **CQRS** | Command Query Responsibility Segregation -- separate models for reads and writes. |
| **DDD** | Domain-Driven Design -- software design approach focusing on business domain modeling. |
| **Event Sourcing** | Storing state changes as a sequence of events rather than current state. |
| **FastAPI** | Python async web framework used by the asset management service. |
| **Idempotency Key** | A unique identifier ensuring an operation is processed only once, even if retried. |
| **JetStream** | NATS durable streaming layer providing at-least-once delivery guarantees. |
| **mTLS** | Mutual TLS -- both client and server authenticate each other with certificates. |
| **NATS** | Lightweight messaging system used as the event backbone. |
| **PCI-DSS** | Payment Card Industry Data Security Standard. |
| **RLS** | Row-Level Security -- PostgreSQL feature for row-level access control. |
| **Saga Pattern** | Distributed transaction pattern using a sequence of local transactions with compensations. |
| **SQLx** | Rust async SQL toolkit with compile-time query verification. |
| **Tokenization** | Replacing sensitive data (card numbers) with non-sensitive tokens. |
| **UUID v7** | Time-ordered universally unique identifier, database-friendly for indexing. |
