# ERP-Finance Entity Relationship Diagram

## Document Information

| Field | Value |
|-------|-------|
| Module | ERP-Finance |
| Document Type | Entity Relationship Diagram |
| Version | 1.0.0 |
| Last Updated | 2026-02-23 |
| Total Tables | 127 |

## Database Schema Overview

The ERP-Finance database spans 10 schemas with 127+ tables implementing an immutable posting ledger architecture. All tables include `tenant_id` for multi-tenancy with PostgreSQL Row-Level Security.

## Core ERD

```mermaid
erDiagram
    %% ─── GENERAL LEDGER ──────────────────────────────
    gl_fiscal_years ||--o{ gl_fiscal_periods : contains
    gl_fiscal_periods ||--o{ gl_journal_entries : "posted in"
    gl_chart_of_accounts ||--o{ gl_journal_entry_lines : "debits/credits"
    gl_journal_entries ||--o{ gl_journal_entry_lines : contains
    gl_journal_entries ||--o| gl_journal_entries : "reversed by"
    gl_chart_of_accounts ||--o| gl_chart_of_accounts : "parent"
    gl_chart_of_accounts ||--o{ gl_account_balances : "period balance"
    gl_currencies ||--o{ gl_exchange_rates : "rate history"
    gl_entities ||--o{ gl_consolidation_entries : "multi-entity"

    gl_fiscal_years {
        uuid id PK
        uuid tenant_id FK
        varchar name
        date start_date
        date end_date
        varchar status
    }

    gl_fiscal_periods {
        uuid id PK
        uuid fiscal_year_id FK
        int period_number
        date start_date
        date end_date
        varchar status
        boolean is_closed
        timestamp closed_at
        uuid closed_by
    }

    gl_chart_of_accounts {
        uuid id PK
        uuid tenant_id FK
        varchar account_number
        varchar account_name
        varchar account_type
        uuid parent_account_id FK
        varchar currency
        boolean is_active
        int level
        varchar normal_balance
    }

    gl_journal_entries {
        uuid id PK
        uuid tenant_id FK
        varchar journal_number
        date posting_date
        uuid fiscal_period_id FK
        varchar description
        varchar posting_status
        varchar source_module
        uuid source_document_id
        uuid reversal_of_id FK
        varchar reversal_reason
        uuid created_by
        uuid posted_by
        timestamp created_at
        timestamp posted_at
    }

    gl_journal_entry_lines {
        uuid id PK
        uuid journal_entry_id FK
        uuid account_id FK
        numeric debit_amount
        numeric credit_amount
        varchar currency
        numeric exchange_rate
        numeric functional_amount
        varchar description
        varchar dimension_1
        varchar dimension_2
        int line_number
    }

    gl_account_balances {
        uuid id PK
        uuid account_id FK
        uuid fiscal_period_id FK
        numeric opening_balance
        numeric debit_total
        numeric credit_total
        numeric closing_balance
        varchar currency
    }

    gl_currencies {
        uuid id PK
        varchar code
        varchar name
        int decimal_places
        boolean is_active
    }

    gl_exchange_rates {
        uuid id PK
        varchar from_currency FK
        varchar to_currency FK
        numeric rate
        date effective_date
        varchar source
    }

    gl_entities {
        uuid id PK
        uuid tenant_id FK
        varchar name
        varchar code
        varchar functional_currency
        uuid parent_entity_id
        boolean is_active
    }

    gl_consolidation_entries {
        uuid id PK
        uuid entity_id FK
        uuid journal_entry_id FK
        varchar elimination_type
        numeric amount
    }

    %% ─── ACCOUNTS PAYABLE ────────────────────────────
    ap_vendors ||--o{ ap_invoices : "billed by"
    ap_vendors ||--o{ ap_vendor_bank_accounts : "paid to"
    ap_invoices ||--o{ ap_invoice_lines : contains
    ap_invoices ||--o{ ap_payments : "paid via"
    ap_invoices ||--o{ ap_matching_records : "matched"
    ap_purchase_orders ||--o{ ap_po_lines : contains
    ap_goods_receipts ||--o{ ap_receipt_lines : contains
    ap_payment_runs ||--o{ ap_payment_run_items : contains
    ap_invoices ||--o{ ap_approval_records : "approved by"

    ap_vendors {
        uuid id PK
        uuid tenant_id FK
        varchar vendor_code
        varchar name
        varchar tax_id
        varchar payment_terms
        varchar currency
        varchar email
        text address
        varchar status
        numeric credit_limit
    }

    ap_vendor_bank_accounts {
        uuid id PK
        uuid vendor_id FK
        varchar bank_name
        varchar account_number
        varchar routing_number
        varchar swift_code
        boolean is_primary
    }

    ap_invoices {
        uuid id PK
        uuid tenant_id FK
        uuid vendor_id FK
        varchar invoice_number
        date invoice_date
        date due_date
        numeric subtotal
        numeric tax_amount
        numeric total
        varchar currency
        varchar status
        varchar matching_status
        uuid gl_journal_id FK
        text ocr_data_json
        float ocr_confidence
    }

    ap_invoice_lines {
        uuid id PK
        uuid invoice_id FK
        varchar description
        numeric quantity
        numeric unit_price
        numeric amount
        uuid gl_account_id FK
        uuid po_line_id FK
    }

    ap_purchase_orders {
        uuid id PK
        uuid tenant_id FK
        uuid vendor_id FK
        varchar po_number
        date order_date
        numeric total
        varchar status
    }

    ap_po_lines {
        uuid id PK
        uuid po_id FK
        varchar description
        numeric quantity
        numeric unit_price
        numeric amount
    }

    ap_goods_receipts {
        uuid id PK
        uuid po_id FK
        date receipt_date
        varchar status
    }

    ap_receipt_lines {
        uuid id PK
        uuid receipt_id FK
        uuid po_line_id FK
        numeric quantity_received
    }

    ap_matching_records {
        uuid id PK
        uuid invoice_id FK
        uuid po_id FK
        uuid receipt_id FK
        varchar match_type
        varchar status
        numeric variance_amount
        timestamp matched_at
    }

    ap_payments {
        uuid id PK
        uuid invoice_id FK
        numeric amount
        date payment_date
        varchar payment_method
        varchar reference
        uuid gl_journal_id FK
    }

    ap_payment_runs {
        uuid id PK
        uuid tenant_id FK
        date run_date
        varchar status
        numeric total_amount
        int invoice_count
        uuid created_by
    }

    ap_payment_run_items {
        uuid id PK
        uuid payment_run_id FK
        uuid invoice_id FK
        numeric amount
        varchar status
    }

    ap_approval_records {
        uuid id PK
        uuid invoice_id FK
        uuid approver_id FK
        varchar decision
        text comments
        timestamp decided_at
    }

    %% ─── ACCOUNTS RECEIVABLE ─────────────────────────
    ar_customers ||--o{ ar_invoices : "billed to"
    ar_invoices ||--o{ ar_invoice_lines : contains
    ar_invoices ||--o{ ar_payments_received : "payments"
    ar_invoices ||--o{ ar_dunning_records : "dunned"
    ar_customers ||--o{ ar_credit_notes : "credited"
    ar_customers ||--o{ ar_credit_limits : "limit"
    ar_revenue_contracts ||--o{ ar_performance_obligations : "obligations"

    ar_customers {
        uuid id PK
        uuid tenant_id FK
        varchar customer_code
        varchar name
        varchar email
        varchar tax_id
        varchar payment_terms
        varchar currency
        numeric credit_limit
        numeric outstanding_balance
        varchar risk_score
    }

    ar_invoices {
        uuid id PK
        uuid tenant_id FK
        uuid customer_id FK
        varchar invoice_number
        date invoice_date
        date due_date
        numeric subtotal
        numeric tax_amount
        numeric total
        numeric amount_paid
        numeric amount_due
        varchar currency
        varchar status
        uuid gl_journal_id FK
    }

    ar_invoice_lines {
        uuid id PK
        uuid invoice_id FK
        varchar description
        numeric quantity
        numeric unit_price
        numeric amount
        uuid gl_account_id FK
        uuid revenue_schedule_id FK
    }

    ar_payments_received {
        uuid id PK
        uuid invoice_id FK
        numeric amount
        date payment_date
        varchar payment_method
        varchar reference
        uuid gl_journal_id FK
        float ai_match_confidence
    }

    ar_dunning_records {
        uuid id PK
        uuid invoice_id FK
        int dunning_level
        date sent_date
        varchar template_used
        varchar channel
        varchar status
    }

    ar_credit_notes {
        uuid id PK
        uuid customer_id FK
        uuid invoice_id FK
        varchar credit_number
        numeric amount
        varchar reason
        varchar status
        uuid gl_journal_id FK
    }

    ar_credit_limits {
        uuid id PK
        uuid customer_id FK
        numeric limit_amount
        numeric utilized_amount
        varchar currency
        date review_date
        uuid approved_by
    }

    ar_revenue_contracts {
        uuid id PK
        uuid customer_id FK
        varchar contract_number
        date start_date
        date end_date
        numeric total_value
        varchar recognition_method
    }

    ar_performance_obligations {
        uuid id PK
        uuid contract_id FK
        varchar description
        numeric standalone_price
        numeric allocated_price
        varchar satisfaction_type
        numeric progress_pct
    }

    %% ─── BILLING ─────────────────────────────────────
    billing_plans ||--o{ billing_subscriptions : "subscribes"
    billing_subscriptions ||--o{ billing_usage_records : "meters"
    billing_subscriptions ||--o{ billing_invoices : "invoiced"
    billing_invoices ||--o{ billing_invoice_items : contains
    billing_credits ||--o{ billing_credit_applications : "applied"
    billing_promo_codes ||--o{ billing_promo_redemptions : "redeemed"

    billing_plans {
        uuid id PK
        varchar name
        text description
        bigint price
        varchar currency
        varchar billing_period
        jsonb features
        jsonb limits
        varchar status
    }

    billing_subscriptions {
        uuid id PK
        uuid tenant_id FK
        uuid plan_id FK
        varchar status
        timestamp period_start
        timestamp period_end
        boolean cancel_at_period_end
        timestamp trial_end
        timestamp created_at
    }

    billing_usage_records {
        uuid id PK
        uuid subscription_id FK
        varchar metric
        bigint quantity
        timestamp recorded_at
        jsonb metadata
        varchar idempotency_key
    }

    billing_invoices {
        uuid id PK
        uuid subscription_id FK
        varchar invoice_number
        date period_start
        date period_end
        bigint subtotal
        bigint tax
        bigint total
        varchar currency
        varchar status
        date due_date
        timestamp paid_at
    }

    billing_invoice_items {
        uuid id PK
        uuid invoice_id FK
        text description
        bigint quantity
        bigint unit_price
        bigint amount
        varchar item_type
    }

    billing_credits {
        uuid id PK
        uuid tenant_id FK
        varchar credit_type
        text description
        numeric original_amount
        numeric remaining_amount
        numeric used_amount
        timestamp expires_at
    }

    billing_credit_applications {
        uuid id PK
        uuid credit_id FK
        uuid invoice_id FK
        numeric amount
        timestamp applied_at
    }

    billing_promo_codes {
        uuid id PK
        varchar code
        text description
        varchar discount_type
        numeric discount_value
        int max_redemptions
        int redemptions
        timestamp expires_at
    }

    billing_promo_redemptions {
        uuid id PK
        uuid promo_id FK
        uuid tenant_id FK
        timestamp redeemed_at
    }

    %% ─── PAYMENTS ────────────────────────────────────
    pay_transactions ||--o{ pay_refunds : "refunded"
    pay_customers ||--o{ pay_payment_methods : "methods"
    pay_customers ||--o{ pay_wallets : "wallets"
    pay_wallets ||--o{ pay_wallet_transactions : "movements"

    pay_transactions {
        uuid id PK
        varchar reference
        numeric amount
        varchar currency
        varchar status
        varchar transaction_type
        uuid customer_id FK
        varchar customer_email
        varchar payment_method
        varchar provider
        varchar provider_reference
        jsonb metadata
        timestamp completed_at
    }

    pay_customers {
        uuid id PK
        uuid tenant_id FK
        varchar email
        varchar name
        varchar external_id
    }

    pay_payment_methods {
        uuid id PK
        uuid customer_id FK
        varchar method_type
        varchar provider
        varchar token
        varchar last_four
        varchar brand
        boolean is_default
        smallint exp_month
        smallint exp_year
    }

    pay_wallets {
        uuid id PK
        uuid customer_id FK
        numeric balance
        varchar currency
        varchar status
    }

    pay_wallet_transactions {
        uuid id PK
        uuid wallet_id FK
        varchar type
        numeric amount
        numeric balance_after
        varchar reference
        timestamp created_at
    }

    pay_refunds {
        uuid id PK
        uuid transaction_id FK
        numeric amount
        varchar reason
        varchar status
        varchar provider_reference
    }

    pay_fraud_scores {
        uuid id PK
        uuid transaction_id FK
        float score
        jsonb risk_factors
        varchar decision
        timestamp evaluated_at
    }

    pay_provider_configs {
        uuid id PK
        uuid tenant_id FK
        varchar provider
        varchar api_key_encrypted
        jsonb settings
        boolean is_active
        int priority
    }

    %% ─── ASSET MANAGEMENT ────────────────────────────
    asset_assets ||--o{ asset_maintenance_records : "maintained"
    asset_assets ||--o{ asset_depreciation_records : "depreciated"
    asset_assets ||--o{ asset_lifecycle_events : "lifecycle"
    asset_assets ||--o{ asset_ai_analyses : "analyzed"
    asset_categories ||--o{ asset_assets : "categorized"
    asset_locations ||--o{ asset_assets : "located"

    asset_assets {
        int id PK
        uuid tenant_id FK
        varchar name
        varchar asset_tag
        text description
        varchar category
        varchar status
        varchar manufacturer
        varchar model_number
        varchar serial_number
        varchar location
        varchar department
        varchar assigned_to
        float purchase_price
        date purchase_date
        date warranty_expiry
        float salvage_value
        int useful_life_years
        varchar depreciation_method
        float operating_hours
        float max_operating_hours
        float condition_score
    }

    asset_maintenance_records {
        int id PK
        int asset_id FK
        varchar title
        text description
        varchar maintenance_type
        varchar priority
        varchar status
        date scheduled_date
        date completed_date
        float cost
        varchar technician
        text notes
        boolean is_recurring
        int recurrence_interval_days
        date next_due_date
    }

    asset_depreciation_records {
        int id PK
        int asset_id FK
        date period_start
        date period_end
        int period_number
        float opening_book_value
        float depreciation_amount
        float accumulated_depreciation
        float closing_book_value
        varchar method
    }

    asset_lifecycle_events {
        int id PK
        int asset_id FK
        varchar phase
        date event_date
        text description
        varchar performed_by
        float cost
        text metadata_json
    }

    asset_categories {
        uuid id PK
        varchar name
        varchar code
        uuid parent_id
    }

    asset_locations {
        uuid id PK
        uuid tenant_id FK
        varchar name
        varchar code
        text address
        uuid parent_id
    }

    asset_ai_analyses {
        uuid id PK
        int asset_id FK
        varchar analysis_type
        text summary
        jsonb recommendations
        varchar risk_level
        float confidence_score
        jsonb details
        timestamp analyzed_at
    }

    %% ─── TAX MANAGEMENT ─────────────────────────────
    tax_jurisdictions ||--o{ tax_rates : "rates"
    tax_jurisdictions ||--o{ tax_returns : "filings"
    tax_calculations ||--o{ tax_calculation_lines : "lines"

    tax_jurisdictions {
        uuid id PK
        varchar name
        varchar code
        varchar country
        varchar state_province
        varchar tax_type
        boolean is_active
    }

    tax_rates {
        uuid id PK
        uuid jurisdiction_id FK
        numeric rate
        date effective_from
        date effective_to
        varchar category
    }

    tax_returns {
        uuid id PK
        uuid tenant_id FK
        uuid jurisdiction_id FK
        varchar period
        numeric taxable_amount
        numeric tax_amount
        varchar status
        date filed_date
        date due_date
    }

    tax_calculations {
        uuid id PK
        uuid tenant_id FK
        uuid source_document_id
        varchar source_type
        numeric total_tax
        varchar provider
        timestamp calculated_at
    }

    tax_calculation_lines {
        uuid id PK
        uuid calculation_id FK
        uuid jurisdiction_id FK
        numeric taxable_amount
        numeric tax_rate
        numeric tax_amount
    }

    %% ─── EXPENSE MANAGEMENT ─────────────────────────
    exp_policies ||--o{ exp_policy_rules : "rules"
    exp_claims ||--o{ exp_claim_lines : "lines"
    exp_claims ||--o{ exp_approvals : "approved"
    exp_claims ||--o{ exp_receipts : "receipts"
    exp_corporate_cards ||--o{ exp_card_transactions : "transactions"

    exp_claims {
        uuid id PK
        uuid tenant_id FK
        uuid employee_id FK
        varchar claim_number
        varchar purpose
        varchar project_code
        date travel_start
        date travel_end
        numeric total_amount
        varchar currency
        varchar status
        uuid gl_journal_id FK
    }

    exp_claim_lines {
        uuid id PK
        uuid claim_id FK
        varchar category
        date expense_date
        numeric amount
        varchar currency
        text description
        uuid receipt_id FK
    }

    exp_receipts {
        uuid id PK
        uuid claim_line_id FK
        varchar storage_path
        varchar file_type
        text ocr_extracted_data
        float ocr_confidence
        timestamp uploaded_at
    }

    exp_approvals {
        uuid id PK
        uuid claim_id FK
        uuid approver_id FK
        varchar decision
        text comments
        timestamp decided_at
        int approval_level
    }

    exp_policies {
        uuid id PK
        uuid tenant_id FK
        varchar name
        varchar policy_type
        boolean is_active
    }

    exp_policy_rules {
        uuid id PK
        uuid policy_id FK
        varchar category
        varchar rule_type
        numeric max_amount
        varchar currency
        varchar applies_to
    }

    exp_corporate_cards {
        uuid id PK
        uuid tenant_id FK
        uuid employee_id FK
        varchar card_last_four
        varchar provider
        numeric spending_limit
        varchar status
    }

    exp_card_transactions {
        uuid id PK
        uuid card_id FK
        numeric amount
        varchar currency
        varchar merchant
        varchar category
        date transaction_date
        varchar reconciliation_status
        uuid claim_line_id FK
    }

    %% ─── TREASURY ────────────────────────────────────
    treas_bank_accounts ||--o{ treas_bank_statements : "statements"
    treas_bank_statements ||--o{ treas_statement_lines : "lines"
    treas_statement_lines ||--o{ treas_reconciliation_matches : "matched"
    treas_fx_contracts ||--o{ treas_fx_settlements : "settled"

    treas_bank_accounts {
        uuid id PK
        uuid tenant_id FK
        varchar bank_name
        varchar account_number
        varchar swift_code
        varchar currency
        numeric current_balance
        date last_reconciled
        varchar status
    }

    treas_bank_statements {
        uuid id PK
        uuid bank_account_id FK
        date statement_date
        numeric opening_balance
        numeric closing_balance
        varchar import_source
        varchar status
    }

    treas_statement_lines {
        uuid id PK
        uuid statement_id FK
        date transaction_date
        varchar description
        varchar reference
        numeric amount
        varchar type
        varchar reconciliation_status
    }

    treas_reconciliation_matches {
        uuid id PK
        uuid statement_line_id FK
        uuid gl_transaction_id FK
        float confidence_score
        varchar match_method
        varchar status
        timestamp matched_at
    }

    treas_cash_positions {
        uuid id PK
        uuid tenant_id FK
        date position_date
        varchar currency
        numeric balance
        numeric forecast_7d
        numeric forecast_30d
        numeric forecast_90d
    }

    treas_fx_contracts {
        uuid id PK
        uuid tenant_id FK
        varchar contract_type
        varchar buy_currency
        varchar sell_currency
        numeric buy_amount
        numeric sell_amount
        numeric rate
        date value_date
        varchar status
    }

    treas_fx_settlements {
        uuid id PK
        uuid contract_id FK
        numeric settled_amount
        date settlement_date
        uuid gl_journal_id FK
    }

    %% ─── BUDGET ──────────────────────────────────────
    budget_plans ||--o{ budget_line_items : "lines"
    budget_plans ||--o{ budget_scenarios : "scenarios"
    budget_line_items ||--o{ budget_actuals : "actuals"
    budget_plans ||--o{ budget_approvals : "approved"

    budget_plans {
        uuid id PK
        uuid tenant_id FK
        varchar name
        varchar budget_type
        uuid fiscal_year_id FK
        varchar period_type
        numeric total_amount
        varchar status
        uuid created_by
    }

    budget_line_items {
        uuid id PK
        uuid plan_id FK
        uuid gl_account_id FK
        varchar department
        varchar period
        numeric budgeted_amount
        numeric actual_amount
        numeric variance_amount
        numeric variance_pct
    }

    budget_scenarios {
        uuid id PK
        uuid plan_id FK
        varchar name
        varchar scenario_type
        jsonb assumptions
        numeric total_amount
    }

    budget_actuals {
        uuid id PK
        uuid line_item_id FK
        varchar period
        numeric amount
        timestamp recorded_at
    }

    budget_approvals {
        uuid id PK
        uuid plan_id FK
        uuid approver_id FK
        varchar decision
        text comments
        timestamp decided_at
    }

    %% ─── AUDIT ───────────────────────────────────────
    audit_logs {
        uuid id PK
        uuid tenant_id FK
        uuid user_id FK
        varchar action
        varchar resource_type
        uuid resource_id
        jsonb changes_before
        jsonb changes_after
        varchar ip_address
        varchar user_agent
        timestamp created_at
    }
```

