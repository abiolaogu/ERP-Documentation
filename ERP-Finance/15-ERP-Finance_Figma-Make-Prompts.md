# Figma & Make Prompts — ERP-Finance
> Version: 1.0 | Last Updated: 2026-02-23 | Status: Draft
> Classification: Internal | Author: AIDD System

---

## 1. Purpose

This document provides structured, copy-paste-ready prompts for generating production-grade UI designs in Figma Make and automation workflows in Make (formerly Integromat) for the ERP-Finance module. The platform provides comprehensive financial management: general ledger (GL), accounts payable (AP), accounts receivable (AR), billing and invoicing, budget management, expense management, asset management, tax management, treasury operations, and payment processing. All prompts reference the backend services (`general-ledger-service`, `accounts-payable-service`, `accounts-receivable-service`, `billing-service`, `budget-service`, `expense-management-service`, `asset-management-service`, `tax-management-service`, `treasury-service`, `payments-service`) and are designed for multi-tenant, entitlement-aware deployment via ERP-Platform with ERP-IAM authentication (OIDC/JWT). Design intelligence draws from SAP S/4HANA, Oracle NetSuite, QuickBooks, Xero, Sage Intacct, and Stripe.

---

## 2. AIDD Guardrails (Apply To All Prompts)

### 2.1 User Experience And Accessibility
- WCAG 2.1 AA compliance across all surfaces
- Minimum 44x44px touch targets on all interactive elements
- Clear visual states: default, hover, focus (2px ring), active, disabled, loading, error
- Plain-language labels with contextual tooltips for accounting terminology (COGS, EBITDA, accrual, amortization, reconciliation)
- Color-blind-safe palette; never rely on color alone (debits/credits use icons + text + color)
- Keyboard navigation and command palette (Cmd+K) parity for power users (accountants are keyboard-heavy users)
- Screen reader announcements for dynamic content (balance updates, approval notifications)
- Right-aligned numeric columns in all financial tables for readability
- Negative values displayed in parentheses and red: ($1,234.56)

### 2.2 Performance And Frontend Efficiency
- Route-level lazy loading for all page modules
- Initial route gzip bundle < 220KB
- Skeleton UIs for every data-dependent component (ledger tables, balance sheets, charts)
- Optimistic UI for safe mutations (journal entry drafts, expense categorization)
- Autosave for long forms (journal entries, invoice creation, budget planning)
- Virtual scrolling for large ledger tables (10,000+ rows)

### 2.3 Reliability, Trust, And Safety
- Recovery paths for all error states (retry button, support link, fallback view)
- Trust signals: "Books balanced", "Last reconciled: [date]", "Audit trail complete"
- Audit trail badges on ALL financial transactions: "Posted by [user] on [date]", "Approved by [user]"
- Two-step confirmation for all financial postings (post journal entry, approve payment, void invoice)
- Multi-step approval workflows for payments, expenses, and budget changes above threshold
- Tenant isolation indicator: organization name and fiscal year always visible in header
- Human-in-the-loop for AI recommendations: never auto-post financial entries
- Double-entry integrity: system prevents unbalanced journal entries
- Period locking: visual indicator when posting to a closed/locked period

### 2.4 Observability And Testability
- Analytics events on: page view, report generation, journal entry post, approval action, reconciliation match
- Error events on: API failure, posting failure, balance mismatch, period lock violation
- Performance instrumentation: FCP, LCP, CLS, TTI per route
- Financial integrity monitoring: automatic balance verification after each posting operation
- Feature flag awareness: components degrade gracefully when features are disabled

---

## 3. Figma Design Prompts

### 3.1 Design System Foundation

#### Prompt F-001: Core Design Tokens

```text
Create a comprehensive design system for an enterprise financial management platform called "ERP-Finance" that manages general ledger, accounts payable, accounts receivable, billing, budgets, expenses, assets, tax, treasury, and payments.

DESIGN TOKENS:

Color System:
- Primary: Deep Charcoal Blue (#1E293B) for navigation, headers, and authority — conveys precision and trustworthiness
- Secondary: Professional Green (#047857) for credits, income, positive variances, and primary actions
- Accent: Steel Blue (#3B82F6) for interactive elements, links, and secondary actions
- Debit: Rich Red (#DC2626) for debits, expenses, negative variances, and overspend
- Credit: Forest Green (#047857) for credits, revenue, positive variances, and under-budget
- Warning: Accounting Amber (#D97706) for pending approvals, approaching limits, and cautions
- Danger: Alert Red (#EF4444) for overdue payments, budget overruns, and destructive actions
- Info: Slate Blue (#6366F1) for informational badges, tooltips, and annotations
- Neutral: Slate scale (50-950) for text hierarchy, borders, backgrounds, surfaces
- Surface hierarchy: Base (#F8FAFC), Raised (#FFFFFF with shadow-sm), Overlay (#FFFFFF with shadow-lg)
- Account type colors: Assets (#3B82F6), Liabilities (#8B5CF6), Equity (#059669), Revenue (#10B981), Expenses (#EF4444)
- Budget status: Under Budget (#059669), On Budget (#3B82F6), Warning (#D97706), Over Budget (#EF4444)
- Approval status: Draft (#94A3B8), Pending (#D97706), Approved (#059669), Rejected (#EF4444), Posted (#3B82F6)
- Dark mode: Slate-950 base, Slate-900 raised, all accounting colors adjusted for dark contrast

Typography:
- Font family: Inter (UI and body text) / JetBrains Mono (ALL financial figures, account codes, amounts, ledger entries)
- Scale: 11px (caption), 12px (label/small), 13px (body-sm), 14px (body), 16px (subtitle), 20px (title), 24px (heading), 32px (display), 48px (hero metric)
- Weight tokens: regular (400), medium (500), semibold (600), bold (700)
- Line-height tokens: tight (1.2 for tables), normal (1.5), relaxed (1.75)
- CRITICAL: ALL numeric/monetary values MUST use JetBrains Mono with tabular figures for column alignment
- Negative amounts: parentheses format in red — ($1,234.56)
- Currency formatting: locale-aware with currency symbol, thousands separator, 2 decimal places

Spacing & Layout:
- 4px base grid; spacing scale: 4, 8, 12, 16, 20, 24, 32, 40, 48, 64, 80
- Breakpoints: mobile (390px), tablet (1024px), desktop (1440px), wide (1920px)
- Max content width: 1440px with 24px gutters
- Sidebar widths: collapsed (64px), expanded (256px)
- Table row height: 40px (compact financial tables), 48px (standard)
- Right-aligned numeric columns with consistent decimal alignment

Elevation & Motion:
- Shadow tokens: sm (cards), md (dropdowns), lg (modals/drawers), xl (command palette)
- Border-radius tokens: none (financial tables — sharp corners for precision), sm (4px for cards), md (8px), lg (12px for modals)
- Motion tokens: instant (0ms), fast (100ms), normal (200ms), slow (300ms)
- Minimal motion in financial context — precision over animation
- Subtle transitions only: drawer open/close, tab switch, tooltip reveal

Dark Mode:
- Background: #020617 base, #0F172A raised, #1E293B overlay
- Text: #F8FAFC primary, #CBD5E1 secondary, #64748B tertiary
- Debit red: adjusted to #FCA5A5 for dark contrast
- Credit green: adjusted to #6EE7B7 for dark contrast
- Charts: lighter palette with increased stroke weights

Deliver as a structured token library with light/dark theme variants, auto-layout primitives, and financial color semantics.
```

#### Prompt F-002: Component Library

```text
Create a production-ready component library for ERP-Finance with these domain-specific components:

NAVIGATION:
- Sidebar with sections: Dashboard, General Ledger, Accounts Payable, Accounts Receivable, Billing, Budgets, Expenses, Asset Management, Tax, Treasury, Payments, Reports, Settings
- Each item: icon (Lucide icon set) + label + optional count badge (pending approvals, overdue items)
- Collapsed (64px) and expanded (256px) states
- Active state: left 3px accent bar + background highlight
- Top bar: org name, fiscal period indicator ("FY 2026 — Q1"), global search ("Search transactions, accounts, vendors..."), notification bell, user avatar with role badge (Controller, AP Clerk, AR Analyst, Auditor)
- Breadcrumb trail
- Command palette (Cmd+K) for searching accounts, transactions, vendors, invoices

DATA DISPLAY — FINANCIAL:
- Ledger table: date, journal entry #, account code, account name, description, debit (right-aligned, red), credit (right-aligned, green), balance (right-aligned, bold), posted-by. Features: sortable headers, sticky header + footer (totals row), row highlighting, expandable rows for sub-entries
- Trial balance table: account code, account name, debit balance, credit balance, with totals row showing debit/credit equality
- Balance sheet layout: hierarchical tree — Assets (Current/Non-Current) + Liabilities (Current/Non-Current) + Equity sections with subtotals and grand total
- Income statement layout: Revenue - COGS = Gross Profit - Operating Expenses = Operating Income +/- Other = Net Income
- KPI stat card: metric name, value (monospace), delta %, trend sparkline, period selector
- Budget vs. Actual bar: horizontal bar with budget line, actual bar (color-coded vs. budget), variance text
- Cash flow chart: waterfall chart showing inflows (green up) and outflows (red down) with running balance
- Aging schedule: horizontal stacked bar (Current, 1-30, 31-60, 61-90, 90+) with dollar amounts and percentages

FORMS & INPUT — FINANCIAL:
- Account selector: searchable dropdown with account code + name, grouped by account type (Assets, Liabilities, Equity, Revenue, Expenses)
- Currency input: right-aligned, with currency symbol, thousands separator auto-formatting, 2 decimal places, negative value support
- Journal entry form: multi-line entry with debit/credit columns, running total, balance indicator (balanced/unbalanced)
- Date input: date picker with fiscal period indicator and period-lock warning
- Vendor/customer selector: searchable with recent selections
- Approval workflow indicator: stepper showing current approval stage
- File attachment: drag-and-drop for receipts, invoices, supporting documents
- Tax rate selector: dropdown with tax codes and rates
- Multi-currency input: amount + currency selector + exchange rate display + converted amount

FEEDBACK & OVERLAY:
- Toast: success (green), error (red), warning (amber), info (blue)
- Modal dialog (sm, md, lg) for journal entry posting confirmation, payment approval, void confirmation
- Drawer (right-slide) for transaction detail, vendor profile, invoice preview
- Posting confirmation modal: "Post journal entry JE-2026-0847? This action creates a permanent audit record." with "Post" (primary) and "Cancel" buttons
- Void confirmation: destructive dialog with reason field and type-to-confirm
- Approval dialog: "Approve/Reject" with required comment for reject
- Period lock warning: banner when attempting to post to locked period
- Balance verification badge: "Debits = Credits" (green) or "UNBALANCED: Difference of $X.XX" (red, blocks posting)
- Skeleton loaders for all tables and charts
- Empty state with financial illustration and CTA

FINANCE-SPECIFIC:
- Chart of Accounts tree: hierarchical tree view with account code, name, type badge, normal balance indicator (Dr/Cr), balance
- Journal entry composer: multi-line form with auto-balancing, debit/credit columns, account selector per line, memo per line, totals row
- Invoice template: professional invoice layout with company logo, bill-to, ship-to, line items, subtotal, tax, total, payment terms, due date
- Payment batch: list of payments to process with vendor, amount, method, due date, select/deselect, batch total
- Bank reconciliation workspace: two-column layout — bank statement items (left) and book entries (right) with match/unmatch actions and auto-match suggestions
- Expense report: list of expenses with receipt thumbnails, category, amount, approval status, policy compliance indicator
- Depreciation schedule: table with asset, method, useful life, annual depreciation, accumulated, book value per year
- Budget planning grid: rows = GL accounts, columns = months, cells = editable amounts, row/column totals, variance column
- Cash position widget: available balances across bank accounts with total

ACCESSIBILITY:
- All interactive elements: visible focus rings (2px offset)
- Minimum contrast 4.5:1 for text, 3:1 for UI elements
- All icons paired with labels or aria-labels
- Financial data: right-aligned with decimal alignment for readability
- Screen reader support: announce debit/credit values, balance status, approval actions
- Keyboard navigation optimized for accountant workflows: Tab through table cells, Enter to edit, Escape to cancel

Deliver as a component library with variant properties, auto-layout, design token references, light and dark mode variants.
```

### 3.2 Desktop Pages (1440px)

#### Prompt F-003: Finance Command Dashboard (1440px)

```text
Design a financial management command dashboard for ERP-Finance at 1440px desktop width.

TOP BAR:
- Left: ERP-Finance logo, organization name "Apex Holdings Ltd"
- Center: fiscal period badge "FY 2026 — Q1 (Jan-Mar)" with period selector, global search "Search accounts, transactions, vendors, invoices..."
- Right: notification bell with "9" badge (5 pending approvals, 3 overdue invoices, 1 reconciliation alert), user avatar "CO" with name "Catherine Okonkwo" and role "Financial Controller"

LEFT SIDEBAR (expanded, 256px):
- Dashboard (active)
- General Ledger
- Accounts Payable (badge: "12 pending")
- Accounts Receivable (badge: "$45K overdue")
- Billing & Invoicing
- Budgets
- Expenses (badge: "8 pending")
- Asset Management
- Tax Management
- Treasury
- Payments (badge: "3 batch")
- Reports
- Settings

MAIN CONTENT:

Row 1 — Welcome & Period Context:
- "Good morning, Catherine. FY 2026 Q1 — 36 days remaining."
- Quick actions: "Post Journal Entry" (primary), "Create Invoice" (secondary), "Process Payment" (secondary), "Run Report" (secondary)
- Alert banner (if any): "Period Jan 2026 closes in 5 days. 3 unposted entries pending." (amber)

Row 2 — Financial Overview Cards (6 columns):
- Total Revenue (MTD): $1,247,500 (up 8% vs. prior month, green sparkline)
- Total Expenses (MTD): $892,300 (up 3%, trend sparkline)
- Net Income (MTD): $355,200 (up 18%, green)
- Cash Position: $2,145,000 (across all accounts, blue)
- AP Outstanding: $234,500 (12 invoices pending, amber)
- AR Outstanding: $187,300 ($45,200 overdue, red indicator on overdue)

Row 3 — Two Panels:
- Left (55%): "Revenue vs. Expenses" — dual-line chart, last 12 months
  - Revenue line (green)
  - Expenses line (red)
  - Net income shaded area between them
  - X-axis: months (Mar 2025 — Feb 2026)
  - Hover tooltip with exact values and margins
  - Budget overlay toggle (dashed line)

- Right (45%): "Cash Flow" — waterfall chart for current month
  - Opening Balance: $1,890,000 (blue)
  - + Revenue Received: $1,180,000 (green up)
  - + Other Income: $45,000 (green up)
  - - Vendor Payments: ($720,000) (red down)
  - - Payroll: ($180,000) (red down)
  - - Tax Payments: ($52,000) (red down)
  - - Other Expenses: ($18,000) (red down)
  - = Closing Balance: $2,145,000 (blue, bold)

Row 4 — Three Panels:
- Left (33%): "Budget vs. Actual" — summary card
  - Revenue: Budget $1,200K | Actual $1,248K | Variance +$48K (+4%, green)
  - Expenses: Budget $900K | Actual $892K | Variance -$8K (-1%, green — under budget)
  - Net Income: Budget $300K | Actual $356K | Variance +$56K (+19%, green)
  - "View Budget Detail" link

- Center (33%): "Accounts Payable Aging" — horizontal stacked bar
  - Current: $156,000 (green)
  - 1-30 days: $45,200 (light amber)
  - 31-60 days: $22,100 (amber)
  - 61-90 days: $8,200 (orange)
  - 90+ days: $3,000 (red)
  - Total: $234,500
  - "View AP Details" link

- Right (33%): "Accounts Receivable Aging" — horizontal stacked bar
  - Current: $112,300 (green)
  - 1-30 days: $29,800 (light amber)
  - 31-60 days: $25,200 (amber)
  - 61-90 days: $12,000 (orange)
  - 90+ days: $8,000 (red)
  - Total: $187,300
  - "View AR Details" link

Row 5 — Two Panels:
- Left (50%): "Pending Approvals" — action table
  - Columns: Type (icon), Reference, Vendor/Employee, Amount, Submitted By, Date, Actions
  - 5 items requiring approval:
    1. Invoice | INV-2026-0234 | TechSupply Ltd | $12,450.00 | Maria Santos | Feb 22 | [Approve] [Reject]
    2. Expense | EXP-2026-0089 | James Adeyemi (Travel) | $2,340.00 | James Adeyemi | Feb 21 | [Approve] [Reject]
    3. Payment | PAY-2026-0156 | OfficeMax Corp | $8,900.00 | AP Team | Feb 22 | [Approve] [Reject]
    4. Journal Entry | JE-2026-0847 | Accrual Adjustment | $15,000.00 | Catherine O. | Feb 23 | [Post] [Edit]
    5. Budget Transfer | BT-2026-0012 | Marketing > IT | $5,000.00 | David Okafor | Feb 22 | [Approve] [Reject]
  - "View All Pending" link

- Right (50%): "Recent Transactions" — ledger summary
  - Columns: Date, Entry #, Description, Debit, Credit, Posted By
  - 5 recent journal entries with monospace financial figures
  - "View General Ledger" link

Row 6 — Bottom Panel:
- "Key Financial Ratios" — metric cards (6 across)
  - Current Ratio: 2.4:1 (green, target > 2.0)
  - Quick Ratio: 1.8:1 (green)
  - Debt-to-Equity: 0.35 (green, target < 0.5)
  - Gross Margin: 42.8% (up 1.2%, green)
  - Operating Margin: 28.5% (up 2.1%)
  - DSO (Days Sales Outstanding): 34 days (down 2 days, green)

Include light and dark mode variants.
```