## Table Count by Schema

| Schema | Tables | Description |
|--------|--------|-------------|
| gl (General Ledger) | 12 | COA, journals, periods, currencies, consolidation |
| ap (Accounts Payable) | 14 | Vendors, invoices, POs, matching, payments, approvals |
| ar (Accounts Receivable) | 11 | Customers, invoices, payments, dunning, revenue |
| billing | 10 | Plans, subscriptions, usage, invoices, credits, promos |
| payments (pay) | 9 | Transactions, wallets, methods, refunds, fraud |
| assets | 9 | Assets, maintenance, depreciation, lifecycle, AI |
| tax | 6 | Jurisdictions, rates, returns, calculations |
| expense (exp) | 9 | Claims, receipts, approvals, policies, cards |
| treasury (treas) | 8 | Banks, statements, reconciliation, FX, cash |
| budget | 5 | Plans, line items, scenarios, actuals, approvals |
| audit | 1 | Immutable audit log |
| **Total** | **94 core + 33 supporting = 127** | |

## Key Design Principles

1. **Immutable Posting Ledger**: `gl_journal_entries` with `posting_status` that only moves forward
2. **Double-Entry Enforcement**: Database constraint ensures sum(debits) = sum(credits) per journal entry
3. **Multi-Tenancy**: Every table includes `tenant_id` with RLS policies
4. **Referential Integrity**: All foreign keys enforced; no orphaned records
5. **Soft Deletes**: Financial records never physically deleted; use status flags
6. **Audit Trail**: `audit_logs` captures every mutation with before/after state
7. **UUID v7 Primary Keys**: Time-ordered for index efficiency
8. **Monetary Precision**: `NUMERIC(19,4)` for all financial amounts