#### Prompt F-004: General Ledger & Journal Entries (1440px)

```text
Design a general ledger page with journal entry management for ERP-Finance at 1440px desktop width, powered by general-ledger-service.

PAGE HEADER:
- Title: "General Ledger" with fiscal period badge "FY 2026 — Q1"
- Actions: "New Journal Entry" (primary), "Trial Balance" (secondary), "Chart of Accounts" (secondary), "Export" (icon)
- Period selector: dropdown with all open periods and period-lock status indicator

TAB BAR:
- Journal Entries (active) | Chart of Accounts | Trial Balance | Account Activity

JOURNAL ENTRIES TAB:

FILTER BAR:
- Search: "Search by entry #, description, account, amount..."
- Status: All | Draft (3) | Pending Approval (4) | Posted (1,247) | Voided (2)
- Account: searchable dropdown (account code + name)
- Date range: picker
- Entry type: All | Standard | Adjusting | Closing | Reversing | Recurring
- Amount range: min-max
- Posted by: user selector
- "Clear Filters"

JOURNAL ENTRY TABLE:
- Columns: Checkbox, Entry # (e.g., JE-2026-0847), Date, Type (badge), Description, Total Amount (monospace, right-aligned), Status (badge), Posted By (avatar + name), Posted Date, Actions
- 8 sample entries:
  1. JE-2026-0847 | Feb 23 | Standard | "Office supplies purchase" | $2,450.00 | Draft (gray) | Catherine O. | — | [Edit] [Post]
  2. JE-2026-0846 | Feb 23 | Adjusting | "Prepaid insurance amortization" | $4,166.67 | Pending (amber) | Maria S. | — | [Approve] [Reject]
  3. JE-2026-0845 | Feb 22 | Standard | "Revenue recognition — Project Alpha" | $85,000.00 | Posted (green) | James A. | Feb 22 | [View] [Reverse]
  4-8: Additional varied entries

JOURNAL ENTRY COMPOSER (triggered by "New Journal Entry"):
Full-page form or large modal:

Header:
- Entry #: auto-generated "JE-2026-0848"
- Date: date picker with period-lock check
- Type: dropdown (Standard, Adjusting, Closing, Reversing, Recurring)
- Description: text input "Enter entry description..."
- Attachments: file upload area for supporting documents
- Recurring: toggle, if yes: frequency (Monthly, Quarterly, Annual), end date

Line Items Table:
- Columns: Line #, Account (searchable dropdown with code + name), Description, Debit ($), Credit ($), Department (optional), Project (optional), Remove (X)
- Sample entry (3 lines):
  - Line 1: 6100 — Office Supplies | "Q1 supplies order" | Dr: $2,450.00 | Cr: — |
  - Line 2: 2100 — Accounts Payable | "TechSupply Ltd" | Dr: — | Cr: $2,450.00 |
- "Add Line" button at bottom
- Totals row: Total Debits: $2,450.00 | Total Credits: $2,450.00
- Balance indicator: "BALANCED" (green badge with checkmark) or "UNBALANCED: $X.XX difference" (red badge, blocks posting)

Footer Actions:
- "Save as Draft" (secondary)
- "Submit for Approval" (secondary, if amount > approval threshold)
- "Post Entry" (primary green, only enabled when balanced)
- "Cancel" (text link)

Post Confirmation Modal:
- "Post Journal Entry JE-2026-0848?"
- Summary: Date, Description, Total, Account lines preview
- "This action creates a permanent audit record and cannot be deleted."
- "Post" (primary) | "Cancel"

CHART OF ACCOUNTS TAB:
- Hierarchical tree view:
  - 1000 — Assets (blue)
    - 1100 — Current Assets
      - 1110 — Cash & Cash Equivalents: $2,145,000
      - 1120 — Accounts Receivable: $187,300
      - 1130 — Inventory: $345,000
      - 1140 — Prepaid Expenses: $28,500
    - 1200 — Non-Current Assets
      - 1210 — Property & Equipment: $1,200,000
      - 1220 — Accumulated Depreciation: ($320,000)
      - 1230 — Intangible Assets: $75,000
  - 2000 — Liabilities (purple)
    - 2100 — Current Liabilities
      - 2110 — Accounts Payable: $234,500
      - 2120 — Accrued Liabilities: $89,000
      - 2130 — Short-term Debt: $50,000
  - 3000 — Equity (green)
  - 4000 — Revenue (teal)
  - 5000 — Cost of Goods Sold (orange)
  - 6000 — Operating Expenses (red)
  - 7000 — Other Income/Expense
- Each account row: code, name, type badge, normal balance (Dr/Cr), current balance, status (active/inactive), "View Activity" link
- Actions: "Add Account", "Edit", "Deactivate"

TRIAL BALANCE TAB:
- Two-column layout: Account Code + Name | Debit Balance | Credit Balance
- Grouped by account type with subtotals
- Grand total row at bottom: Total Debits = Total Credits (with verification badge)
- Date selector: as-of date
- Filter: account type, non-zero balances only

Include light and dark mode variants.
```

#### Prompt F-005: Accounts Payable Management (1440px)

```text
Design an accounts payable management page for ERP-Finance at 1440px desktop width, powered by accounts-payable-service.

PAGE HEADER:
- Title: "Accounts Payable" with subtitle "12 pending invoices | $234,500 outstanding"
- Actions: "Enter Bill" (primary), "Process Payments" (secondary), "Vendor Directory" (secondary), "AP Aging Report" (icon)

TAB BAR:
- Bills & Invoices (active) | Payments | Vendors | Aging Report

BILLS & INVOICES TAB:

FILTER BAR:
- Status: All | Open (12) | Partially Paid (3) | Paid (187) | Overdue (4) | Voided (1)
- Vendor: searchable dropdown
- Due date range: picker
- Amount range: slider
- Payment terms: Net 30, Net 60, Net 90, COD
- "Clear Filters"

INVOICE TABLE:
- Columns: Checkbox, Invoice # (vendor ref), Vendor (avatar + name), Invoice Date, Due Date, Amount ($, monospace right-aligned), Paid ($), Balance ($), Status (badge), Payment Terms, Actions
- 8 sample invoices:
  1. INV-TS-2026-089 | TechSupply Ltd | Feb 15 | Mar 17 | $12,450.00 | $0.00 | $12,450.00 | Open (blue) | Net 30 | [Pay] [View]
  2. INV-OX-2026-045 | OfficeMax Corp | Feb 10 | Mar 12 | $8,900.00 | $0.00 | $8,900.00 | Open | Net 30 | [Pay] [View]
  3. INV-PW-2026-012 | PowerGrid Utilities | Feb 1 | Feb 28 | $3,200.00 | $0.00 | $3,200.00 | Overdue (red, 3 days) | Net 30 | [Pay] [View]
  4. INV-CL-2026-078 | CloudHost Inc | Jan 25 | Feb 24 | $15,600.00 | $7,800.00 | $7,800.00 | Partial (amber) | Net 30 | [Pay] [View]
  5-8: Additional varied invoices

- Bulk actions: "Pay Selected", "Approve Selected", "Export", "Print"

BILL ENTRY FORM (modal or page):
- Vendor: searchable selector (create new if not found)
- Invoice #: vendor's reference number
- Invoice Date: date picker
- Due Date: auto-calculated based on payment terms, editable
- Payment Terms: dropdown (Net 15, Net 30, Net 45, Net 60, Net 90, COD, Custom)
- Currency: selector with exchange rate display

Line Items:
- Columns: Line #, GL Account (searchable), Description, Quantity, Unit Price, Amount, Tax Code, Department
- Sample lines:
  - Line 1: 6100 — Office Supplies | "Printer paper and toner" | 10 | $45.00 | $450.00 | VAT 7.5% |
  - Line 2: 6200 — Computer Equipment | "Keyboard and mouse sets" | 5 | $400.00 | $2,000.00 | VAT 7.5% |
- Subtotal, Tax, Total row
- Attachment: upload invoice PDF/image
- Approval routing: auto-determined based on amount and GL account

PAYMENT PROCESSING PAGE:
- Select invoices to pay (checkbox table with running total)
- Payment date picker
- Payment method: Bank Transfer, Check, Wire, ACH, Card
- Bank account selector (from treasury-service)
- Payment batch summary: vendor count, invoice count, total amount
- "Create Payment Batch" button
- Approval required if batch > $10,000

VENDOR PROFILE DRAWER (450px, right slide):
- Vendor name, tax ID, contact info, payment terms
- Tabs: Profile | Invoices | Payments | 1099 Summary
- Key metrics: Total YTD spend, average payment timing, open balance
- Payment history: last 10 payments with dates, amounts, methods

Include light and dark mode variants.
```

#### Prompt F-006: Accounts Receivable & Billing (1440px)

```text
Design an accounts receivable and billing page for ERP-Finance at 1440px desktop width, powered by accounts-receivable-service and billing-service.

PAGE HEADER:
- Title: "Accounts Receivable & Billing" with subtitle "$187,300 outstanding | $45,200 overdue"
- Actions: "Create Invoice" (primary green), "Record Payment" (secondary), "Send Statements" (secondary), "AR Aging Report" (icon)

TAB BAR:
- Invoices (active) | Payments Received | Customers | Aging Report | Recurring

INVOICES TAB:

FILTER BAR:
- Status: All | Draft (4) | Sent (8) | Viewed (3) | Partial (2) | Paid (156) | Overdue (6) | Voided (1)
- Customer: searchable dropdown
- Date range, Amount range, "Clear Filters"

INVOICE TABLE:
- Columns: Checkbox, Invoice # (e.g., INV-2026-0234), Customer (avatar + name), Issue Date, Due Date, Amount ($), Paid ($), Balance ($), Status (badge), Days Overdue, Actions
- 8 sample invoices with diverse statuses
- Overdue rows with red left border
- "Send Reminder" bulk action for overdue invoices

INVOICE CREATION (full-page form):
Header Section:
- Company logo and address (auto-filled from settings)
- Invoice #: auto-generated
- Issue Date, Due Date (from payment terms)
- Customer selector: searchable (create new if needed)
- Bill-to address, Ship-to address (from customer record)
- Payment terms: Net 30 (default), selectable
- Currency: default with multi-currency option

Line Items Table:
- Columns: Item/Service, Description, Quantity, Rate, Amount, Tax Code
- Sample:
  - Line 1: "Consulting Services" | "Q1 project advisory" | 40 hrs | $150.00 | $6,000.00 | Standard
  - Line 2: "Software License" | "Annual subscription" | 1 | $12,000.00 | $12,000.00 | Standard
- "Add Line" button
- Subtotal: $18,000.00
- Tax (7.5%): $1,350.00
- Total: $19,350.00

Footer Section:
- Notes to customer: text area
- Terms & Conditions: text area (template selectable)
- Payment instructions: auto-filled based on org settings
- Attachments: upload supporting documents

Invoice Preview (split-screen right panel):
- Professional invoice template rendering as it would appear to customer
- "Download PDF" button

Actions:
- "Save as Draft" (secondary)
- "Send Invoice" (primary green) — opens send modal: email to, CC, subject, message template, attach PDF
- "Print" (icon)

PAYMENT RECORDING MODAL:
- Invoice selector (or accessed from specific invoice)
- Amount received: currency input (full amount or partial)
- Payment date: picker
- Payment method: Bank Transfer, Check, Wire, Card, Cash
- Reference/Check #: text input
- Deposit to account: GL account selector (bank account)
- If overpayment: option to apply to credit memo or refund
- "Record Payment" button

CUSTOMER PROFILE DRAWER (450px):
- Customer name, contact info, payment terms, credit limit
- Key metrics: Total AR, overdue amount, average days to pay, credit utilization
- Tabs: Profile | Invoices | Payments | Statements | Credit Memos
- "Send Statement" button for this customer

Include light and dark mode variants.
```

#### Prompt F-007: Budget Management & Variance Analysis (1440px)

```text
Design a budget management and variance analysis page for ERP-Finance at 1440px desktop width, powered by budget-service.

PAGE HEADER:
- Title: "Budget Management" with fiscal period "FY 2026"
- Actions: "Create Budget" (primary), "Import Budget" (secondary), "Budget Transfer" (secondary), "Export" (icon)
- Budget selector: dropdown — "Operating Budget FY 2026" (selected), "Capital Budget FY 2026", "Department Budgets FY 2026"

TAB BAR:
- Budget vs. Actual (active) | Budget Planning | Forecasting | Transfers | History

BUDGET VS. ACTUAL TAB:

Period Selector:
- Quick buttons: "YTD" (active), "Q1", "Q2", "Q3", "Q4", "Jan", "Feb", "Mar"...
- Comparison: "vs. Budget" (selected), "vs. Prior Year", "vs. Forecast"

Row 1 — Summary Cards (4 columns):
- Revenue: Budget $3,600K | Actual $1,248K (35% of annual, on track) | Variance +$48K (green)
- Expenses: Budget $2,700K | Actual $892K (33% of annual, on track) | Variance -$8K (green, under)
- Net Income: Budget $900K | Actual $356K (40% of annual, ahead) | Variance +$56K (green)
- Capital Expenditure: Budget $500K | Actual $125K (25% of annual) | Variance $0 (neutral)

Row 2 — Budget vs. Actual Chart:
- Full-width grouped bar chart by month (Jan-Dec 2026)
  - Budget bars (outline/light)
  - Actual bars (solid)
  - Color: green if actual <= budget (expense) or actual >= budget (revenue), red if adverse
  - Months Jan-Feb show actual data, Mar-Dec show budget only
  - YTD cumulative line overlay

Row 3 — Variance Analysis Table (the core of this page):
- Hierarchical tree table expandable by account category:
  - Revenue:
    - Product Sales: Budget $250K | Actual $268K | Variance +$18K | Var% +7.2% | (green)
    - Service Revenue: Budget $150K | Actual $145K | Variance -$5K | Var% -3.3% | (amber)
    - Other Income: Budget $20K | Actual $25K | Variance +$5K | Var% +25% | (green)
    - Total Revenue: Budget $420K | Actual $438K | Variance +$18K | +4.3% | (green, bold)
  - Cost of Goods Sold:
    - Materials: Budget $120K | Actual $115K | Variance -$5K | -4.2% | (green, under)
    - Labor: Budget $80K | Actual $85K | Variance +$5K | +6.3% | (red, over)
  - Operating Expenses (expandable):
    - Salaries & Wages: Budget $180K | Actual $178K | -$2K | -1.1% | (green)
    - Marketing: Budget $45K | Actual $52K | +$7K | +15.6% | (red)
    - IT & Technology: Budget $30K | Actual $28K | -$2K | -6.7% | (green)
    - Office & Admin: Budget $25K | Actual $27K | +$2K | +8.0% | (amber)
    - Travel & Entertainment: Budget $15K | Actual $18K | +$3K | +20% | (red)

- Column headers: Account, Budget ($), Actual ($), Variance ($), Variance (%), Status (traffic light icon)
- Sort by any column
- Click any row to drill down to transaction-level detail
- Favorable variance (under budget on expenses, over budget on revenue): green
- Unfavorable variance: red if > 10%, amber if 5-10%

Row 4 — Department Budget Comparison:
- Horizontal bar chart comparing budget utilization by department:
  - Finance: 32% utilized (green)
  - Marketing: 38% utilized (amber — slightly ahead)
  - Engineering: 30% utilized (green)
  - Sales: 35% utilized (green)
  - Operations: 33% utilized (green)
  - HR: 28% utilized (green)
- Target line at 33% (2 months of 6-month budget)

BUDGET PLANNING TAB (grid editor):
- Rows: GL accounts (expandable hierarchy)
- Columns: Jan | Feb | Mar | Apr | May | Jun | Jul | Aug | Sep | Oct | Nov | Dec | Total
- Cells: editable currency inputs
- Features: copy prior year, spread annual evenly, percentage increase from prior year, seasonal pattern application
- Approval workflow: Draft > Under Review > Approved > Active

Include light and dark mode variants.
```

#### Prompt F-008: Expense Management & Approvals (1440px)

```text
Design an expense management page for ERP-Finance at 1440px desktop width, powered by expense-management-service.

PAGE HEADER:
- Title: "Expense Management" with subtitle "8 pending approvals | $12,450 this month"
- Actions: "Submit Expense" (primary), "Create Expense Report" (secondary), "Policy Settings" (icon)
- Role context: Manager view (approver) or Employee view (submitter)

TAB BAR:
- Pending Approval (active) | My Expenses | Reports | Analytics | Policies

PENDING APPROVAL TAB (Manager View):

FILTER BAR:
- Submitter: multi-select with avatars
- Category: All | Travel | Meals | Office Supplies | Software | Professional Dev | Mileage | Other
- Amount range, Date range
- Policy compliance: All | Compliant | Flagged | Violation
- "Clear Filters"

EXPENSE TABLE:
- Columns: Checkbox, Expense ID, Submitter (avatar + name), Date, Category (badge), Description, Amount ($), Receipt (thumbnail), Policy Status (badge), Actions
- 8 sample expenses:
  1. EXP-0089 | James Adeyemi | Feb 22 | Travel (blue) | "Client meeting flight — Lagos to Abuja" | $450.00 | Receipt img | Compliant (green) | [Approve] [Reject] [View]
  2. EXP-0088 | Maria Santos | Feb 21 | Meals (amber) | "Team lunch — project celebration" | $280.00 | Receipt img | Compliant | [Approve] [Reject]
  3. EXP-0087 | David Okafor | Feb 21 | Travel (blue) | "Hotel stay — 3 nights" | $1,200.00 | Receipt img | Flagged (amber) — "Exceeds $300/night policy" | [Approve] [Reject] [Override]
  4. EXP-0086 | Lisa Park | Feb 20 | Software (purple) | "Annual SaaS subscription" | $2,400.00 | Invoice PDF | Compliant | [Approve] [Reject]
  5-8: Additional varied expenses

- Bulk actions: "Approve All Compliant", "Reject Selected", "Request Info"
- Policy violations highlighted with amber background and explanation tooltip

EXPENSE DETAIL DRAWER (450px, right slide):
- Expense header: ID, category, amount, date, status
- Submitter: avatar, name, department, budget remaining
- Description: full text
- Receipt viewer: large image/PDF preview with zoom
- Policy check results:
  - Per-diem limit: $300/night — Actual: $400/night — FLAGGED (amber)
  - Category budget: $5,000/quarter — Used: $3,200 — OK (green)
  - Pre-approval required above $1,000: Yes — Pre-approved: No — FLAGGED (amber)
- GL coding: auto-suggested account with option to override
- Department: auto-filled from submitter profile
- Previous similar expenses: 2 similar claims this quarter
- Approval actions: "Approve" (green), "Approve with Override" (amber), "Reject" (red, requires reason), "Request More Info" (blue)

EXPENSE REPORT VIEW (for grouped expense reports):
- Report header: report name, employee, date range, total
- Expense line items table with receipts, categories, amounts
- Per-diem summary if travel
- Mileage log if applicable
- GL distribution summary
- Approval chain: Submitted > Manager Review > Finance Review > Approved
- Current stage highlighted
- Action buttons per approval stage

MY EXPENSES TAB (Employee View):
- "Submit Expense" form:
  - Date, Category (dropdown), Description, Amount (currency input)
  - Receipt upload: camera capture or file upload (drag-and-drop)
  - GL Account: auto-suggested based on category
  - Project/Cost Center: optional selector
  - Notes: text area
  - Policy check: real-time validation showing compliance status as user fills form
  - "Submit" button

- My recent expenses: table with status badges (Draft, Submitted, Approved, Rejected, Reimbursed)
- Reimbursement status: "Last reimbursement: $1,240 on Feb 15, 2026"

Include light and dark mode variants.
```

### 3.3 Mobile Pages (390px)

#### Prompt F-009: Finance Mobile Dashboard (390px)

```text
Design a mobile finance dashboard for ERP-Finance at 390px width (iPhone 14 frame, 390x844px).

STATUS BAR: standard iOS status bar

TOP BAR:
- Left: hamburger menu
- Center: "ERP-Finance" logo
- Right: notification bell with "9" badge, user avatar "CO"

Fiscal Period Badge (below top bar):
- "FY 2026 — Q1 | 36 days remaining" (full-width bar, subtle background)

MAIN CONTENT (scrollable):

Financial Overview Cards (horizontal scroll, 2 visible + peek):
- Revenue MTD: $1,247,500 (up 8%, green)
- Expenses MTD: $892,300 (up 3%)
- Net Income: $355,200 (up 18%, green)
- Cash Position: $2,145,000
- AP Outstanding: $234,500
- AR Outstanding: $187,300

Quick Actions Grid (2x2):
- Post Entry (blue icon)
- Create Invoice (green icon)
- Submit Expense (amber icon)
- Approve (purple icon, badge "8")

Pending Approvals Card:
- "8 items need your approval"
- Swipeable cards:
  - "Invoice — TechSupply Ltd — $12,450" [Approve] [Reject]
  - "Expense — James A. — Travel $450" [Approve] [Reject]
  - "Payment Batch — $28,900" [Approve] [Reject]
- "View All" link

Cash Flow Mini Card:
- Mini waterfall chart: Opening > Inflows > Outflows > Closing
- "Opening: $1.89M → Closing: $2.15M"

AR/AP Summary:
- AR: "$187.3K outstanding | $45.2K overdue" (progress bar)
- AP: "$234.5K outstanding | $3.2K overdue" (progress bar)
- "View Details" links

Budget Alert Card (if variance detected):
- "Marketing budget at 38% (target: 33%). $7K over budget."
- "View Budget" link

BOTTOM NAVIGATION BAR (5 items):
- Dashboard (active)
- Ledger
- Invoices
- Expenses
- More

Include light and dark mode variants.
```

#### Prompt F-010: Mobile Expense Submission (390px)

```text
Design a mobile expense submission flow for ERP-Finance at 390px width (iPhone 14 frame).

SCREEN 1 — EXPENSE CAPTURE:

TOP BAR:
- Back arrow
- Title: "Submit Expense"
- "Save Draft" link

Receipt Capture Section (top 40%):
- Large camera button: "Take Photo of Receipt"
- Or "Upload from Gallery" link
- After capture: receipt image preview with crop/rotate options
- OCR processing indicator: "Extracting receipt data..." with spinner

Auto-Filled Form (from OCR, editable):
- Date: Feb 23, 2026 (auto-detected from receipt)
- Vendor: "OfficeMax Corp" (auto-detected)
- Amount: $245.00 (auto-detected, currency input)
- Category: "Office Supplies" (auto-suggested, dropdown to change)
- Description: "Printer toner and paper" (from receipt or manual)

Additional Fields:
- GL Account: auto-suggested "6100 — Office Supplies"
- Project/Cost Center: optional dropdown
- Notes: text area

Policy Check (inline):
- Green check: "Within office supplies policy ($500/month limit, $180 used)"
- Or amber warning: "Exceeds per-item limit of $200"

SUBMIT BUTTON (sticky bottom):
- "Submit Expense — $245.00" (primary, full width)

SCREEN 2 — CONFIRMATION:
- Green checkmark animation
- "Expense Submitted!"
- "EXP-2026-0090 | $245.00 | Office Supplies"
- "Routed to: Catherine Okonkwo (Manager)"
- "Expected approval: within 48 hours"
- Buttons: "Submit Another" (primary), "View My Expenses" (secondary)

SCREEN 3 — MY EXPENSES LIST:
- Filter tabs: All | Pending | Approved | Rejected | Reimbursed
- Expense cards:
  - Category icon + Category name | Date
  - Description (1 line, truncated)
  - Amount (large, bold) | Status badge
  - Tap to view detail

Include light and dark mode variants.
```

#### Prompt F-011: Mobile Invoice Viewer (390px)

```text
Design a mobile invoice viewing and approval screen for ERP-Finance at 390px width (iPhone 14 frame).

SCREEN 1 — INVOICE LIST:

TOP BAR:
- Back arrow
- Title: "Invoices"
- Filter icon

Tabs: All | To Pay (12) | Overdue (4) | Paid

Invoice Cards (scrollable list):
- Each card:
  - Vendor/Customer avatar + name
  - Invoice # (small, monospace)
  - Amount (large, bold, right-aligned)
  - Due date + days until due or "3 days overdue" (red)
  - Status badge: Open (blue), Overdue (red), Paid (green)
  - Tap to view detail

SCREEN 2 — INVOICE DETAIL:

TOP BAR:
- Back arrow
- Title: "INV-TS-2026-089"
- Share icon

Invoice Header Card:
- Vendor: "TechSupply Ltd"
- Invoice #: INV-TS-2026-089
- Date: Feb 15, 2026
- Due: Mar 17, 2026 (22 days remaining)
- Status: Open (blue badge)
- Amount: $12,450.00 (large, bold)

Line Items (expandable section):
- Line 1: "Printer Paper — Qty: 100 — $4.50 — $450.00"
- Line 2: "Toner Cartridges — Qty: 10 — $120.00 — $1,200.00"
- Line 3: "Computer Equipment — Qty: 5 — $2,160.00 — $10,800.00"
- Subtotal: $12,450.00
- Tax: $0.00
- Total: $12,450.00

Attached Document:
- Invoice PDF preview (thumbnail, tap to view full-screen)

GL Coding:
- Line 1: 6100 — Office Supplies
- Line 2: 6100 — Office Supplies
- Line 3: 6200 — Equipment

Approval Status:
- Submitted by: Maria Santos, Feb 16
- Pending approval: Catherine Okonkwo
- Approval threshold: > $5,000 requires controller sign-off

ACTION BUTTONS (sticky bottom):
- "Approve" (green, full width)
- Row below: "Reject" (red) | "Request Info" (blue)
- Reject opens: reason textarea + "Confirm Reject" button

Include light and dark mode variants.
```

#### Prompt F-012: Mobile Approval Queue (390px)

```text
Design a mobile approval queue for ERP-Finance at 390px width (iPhone 14 frame).

TOP BAR:
- Back arrow
- Title: "Approvals"
- Badge: "8 pending"

FILTER TABS (horizontal scroll):
- All (8) | Invoices (3) | Expenses (3) | Payments (1) | Journal Entries (1)

APPROVAL CARDS (scrollable list):

Card 1 — Invoice:
- Icon: invoice icon + "Invoice" label
- "TechSupply Ltd — INV-TS-2026-089"
- Amount: $12,450.00 (large)
- Due: Mar 17, 2026
- Submitted by: Maria Santos, Feb 16
- Swipe right: Approve (green)
- Swipe left: Reject (red)
- Tap: view detail

Card 2 — Expense:
- Icon: receipt icon + "Expense" label
- "James Adeyemi — Client meeting flight"
- Amount: $450.00
- Category: Travel
- Policy: Compliant (green badge)
- Swipe actions

Card 3 — Expense (flagged):
- Icon: receipt icon + "Expense" label + amber flag
- "David Okafor — Hotel stay (3 nights)"
- Amount: $1,200.00
- Category: Travel
- Policy: "Exceeds $300/night limit" (amber badge)
- Swipe or tap to review

Card 4 — Payment Batch:
- Icon: payment icon + "Payment Batch" label
- "5 vendor payments — Batch #PAY-2026-B012"
- Amount: $28,900.00
- "Review Required" (blue badge)
- Tap to view batch details

Card 5 — Journal Entry:
- Icon: ledger icon + "Journal Entry" label
- "Accrual Adjustment — JE-2026-0847"
- Amount: $15,000.00 (debit/credit)
- "Balanced" (green check)
- Tap to review

BATCH APPROVAL (bottom bar, appears when items selected):
- "Approve All Compliant (5)" button (green)
- Excludes flagged items from batch approval

DETAIL VIEW (tap a card):
- Full details rendered based on type (invoice, expense, payment, journal entry)
- Receipt/document viewer
- Policy compliance details
- GL coding
- "Approve" (green) | "Reject" (red, requires reason) | "Request Info" (blue)

Include light and dark mode variants.
```

### 3.4 Tablet/Responsive (1024px)

#### Prompt F-013: Finance Dashboard Tablet (1024px)

```text
Design the finance command dashboard adapted for 1024px tablet width.

LAYOUT ADAPTATIONS FROM 1440px DESKTOP:
- Sidebar: collapsed to 64px (icon-only with tooltips), expandable as overlay
- Top bar: fiscal period badge condensed, search becomes icon trigger
- Page padding: 16px

Row 1 — Welcome:
- Full-width card, quick actions in single row (4 buttons)

Row 2 — KPI Cards:
- 3x2 grid (3 cards per row, 2 rows) instead of 6 across

Row 3 — Charts:
- Stacked vertically: revenue vs. expenses chart full width, then cash flow waterfall full width

Row 4 — Three Panels:
- Budget vs. Actual: full width
- AP and AR Aging: side by side (50/50)

Row 5 — Tables:
- Stacked: pending approvals full width (with horizontal scroll for extra columns), then recent transactions

Row 6 — Ratios:
- 3x2 grid instead of 6 across

INTERACTIONS:
- All touch targets 44px minimum
- Tables support horizontal swipe
- Charts are tap-interactive
- Pull-to-refresh
- Approval actions: tap to view detail, swipe for quick approve/reject

Include both light and dark mode variants.
```

#### Prompt F-014: General Ledger Tablet (1024px)

```text
Design the general ledger page adapted for 1024px tablet width.

LAYOUT ADAPTATIONS:
- Sidebar collapsed to 64px
- Journal entry table: hide "Posted By" and "Posted Date" columns (available via horizontal scroll)
- Filter bar: collapse to "Filters" button with active count badge, opens dropdown
- Page padding: 16px

Journal Entry Composer (tablet):
- Full-width form instead of modal
- Line items table: account and description on one row, debit/credit on second row (stacked for space)
- Totals row always visible (sticky bottom)
- Balance indicator prominently displayed

Chart of Accounts (tablet):
- Tree view with slightly less indentation
- Balance column always visible
- Tap to expand/collapse nodes

Trial Balance (tablet):
- Full-width table
- Horizontal scroll for account details

Journal Entry Detail:
- Opens as bottom sheet (70% height) instead of side drawer
- Drag handle, swipe down to dismiss
- Full entry details with line items

INTERACTIONS:
- Touch-optimized: 44px row heights in tables
- Swipe actions on journal entries: approve (right/green), edit (left/blue)
- Tap account code to view account activity

Include both light and dark mode variants.
```

---

## 4. Make Automation Prompts

### Prompt M-001: Invoice Processing & AP Automation

```text
Create a Make (Integromat) scenario for ERP-Finance automated invoice processing and AP workflow:

Trigger: Webhook — receives POST from accounts-payable-service when a new vendor invoice is received
Payload: { invoiceId, vendorId, vendorName, invoiceNumber, invoiceDate, dueDate, amount, currency, lineItems: [{ glAccount, description, amount, taxCode }], attachmentUrl, tenantId }

Step 1: HTTP Module — GET /v1/accounts-payable/vendor/{vendorId} via accounts-payable-service
  - Fetch vendor profile: payment terms, tax status, 1099 requirement, default GL coding

Step 2: Data Transform — Validate invoice
  - Check for duplicate invoice number from same vendor
  - Verify amounts match line item totals
  - Validate GL account codes exist via general-ledger-service
  - Apply tax calculation via tax-management-service

Step 3: Router — Branch by amount and policy
  - Branch A: Amount <= $1,000 — auto-approve (if vendor verified and compliant)
  - Branch B: Amount $1,001-$10,000 — manager approval required
  - Branch C: Amount > $10,000 — controller approval required
  - Branch D: New vendor or flagged — manual review required

Branch A — Auto-Approve:
  Step 4A: HTTP Module — POST /v1/accounts-payable/invoice/{invoiceId}/approve
    - Auto-approve with system note: "Auto-approved per policy AP-001 (amount <= $1,000, verified vendor)"
  Step 5A: HTTP Module — POST /v1/general-ledger/journal-entry via general-ledger-service
    - Create journal entry: Dr: Expense account(s), Cr: Accounts Payable
    - Link to invoice record
  Step 6A: HTTP Module — POST /v1/payments/schedule via payments-service
    - Schedule payment based on vendor payment terms and optimization
    - If early payment discount available: flag for review

Branch B — Manager Approval:
  Step 4B: HTTP Module — PATCH /v1/accounts-payable/invoice/{invoiceId}
    - Set status to "Pending Manager Approval"
  Step 5B: Email — Notify department manager
    - "Invoice requiring approval: {vendorName} — {invoiceNumber} — ${amount}. Review and approve in ERP-Finance."
    - Include one-click approve/reject links (with auth token)

Branch C — Controller Approval:
  Step 4C: HTTP Module — PATCH /v1/accounts-payable/invoice/{invoiceId}
    - Set status to "Pending Controller Approval"
  Step 5C: Email — Notify financial controller
  Step 6C: Slack/Teams — Post to #finance-approvals channel

Branch D — Manual Review:
  Step 4D: Email — Notify AP team lead
    - "New vendor invoice requires manual review: {vendorName} — ${amount}"

Step 7: Google Sheets — Log invoice in "AP Invoice Tracker"
  - Row: Date, Vendor, Invoice #, Amount, GL Account, Status, Approval Route, Scheduled Payment Date

Error Handler: Log to error tracking, notify AP team
```

### Prompt M-002: Payment Run & Treasury Automation

```text
Create a Make (Integromat) scenario for ERP-Finance automated payment runs:

Trigger: Scheduled — runs every Tuesday and Thursday at 10:00 AM (payment run days)

Step 1: HTTP Module — GET /v1/accounts-payable/invoices?status=approved&dueDate=before_{7_days_from_now} via accounts-payable-service
  - Fetch all approved invoices due within 7 days

Step 2: HTTP Module — GET /v1/treasury/bank-accounts via treasury-service
  - Fetch available bank account balances

Step 3: Data Transform — Optimize payment batch
  - Group invoices by vendor (single payment per vendor if multiple invoices)
  - Check for early payment discounts (2/10 Net 30 etc.) — flag if discount available
  - Calculate total payment amount
  - Verify sufficient funds in disbursement account
  - If insufficient: prioritize by due date, then by vendor criticality

Step 4: Router — Branch by fund availability
  - Branch A: Sufficient funds — process full batch
  - Branch B: Insufficient funds — partial batch with prioritization alert

Branch A — Full Batch:
  Step 5A: HTTP Module — POST /v1/payments/batch via payments-service
    - Create payment batch:
      - Batch ID, total amount, vendor count, invoice count
      - Payment method per vendor (based on vendor preference)
      - Debit: AP accounts, Credit: Bank account
  Step 6A: HTTP Module — POST /v1/general-ledger/journal-entry via general-ledger-service
    - Post payment journal entry: Dr: Accounts Payable, Cr: Cash/Bank
  Step 7A: Email — Send payment batch summary to controller for final review
    - Include: batch total, vendor list, early discount savings captured

Branch B — Insufficient Funds:
  Step 5B: Email — Alert treasury manager
    - "Payment run alert: ${batchTotal} due but only ${available} available. ${shortfall} shortfall."
    - Priority list of payments that can be covered
  Step 6B: HTTP Module — POST /v1/payments/batch (partial)
    - Process priority payments only

Step 8: Iterator — For each payment processed
  Step 9: HTTP Module — POST /v1/accounts-payable/payment-notification
    - Update invoice status to "Paid"
  Step 10: Email — Send payment confirmation to vendor (optional, based on vendor preference)
    - "Payment of ${amount} for invoice(s) {invoiceNumbers} has been processed via {method}. Expected receipt: {date}."

Step 11: Aggregator — Payment run summary
  - Total paid, vendor count, invoices settled, discounts captured, next payment run date

Step 12: Email — Send payment run report to finance team
Step 13: Google Sheets — Update "Payment Run Log"

Error Handler: On payment failure, rollback batch status, alert finance team with failed payment details
```

### Prompt M-003: Revenue Recognition & AR Aging Automation

```text
Create a Make (Integromat) scenario for ERP-Finance revenue recognition and AR aging management:

Trigger: Scheduled — runs daily at 7:00 AM

PART 1 — Revenue Recognition:
Step 1: HTTP Module — GET /v1/billing/invoices?status=delivered&recognized=false via billing-service
  - Fetch invoices eligible for revenue recognition (delivered but not yet recognized)

Step 2: Iterator — For each eligible invoice
  Step 3: HTTP Module — GET /v1/billing/invoice/{invoiceId}/recognition-schedule
    - Fetch recognition schedule (point-in-time or over-time)
  Step 4: Router — Branch by recognition type
    - Branch A: Point-in-time (product delivered) — recognize full amount
    - Branch B: Over-time (service/subscription) — recognize proportional amount
  Step 5: HTTP Module — POST /v1/general-ledger/journal-entry via general-ledger-service
    - Post recognition entry: Dr: Deferred Revenue / Contract Liability, Cr: Revenue
    - Include reference to invoice and recognition policy

PART 2 — AR Aging Management:
Step 6: HTTP Module — GET /v1/accounts-receivable/aging via accounts-receivable-service
  - Fetch AR aging report: current, 1-30, 31-60, 61-90, 90+ buckets per customer

Step 7: Iterator — For each customer with overdue balance

Step 8: Router — Branch by aging bucket
  - Branch A: 1-15 days overdue — friendly reminder
  - Branch B: 16-30 days overdue — formal notice
  - Branch C: 31-60 days overdue — escalation
  - Branch D: 61-90 days overdue — final notice
  - Branch E: 90+ days overdue — collection action

Branch A — Friendly Reminder:
  Step 9A: Email — Send payment reminder
    - "This is a friendly reminder that invoice(s) {invoiceNumbers} totaling ${amount} are past due. Please arrange payment at your earliest convenience."

Branch B — Formal Notice:
  Step 9B: Email — Send formal overdue notice
    - "Your account has ${amount} overdue by {days} days. Please remit payment immediately to avoid late fees."
  Step 10B: HTTP Module — PATCH /v1/accounts-receivable/customer/{customerId}
    - Flag account as "Past Due"

Branch C — Escalation:
  Step 9C: Email — Send escalation letter
  Step 10C: Email — Notify AR manager and sales account owner
  Step 11C: HTTP Module — PATCH /v1/accounts-receivable/customer/{customerId}
    - Restrict new credit orders

Branch D — Final Notice:
  Step 9D: Email — Send final notice with legal warning
  Step 10D: Email — Notify finance director

Branch E — Collection:
  Step 9E: HTTP Module — PATCH /v1/accounts-receivable/customer/{customerId}
    - Flag for collection
    - Suspend account
  Step 10E: HTTP Module — POST /v1/general-ledger/journal-entry
    - Evaluate for doubtful debt provision

Step 12: Aggregator — Daily AR aging summary
Step 13: Email — AR aging report to finance team
Step 14: Google Sheets — Update "AR Aging Tracker"

Error Handler: Log failures, notify AR team
```

### Prompt M-004: Month-End Close Automation

```text
Create a Make (Integromat) scenario for ERP-Finance month-end close orchestration:

Trigger: Scheduled — runs on the 1st of each month at 6:00 AM (for closing the prior month)

Step 1: HTTP Module — GET /v1/general-ledger/period/{priorMonth}/status via general-ledger-service
  - Check if prior month period is open and ready for close
  - Verify no pending journal entries

Step 2: HTTP Module — GET /v1/general-ledger/close-checklist?period={priorMonth}
  - Fetch close checklist status:
    - All sub-ledgers posted (AP, AR, payroll)
    - Bank reconciliations complete
    - Intercompany entries posted
    - Accruals and prepaid amortizations posted
    - Depreciation entries posted
    - Revenue recognition complete
    - Tax accruals posted

Step 3: Router — Branch by checklist completeness
  - Branch A: All items complete — proceed with close
  - Branch B: Items incomplete — alert and list outstanding items

Branch A — Ready to Close:
  Step 4A: HTTP Module — POST /v1/asset-management/depreciation/run?period={priorMonth} via asset-management-service
    - Run monthly depreciation calculation and post entries
  Step 5A: HTTP Module — POST /v1/expense-management/accruals/run?period={priorMonth} via expense-management-service
    - Post prepaid expense amortization entries
  Step 6A: HTTP Module — POST /v1/tax-management/accruals/run?period={priorMonth} via tax-management-service
    - Calculate and post tax accruals
  Step 7A: HTTP Module — POST /v1/general-ledger/trial-balance?period={priorMonth}
    - Generate trial balance and verify debits = credits
  Step 8A: HTTP Module — POST /v1/general-ledger/period/{priorMonth}/close
    - Lock period (prevent new postings)
  Step 9A: HTTP Module — POST /v1/general-ledger/financial-statements?period={priorMonth}
    - Generate: Balance Sheet, Income Statement, Cash Flow Statement
  Step 10A: Email — Send close completion notification to finance team
    - "Month-end close for {month} complete. Financial statements attached."
    - Attach: PDF reports
  Step 11A: Slack/Teams — Post to #finance-close channel
    - "Month {month} closed successfully. TB balanced. Statements generated."

Branch B — Not Ready:
  Step 4B: Email — Send outstanding items alert to finance team
    - "Month-end close BLOCKED for {month}. Outstanding items:"
    - List each incomplete checklist item with responsible person and deadline
  Step 5B: Slack/Teams — Post urgent alert to #finance-close
  Step 6B: Iterator — For each incomplete item:
    - Email responsible person with reminder and deadline

Step 12: Google Sheets — Update "Month-End Close Tracker"
  - Row: Period, Start Time, End Time, Status, Checklist Score, Issues, Sign-off By

Error Handler: Critical — alert finance director and IT, pause close process
```

### Prompt M-005: Budget Variance Alert Automation

```text
Create a Make (Integromat) scenario for ERP-Finance real-time budget variance monitoring:

Trigger: Webhook — receives POST from general-ledger-service when a journal entry is posted
Payload: { journalEntryId, date, lineItems: [{ glAccount, debit, credit, department }], totalAmount, tenantId }

Step 1: Iterator — For each line item in the journal entry

Step 2: HTTP Module — GET /v1/budget/account/{glAccount}/status?period=current via budget-service
  - Fetch budget for this GL account in the current period:
    - Budget amount, actual (before this posting), remaining, utilization %

Step 3: Data Transform — Calculate new utilization
  - New actual = prior actual + posted amount
  - New utilization % = new actual / budget
  - Variance = budget - new actual

Step 4: Router — Branch by utilization threshold
  - Branch A: Utilization < 80% — no alert (normal)
  - Branch B: Utilization 80-99% — warning alert
  - Branch C: Utilization >= 100% — over-budget alert
  - Branch D: Utilization >= 120% — critical over-budget

Branch B — Warning (80-99%):
  Step 5B: Email — Notify department manager
    - "Budget Warning: {accountName} ({accountCode}) is at {utilization}% of budget. Budget: ${budget}, Actual: ${actual}, Remaining: ${remaining}."
  Step 6B: HTTP Module — POST /v1/budget/alert via budget-service
    - Log warning alert

Branch C — Over Budget (100-119%):
  Step 5C: Email — Notify department manager and financial controller
    - "OVER BUDGET: {accountName} ({accountCode}) has exceeded budget by ${overAmount} ({utilization}%)."
  Step 6C: Slack/Teams — Post to #finance-alerts
    - "Budget Overrun: {departmentName} — {accountName} at {utilization}% (${overAmount} over)"
  Step 7C: HTTP Module — POST /v1/budget/alert
    - Log over-budget alert with recommended actions

Branch D — Critical (>= 120%):
  Step 5D: Email — Urgent notification to department VP, controller, and CFO
    - "CRITICAL BUDGET OVERRUN: {accountName} at {utilization}%. Immediate review required."
  Step 6D: Slack/Teams — Post to #finance-alerts and #leadership
  Step 7D: HTTP Module — PATCH /v1/budget/account/{glAccount}/freeze via budget-service
    - Consider freezing further spending (pending approval)
  Step 8D: HTTP Module — POST /v1/budget/alert
    - Log critical alert with freeze recommendation

Step 9: Google Sheets — Update "Budget Variance Log"
  - Row: Date, Account, Department, Budget, Actual, Variance, Utilization %, Alert Level

Error Handler: Log calculation errors, do not block journal entry posting
```

### Prompt M-006: Financial Reporting & Distribution Automation

```text
Create a Make (Integromat) scenario for ERP-Finance automated financial report generation and distribution:

Trigger: Scheduled — runs on multiple schedules:
  - Daily at 7:00 AM: daily flash report
  - Weekly on Monday at 8:00 AM: weekly management report
  - Monthly on 5th at 9:00 AM: monthly financial package

Step 1: Data Transform — Determine report type based on trigger schedule

Step 2: Router — Branch by report type
  - Branch A: Daily flash report
  - Branch B: Weekly management report
  - Branch C: Monthly financial package

Branch A — Daily Flash:
  Step 3A: HTTP Module — GET /v1/treasury/cash-position via treasury-service
    - Fetch bank balances across all accounts
  Step 4A: HTTP Module — GET /v1/accounts-receivable/daily-collections via accounts-receivable-service
    - Fetch yesterday's collections
  Step 5A: HTTP Module — GET /v1/accounts-payable/daily-disbursements via accounts-payable-service
    - Fetch yesterday's payments
  Step 6A: Text Aggregator — Compose daily flash
    - Opening cash position, collections, disbursements, closing position
    - AR/AP highlights, any critical items
  Step 7A: Email — Send to finance team
    - Subject: "Daily Financial Flash — {date}"

Branch B — Weekly Management:
  Step 3B: HTTP Module — GET /v1/general-ledger/income-statement?period=wtd via general-ledger-service
  Step 4B: HTTP Module — GET /v1/budget/variance-summary?period=mtd via budget-service
  Step 5B: HTTP Module — GET /v1/accounts-receivable/aging via accounts-receivable-service
  Step 6B: HTTP Module — GET /v1/accounts-payable/aging via accounts-payable-service
  Step 7B: HTTP Module — GET /v1/treasury/cash-forecast?horizon=4weeks via treasury-service
  Step 8B: Google Docs — Generate formatted weekly report PDF
    - Income statement summary, budget variance highlights, AR/AP aging, cash forecast
  Step 9B: Email — Send to management team
    - Subject: "Weekly Financial Report — Week of {date}"
    - Attach PDF

Branch C — Monthly Financial Package:
  Step 3C: HTTP Module — GET /v1/general-ledger/balance-sheet?period={priorMonth} via general-ledger-service
  Step 4C: HTTP Module — GET /v1/general-ledger/income-statement?period={priorMonth}
  Step 5C: HTTP Module — GET /v1/general-ledger/cash-flow-statement?period={priorMonth}
  Step 6C: HTTP Module — GET /v1/budget/variance-detail?period={priorMonth} via budget-service
  Step 7C: HTTP Module — GET /v1/asset-management/depreciation-schedule via asset-management-service
  Step 8C: HTTP Module — GET /v1/tax-management/liability-summary?period={priorMonth} via tax-management-service
  Step 9C: Google Docs — Generate comprehensive monthly package PDF
    - Balance sheet, income statement, cash flow statement
    - Budget variance analysis with commentary
    - Key financial ratios and trends
    - Asset depreciation summary
    - Tax liability summary
    - Executive summary with CFO commentary template
  Step 10C: Email — Send to board, C-suite, and finance team
    - Subject: "Monthly Financial Package — {month} {year}"
    - Attach full PDF package
  Step 11C: Slack/Teams — Post summary to #finance-leadership
  Step 12C: Google Sheets — Archive monthly metrics for trend analysis

Step 13: Google Sheets — Log report generation in "Reporting Log"
  - Row: Date, Report Type, Generated At, Recipients, Status

Error Handler: On failure, notify finance admin, retry once
```

---

## 5. Prompt Usage Guidelines

### 5.1 Execution Order
1. Run **F-001** (Design Tokens) and **F-002** (Component Library) first to establish the foundation
2. Run desktop page prompts (**F-003** through **F-008**) for primary layouts
3. Run mobile prompts (**F-009** through **F-012**) for mobile-specific designs
4. Run tablet prompts (**F-013**, **F-014**) for responsive adaptations
5. Make automation prompts (**M-001** through **M-006**) can be executed independently

### 5.2 Customization
- Replace "Apex Holdings Ltd" with your organization name
- Adjust currency to match primary operating currency (USD, NGN, EUR, GBP, etc.)
- Modify chart of accounts structure to match your GL schema
- Adjust approval thresholds ($1,000, $10,000) to match your policies
- Modify fiscal year settings (calendar vs. non-calendar FY)
- Configure tax codes and rates for your jurisdiction

### 5.3 Accounting Standards Compliance
- All financial displays follow GAAP/IFRS presentation standards
- Negative amounts in parentheses: ($1,234.56)
- All monetary values right-aligned with decimal alignment
- Double-entry integrity enforced at UI level
- Period-locking prevents backdating of entries to closed periods

### 5.4 Design Quality Checks
- Every page must have: loading (skeleton), empty, populated, and error states
- Every interactive element must show: default, hover, focus, active, disabled states
- All forms must show: valid, invalid, and submitting states
- All financial amounts must use JetBrains Mono with tabular figures
- Dark mode must maintain debit/credit color distinction
- Approval flows must show current stage and remaining steps

---

## 6. Output Packaging Convention

| Artifact | Format | Naming Convention |
|----------|--------|-------------------|
| Design tokens | Figma Variables / JSON | `erp-finance-tokens-v{version}` |
| Component library | Figma Library | `ERP-Finance Components v{version}` |
| Desktop pages | Figma Pages | `Desktop — {PageName}` |
| Mobile pages | Figma Pages | `Mobile — {PageName}` |
| Tablet pages | Figma Pages | `Tablet — {PageName}` |
| Prototype flows | Figma Prototype | `Flow — {JourneyName}` |
| Make blueprints | Make JSON export | `make-finance-{scenario-name}-v{version}.json` |
| Design specs | Figma Dev Mode | Linked to each page |

---

## 7. Performance Acceptance Targets

| Metric | Target | Measurement |
|--------|--------|-------------|
| Lighthouse Performance | >= 95 | Per-route audit |
| Lighthouse Accessibility | >= 95 | Per-route audit |
| First Contentful Paint | < 1.2s | Lab + RUM |
| Largest Contentful Paint | < 2.5s | Lab + RUM |
| Cumulative Layout Shift | < 0.1 | Lab + RUM |
| Time to Interactive | < 3.0s | Lab measurement |
| Initial route bundle (gzip) | < 220KB | Build analysis |
| Ledger table render (10K rows) | < 1.0s | Virtual scroll benchmark |
| Report generation (p95) | < 5.0s | Backend + PDF rendering |
| Journal entry post (p95) | < 1.0s | Backend monitoring |
| API response (p95) | < 500ms | Backend monitoring |
| Approval action to confirmation | < 2.0s | End-to-end measurement |

---

## 8. AIDD Handoff Gate Template

| Gate | Check | Status |
|------|-------|--------|
| G-01 | All breakpoints designed (1440, 1024, 390) | Pending |
| G-02 | Light and dark mode variants complete | Pending |
| G-03 | All states covered (loading, empty, populated, error) | Pending |
| G-04 | WCAG 2.1 AA contrast ratios verified | Pending |
| G-05 | Touch targets >= 44px on all interactive elements | Pending |
| G-06 | Skeleton loaders designed for all data-dependent areas | Pending |
| G-07 | All monetary values use JetBrains Mono tabular figures | Pending |
| G-08 | Negative values displayed in parentheses with red color | Pending |
| G-09 | Double-entry balance verification UI implemented | Pending |
| G-10 | Period-lock warnings displayed on closed period access | Pending |
| G-11 | Approval workflows show current stage and approver | Pending |
| G-12 | Destructive actions (void, reverse) have type-to-confirm | Pending |
| G-13 | Audit trail badges on all financial transactions | Pending |
| G-14 | Tenant isolation and fiscal period visible in header | Pending |
| G-15 | Error recovery paths with retry and support link | Pending |
| G-16 | Design tokens reference Figma Variables (not hard-coded) | Pending |
| G-17 | Component auto-layout verified at all breakpoints | Pending |
| G-18 | Make automation scenarios tested with sample payloads | Pending |

---

## 9. Revision History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2026-02-23 | AIDD System | Initial Figma and Make design prompts for ERP-Finance covering GL, AP, AR, billing, budgets, expenses, asset management, tax, treasury, and payments services. Includes design system with accounting-specific tokens, 6 desktop pages (dashboard, general ledger, AP, AR/billing, budget/variance, expense management), 4 mobile screens (dashboard, expense submission, invoice viewer, approval queue), 2 tablet adaptations, and 6 Make automations (invoice processing, payment runs, revenue recognition/AR aging, month-end close, budget variance alerts, financial reporting). Design intelligence drawn from SAP S/4HANA, Oracle NetSuite, QuickBooks, Xero, Sage Intacct, and Stripe. |
