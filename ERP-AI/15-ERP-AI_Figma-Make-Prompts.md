# Figma & Make Prompts — ERP-AI
> Version: 1.0 | Last Updated: 2026-02-23 | Status: Draft
> Classification: Internal | Author: AIDD System

---

## 1. Purpose

This prompt pack provides production-ready Figma Make design prompts and Make automation scenario prompts for **ERP-AI**, the AI Intelligence Layer of the ERP platform. ERP-AI encompasses seven microservices — agent-catalog, agent-orchestrator, copilot-service, embedding-service, guardrail-service, ml-pipeline-service, and nlp-service — all accessible through a unified Gateway/API layer authenticated via ERP-IAM (OIDC/JWT) with tenant-scoped entitlements managed by ERP-Platform.

Designers use the Figma prompts to generate screens, components, and flows. Automation engineers use the Make prompts to build no-code integration scenarios for operational workflows.

---

## 2. AIDD Guardrails (Apply To All Prompts)

These are non-negotiable constraints that must be enforced in every generated output.

### 2.1 User Experience And Accessibility
- Meet WCAG 2.1 AA for color contrast, keyboard access, focus visibility, and semantic structure.
- Keep tap targets at least 44x44px on touch devices.
- Provide clear empty, loading, error, and success states for every asynchronous surface.
- Maintain plain-language labels and error messages; avoid jargon in user-facing copy.
- Support screen-reader announcements for agent status changes and pipeline progress updates.
- Inline AI recommendations must include confidence scores and rationale tooltips.

### 2.2 Performance And Frontend Efficiency
- Design for route-level lazy loading and code splitting by default.
- Keep initial route JS budget under 220KB gzip for user-facing routes.
- Avoid heavy always-mounted UI primitives; defer advanced widgets (pipeline DAG graph, embedding visualizer) until needed.
- Keep animation subtle (150-200ms) and non-blocking.
- Enforce skeleton UIs and progressive content rendering for catalog lists and pipeline logs.
- Chart and visualization components must load lazily with explicit placeholder dimensions.

### 2.3 Reliability, Trust, And Safety
- Build explicit recovery paths (retry, fallback, escalation) for copilot conversations, pipeline runs, and agent orchestrations.
- Guardrail violations must surface inline with clear severity badges and resolution guidance.
- Expose trust signals: model provenance, guardrail check status, confidence intervals on NLP outputs.
- Include audit trail visibility for all agent executions and pipeline runs.

### 2.4 Observability And Testability
- Every critical flow must define analytics events, error events, and performance checkpoints.
- Produce handoff notes that map UI states to test cases (unit, integration, E2E).
- Include performance instrumentation surfaces (LCP, INP, CLS, API latency markers).
- Agent orchestration flows must expose trace IDs for distributed tracing correlation.

---

## 3. Figma Design Prompts

### 3.1 Design System Foundation

#### Prompt F-001: Core Design Tokens

```
Create a comprehensive design token system for an enterprise AI intelligence platform (ERP-AI).

Token categories:

1) Color Roles:
   - Primary: Deep indigo (#312E81) for navigation and headers
   - Accent: Electric violet (#7C3AED) for AI-related actions and highlights
   - Surface: White (#FFFFFF) cards on light gray (#F9FAFB) background
   - Success: Emerald (#059669) for healthy agents, passed guardrails
   - Warning: Amber (#D97706) for degraded performance, guardrail warnings
   - Error: Rose (#DC2626) for failed pipelines, guardrail violations
   - Info: Sky blue (#0284C7) for informational badges, NLP confidence
   - AI Glow: Soft violet gradient (#7C3AED to #4F46E5) for copilot surfaces and AI indicators
   - Dark mode variants for every color role

2) Typography Scale:
   - Font family: Inter, system-ui, sans-serif
   - Display: 36px/2.25rem, weight 700, letter-spacing -0.02em (page titles)
   - H1: 30px/1.875rem, weight 600 (section headers)
   - H2: 24px/1.5rem, weight 600 (card titles)
   - H3: 20px/1.25rem, weight 500 (subsection headers)
   - Body: 16px/1rem, weight 400, line-height 1.5 (default text)
   - Body Small: 14px/0.875rem, weight 400 (table cells, metadata)
   - Caption: 12px/0.75rem, weight 500 (badges, labels, timestamps)
   - Code: JetBrains Mono, 14px/0.875rem (NLP output, JSON payloads, code blocks)

3) Spacing Scale:
   - 4px (xs) | 8px (sm) | 12px (md) | 16px (lg) | 24px (xl) | 32px (2xl) | 48px (3xl) | 64px (4xl)

4) Border Radius:
   - Subtle: 4px (inputs, inline badges)
   - Default: 8px (cards, modals)
   - Rounded: 12px (floating panels, copilot bubble)
   - Pill: 9999px (status badges, tags)

5) Elevation:
   - Level 1: 0 1px 3px rgba(0,0,0,0.08) (cards at rest)
   - Level 2: 0 4px 12px rgba(0,0,0,0.10) (hover cards, dropdowns)
   - Level 3: 0 8px 24px rgba(0,0,0,0.12) (modals, copilot panel)
   - Level 4: 0 16px 48px rgba(0,0,0,0.16) (command palette overlay)

6) Motion:
   - Fast: 100ms ease-out (button press, toggle)
   - Default: 150ms ease-in-out (card hover, panel slide)
   - Smooth: 250ms ease-in-out (modal open, page transition)
   - Slow: 400ms ease-in-out (pipeline DAG expand, embedding visualization)

Deliver both light and dark theme token sets. Include contrast verification annotations for all foreground/background pairs.
```

#### Prompt F-002: Component Library

```
Create an AI intelligence platform component library for ERP-AI with these component families:

Navigation Components:
- Global top bar: Logo "ERP-AI", breadcrumb trail, tenant switcher dropdown, notification bell with unread count, user avatar menu
- Left sidebar: Collapsible, icon + label items for Agent Catalog, Orchestrator, Copilot, Embeddings, Guardrails, ML Pipelines, NLP, Settings; active state with accent bar; collapse to icon-only mode
- Command palette: Centered overlay (Cmd+K trigger), search input, categorized results (agents, pipelines, embeddings, recent actions)

Agent Components:
- Agent card: Agent name, category badge (e.g., "Marketing", "Finance"), framework indicator (LangChain/CrewAI/AutoGen), status dot (healthy/degraded/offline), version tag, "Execute" primary button, "Configure" secondary button
- Agent detail header: Name, description (2-line truncate), version, author, last updated, health sparkline, capability tags
- Agent execution panel: Input schema form (auto-generated fields), output preview area, execution history mini-table

Pipeline Components:
- Pipeline DAG node: Step name, status icon (pending/running/success/failed), duration badge, connector lines with directional arrows
- Pipeline status timeline: Vertical timeline with step cards, progress indicator, log excerpt per step
- Pipeline metrics row: Total runs, success rate ring, avg duration, last run timestamp

Copilot Components:
- Chat bubble: User message (right-aligned, indigo), AI response (left-aligned, violet gradient border), timestamp, feedback thumbs
- Copilot floating panel: Expandable from bottom-right, header with minimize/expand/close, message thread, input bar with attach and send
- Suggestion chip row: Horizontal scrollable chips for follow-up prompts

Guardrail Components:
- Guardrail status badge: Pass (green check), Warn (amber triangle), Fail (red X), with severity label
- Guardrail detail card: Rule name, description, applied scope, last evaluation result, violation count trend sparkline
- Guardrail violation inline alert: Inline banner with severity icon, rule name, explanation, "View Details" link

Data Components:
- KPI tile: Metric name, large number, trend arrow with percentage, sparkline, time range label
- Data table: Sortable columns, row selection checkboxes, inline status badges, pagination footer, bulk action toolbar
- Code/JSON viewer: Syntax-highlighted block with copy button, line numbers, collapsible sections

State variants for every component:
- Default, hover, active, focus-visible, loading (skeleton), disabled, error, success, empty state, offline/degraded
```

### 3.2 Desktop Pages (1440px)

#### Prompt F-003: Agent Catalog Dashboard

```
Design the Agent Catalog dashboard page for ERP-AI at 1440px width.

Layout:
- Global top bar with breadcrumb "ERP-AI > Agent Catalog"
- Left sidebar with all 7 service areas highlighted, "Agent Catalog" active
- Main content area with 24px padding

Content sections:

1) Header row:
   - Page title "Agent Catalog" with subtitle "Browse, deploy, and manage AI agents across your organization"
   - Action bar: "Register New Agent" primary button, "Import from Template" secondary, search input with filter icon
   - View toggle: Grid view (default) | List view

2) Filter bar:
   - Category chips: All, Marketing, Sales, Finance, HR, Legal, Operations, Customer Service, Analytics, Custom
   - Framework filter: All, LangChain, CrewAI, AutoGen, Custom
   - Status filter: All, Healthy, Degraded, Offline
   - Sort: Most Used, Recently Updated, Alphabetical, Health Score

3) Agent grid (3 columns):
   - Each card shows: Agent icon/avatar, name ("Lead Scoring Agent"), category badge ("Sales"), framework tag ("LangChain"), health status dot, version "v2.4.1", description truncated to 2 lines, execution count "12,847 runs", success rate "99.2%", "Execute" primary CTA, "Details" secondary CTA
   - Sample agents: "Lead Scoring Agent", "Invoice Processor", "Customer Sentiment Analyzer", "Email Campaign Optimizer", "Compliance Checker", "Demand Forecaster", "Content Generator", "Meeting Summarizer", "Contract Reviewer"

4) Right sidebar panel (collapsible, 320px):
   - "Quick Execute" widget: Agent selector dropdown, simplified input form, "Run" button
   - "Recent Executions" feed: Last 5 runs with agent name, status badge, timestamp
   - "Health Overview" mini-widget: Pie chart showing healthy/degraded/offline distribution

5) Pagination footer: Page numbers, items per page selector, total count "Showing 1-24 of 156 agents"

States to include: Default loaded, empty catalog (onboarding CTA), search with no results, loading skeletons, API error with retry.

Include dark mode variant.
```

#### Prompt F-004: Agent Orchestrator — Workflow Builder

```
Design the Agent Orchestrator workflow builder page for ERP-AI at 1440px width.

Layout:
- Global top bar with breadcrumb "ERP-AI > Orchestrator > Workflow: Q4 Revenue Pipeline"
- Left sidebar with "Orchestrator" active
- Main content split: Canvas (70%) + Configuration panel (30%)

Canvas area:
- Light dot-grid background
- Workflow displayed as a directed acyclic graph (DAG)
- Sample workflow "Q4 Revenue Pipeline" with 5 steps:
  Step 1: "Fetch CRM Leads" (data source node, blue) ->
  Step 2: "Lead Scoring Agent" (agent node, violet) ->
  Step 3: "Guardrail Check: PII Compliance" (guardrail node, amber) ->
  Step 4: "Email Campaign Optimizer" (agent node, violet) ->
  Step 5: "Notify Sales Team" (action node, green)
- Each node card: Step number, icon, name, status indicator (idle/running/complete/failed), mini input preview
- Connection lines with directional arrows, animated pulse when running
- "+" insertion button on connection lines to add intermediate steps
- Parallel branch support: Fork node splitting into two parallel paths that rejoin
- Start node (green circle) and End node (checkered flag)

Top toolbar:
- Workflow name (inline editable): "Q4 Revenue Pipeline"
- Status badge: Draft | Saved | Running | Completed | Failed
- Actions: Save, Execute, Schedule, Clone, Delete, Version History dropdown
- Execution history: "Last run: 2026-02-22 14:30 UTC — Success (23.4s)"

Configuration panel (right, 30%):
- When node selected: Full configuration form
  - Agent selector (searchable dropdown)
  - Input configuration: Dynamic form fields based on agent's input schema
  - Data mapping: "Use output from Step N" toggle per field with JSONPath picker
  - Guardrail assignment: Multi-select guardrails to apply
  - Advanced: Retry count (0-5), Timeout (5s-300s), Failure strategy (Stop/Skip/Fallback)
- When nothing selected: Workflow summary stats, input/output schema overview

Bottom bar:
- Minimap thumbnail of full workflow
- Zoom controls: +, -, Fit to view
- Execution progress bar (when running)

States: Empty canvas (drag first step CTA), running workflow with animated progress, failed step with error detail popover, completed with summary metrics.

Include dark mode variant.
```

#### Prompt F-005: Copilot Chat Interface

```
Design the AI Copilot full-page interface for ERP-AI at 1440px width.

Layout:
- Global top bar with breadcrumb "ERP-AI > Copilot"
- Left sidebar with "Copilot" active
- Main area: Three-column layout — Conversation list (280px) | Chat thread (flexible) | Context panel (320px)

Conversation list (left, 280px):
- Search conversations input
- "New Conversation" button at top
- Conversation items: Title (auto-generated from first message), preview text, timestamp, unread indicator
- Grouped by: Today, Yesterday, Last 7 Days, Older
- Sample conversations: "Q4 Revenue Forecast Analysis", "Optimize Email Campaigns", "PII Compliance Check Results", "Demand Prediction Model Tuning"

Chat thread (center):
- Message bubbles:
  - User message: Right-aligned, indigo background, white text, timestamp
  - Copilot response: Left-aligned, white background with subtle violet left border, includes:
    - Markdown rendering (headers, lists, code blocks, tables)
    - Confidence badge: "High Confidence (94%)" in green, or "Medium (71%)" in amber
    - Source citations: Clickable links to originating agent/data source
    - Inline chart previews when relevant (mini bar chart, sparkline)
    - Action buttons: "Execute this as workflow", "Save to report", "Share"
  - Feedback row under each AI response: Thumbs up, thumbs down, copy, regenerate
- Typing indicator: Animated dots with "Copilot is thinking..." label
- Guardrail intervention: Orange inline banner "Guardrail 'PII-Filter' redacted sensitive content from this response" with "View original" (admin only)

Input area (bottom):
- Multi-line text input with placeholder "Ask Copilot about your agents, data, or workflows..."
- Attach button (upload files for context)
- Agent selector chip: Click to scope question to specific agent
- Send button (violet accent)
- Keyboard shortcut hint: "Enter to send, Shift+Enter for new line"

Context panel (right, 320px):
- "Active Context" header
- Referenced agents list with health status
- Referenced data sources
- Guardrails applied to this conversation
- Conversation metadata: Started, messages count, tokens used
- "Export Conversation" button (PDF, Markdown)

Suggestion chips row (above input): "Summarize today's pipeline runs", "Which agents have degraded health?", "Compare lead scoring models", "Generate compliance report"

States: Empty conversation (onboarding with example prompts), long conversation with scroll, copilot error with retry, rate limit warning.

Include dark mode variant.
```

#### Prompt F-006: ML Pipeline Monitor

```
Design the ML Pipeline monitoring and management page for ERP-AI at 1440px width.

Layout:
- Global top bar with breadcrumb "ERP-AI > ML Pipelines"
- Left sidebar with "ML Pipelines" active
- Main content area

Content sections:

1) Header:
   - Title "ML Pipelines" with subtitle "Train, deploy, and monitor machine learning models"
   - Actions: "Create Pipeline" primary button, "Import Template" secondary
   - View toggle: Card View | Table View | Timeline View

2) Pipeline overview metrics (4 KPI tiles in a row):
   - Active Pipelines: 12 (trend +2 this week)
   - Total Runs Today: 847 (trend -3% vs yesterday)
   - Average Duration: 4m 23s (trend -12s improvement)
   - Success Rate: 98.7% (trend +0.2%)

3) Active pipelines list (card view, 2 columns):
   Each pipeline card:
   - Pipeline name: "Customer Churn Prediction Model v3"
   - Status badge: Running | Scheduled | Idle | Failed
   - Progress bar (if running): "Step 3/5: Feature Engineering — 67%"
   - Last run: "2026-02-23 09:15 UTC — Success (6m 12s)"
   - Next scheduled: "2026-02-24 00:00 UTC"
   - Model metrics: Accuracy 94.2%, F1 0.91, AUC 0.96
   - Actions: View, Run Now, Edit, Pause Schedule, Logs

   Sample pipelines:
   - "Customer Churn Prediction v3" — Running, Step 3/5
   - "Demand Forecasting - Weekly" — Scheduled, next run in 4h
   - "Lead Score Model Retrain" — Idle, last run 2h ago
   - "Sentiment Analysis Fine-tune" — Failed, retry available
   - "Invoice Classification Pipeline" — Running, Step 1/3
   - "Fraud Detection Model Update" — Idle, healthy

4) Pipeline detail expandable (when card clicked):
   - DAG visualization of pipeline stages (data ingest -> preprocess -> train -> evaluate -> deploy)
   - Step-by-step log viewer with timestamps and log levels
   - Model performance comparison chart: Current vs Previous version (bar chart)
   - Resource utilization: CPU, memory, GPU time
   - Artifact registry: Model files, evaluation reports, feature importance charts

5) Recent run history table:
   Columns: Pipeline Name, Run ID, Trigger (Manual/Scheduled/Event), Start Time, Duration, Status, Actions
   10 rows with pagination

States: No pipelines (onboarding CTA), all pipelines healthy, pipeline failure detail with stack trace, running pipeline with live progress.

Include dark mode variant.
```

#### Prompt F-007: Guardrail Management Console

```
Design the Guardrail management console for ERP-AI at 1440px width.

Layout:
- Global top bar with breadcrumb "ERP-AI > Guardrails"
- Left sidebar with "Guardrails" active
- Main content area

Content sections:

1) Header:
   - Title "Guardrails" with subtitle "Define, monitor, and enforce AI safety and compliance rules"
   - Actions: "Create Guardrail" primary button, "Import Rule Set" secondary
   - Filter: Category (Content Safety, PII Protection, Compliance, Quality, Custom), Status (Active, Inactive, Draft)

2) Guardrail health dashboard (top):
   - 4 metric tiles:
     - Total Active Rules: 47
     - Evaluations Today: 23,841
     - Violation Rate: 1.3% (trend -0.2% this week)
     - Mean Evaluation Latency: 12ms
   - Violations trend chart: Line chart over last 7 days, split by severity (Critical, High, Medium, Low)

3) Guardrail rules list (table view):
   Columns: Rule Name, Category, Severity, Applied To (agents/pipelines), Evaluations (24h), Violations (24h), Status, Actions
   Sample rows:
   - "PII Detection & Redaction" | PII Protection | Critical | 34 agents | 8,241 | 12 | Active
   - "Hate Speech Filter" | Content Safety | Critical | All agents | 15,603 | 3 | Active
   - "GDPR Data Residency" | Compliance | High | EU-scoped agents | 4,102 | 0 | Active
   - "Output Length Limit (4096 tokens)" | Quality | Medium | Copilot | 6,891 | 247 | Active
   - "Hallucination Confidence Threshold" | Quality | High | 12 agents | 3,004 | 89 | Active
   - "Financial Advice Disclaimer" | Compliance | High | Finance agents | 1,200 | 0 | Active

4) Rule detail panel (slide-over from right, 480px):
   - Rule name, description, category badge, severity badge
   - Configuration: Threshold settings, regex patterns, blocked terms, confidence floor
   - Scope: Which agents/pipelines this rule applies to (multi-select with search)
   - Violation examples: Recent violations with context, severity, resolution
   - Performance: Evaluation latency histogram, false positive rate
   - Audit log: Who modified this rule, when, what changed

5) Violation investigation view:
   - Filterable by rule, agent, time range, severity
   - Each violation card: Timestamp, agent, input excerpt (redacted), rule triggered, severity, resolution status
   - Bulk actions: Mark as reviewed, false positive, escalate

States: No guardrails defined (setup wizard CTA), all rules healthy, critical violation alert banner, rule creation wizard (3-step form).

Include dark mode variant.
```

#### Prompt F-008: NLP & Embeddings Analytics

```
Design a combined NLP and Embeddings analytics page for ERP-AI at 1440px width.

Layout:
- Global top bar with breadcrumb "ERP-AI > NLP & Embeddings"
- Left sidebar with "NLP" and "Embeddings" items, "NLP" active with submenu showing "Analytics"
- Main content area with tab bar: NLP Analytics | Embedding Explorer | Model Registry

Tab 1 — NLP Analytics:

1) Filter bar: Date range picker, model selector, language filter, task type (Classification, Extraction, Summarization, Translation, Sentiment)

2) Metrics row (5 tiles):
   - Requests Today: 45,231
   - Avg Latency: 89ms
   - Accuracy (sampled): 96.1%
   - Languages Processed: 14
   - Token Throughput: 2.1M tokens/hr

3) Charts (2x2 grid):
   - Line chart: "NLP Request Volume" over 30 days, split by task type
   - Horizontal bar chart: "Top 10 Models by Usage" with model names and request counts
   - Stacked area chart: "Language Distribution" over time (English, Spanish, French, German, Portuguese, Other)
   - Heatmap: "Latency by Model and Task Type" (rows = models, columns = task types, color = p95 latency)

4) Recent NLP requests table:
   Columns: Timestamp, Request ID, Model, Task Type, Language, Input Preview, Latency, Status
   Sample: "2026-02-23 10:42:01 | req_7a3f | gpt-4o | Sentiment | EN | 'The quarterly results exceeded...' | 67ms | Success"

Tab 2 — Embedding Explorer:

1) Embedding spaces overview:
   - Cards for each active embedding space: "Product Descriptions (1.2M vectors)", "Customer Support Tickets (340K vectors)", "Knowledge Base Articles (89K vectors)"
   - Each card: Vector count, dimension size (1536), index status, last updated, similarity search latency

2) Visual embedding explorer:
   - 2D/3D scatter plot (UMAP/t-SNE projection) of sampled vectors
   - Color-coded by category
   - Click a point to see the original document and nearest neighbors
   - Search input: "Find similar to..." with natural language input

3) Similarity search interface:
   - Text input area: Paste or type content
   - Results: Top-K similar items with similarity score, preview, source link

Tab 3 — Model Registry:
   - Table: Model Name, Provider, Version, Task Types, Status (Active/Deprecated), Avg Latency, Total Requests
   - Actions: Deploy, Deprecate, A/B Test, View Metrics

States: Empty state for new tenant (onboarding with sample data), high-traffic dashboard, model degradation alert, embedding index rebuilding progress.

Include dark mode variant.
```

### 3.3 Mobile Pages (390px)

#### Prompt F-009: Mobile Agent Catalog

```
Design the Agent Catalog page for ERP-AI at 390px mobile width.

Layout:
- Sticky top bar: Hamburger menu (left), "Agent Catalog" title (center), search icon (right)
- Bottom navigation bar: Catalog (active), Orchestrator, Copilot, Pipelines, More

Content:
1) Search bar (full width, revealed on search icon tap): Input with placeholder "Search agents...", filter icon
2) Horizontal scrollable filter chips: All, Marketing, Sales, Finance, HR, Operations
3) Agent cards (single column, full width):
   - Each card: Agent name, category badge, status dot, description (1-line truncate), success rate, "Execute" button (full width)
   - Tap card to expand inline with full details
4) Pull-to-refresh gesture support
5) Infinite scroll with loading indicator at bottom

Bottom sheet (triggered by filter icon):
- Framework filter, status filter, sort options
- "Apply Filters" sticky button at bottom of sheet
- Swipe down to dismiss

States: Loading skeleton (3 card placeholders), empty search results, offline mode with cached agents, pull-to-refresh animation.

Touch targets: All buttons and tappable areas minimum 44x44px. Cards have 16px padding.
```

#### Prompt F-010: Mobile Copilot Chat

```
Design the Copilot chat interface for ERP-AI at 390px mobile width.

Layout:
- Sticky top bar: Back arrow, "Copilot" title, conversation options (three-dot menu)
- Full-screen chat thread
- Sticky bottom input bar

Chat thread:
- User messages: Right-aligned bubbles, indigo background, max width 80%
- Copilot responses: Left-aligned bubbles, white with violet left border, max width 90%
  - Confidence badge inline
  - Code blocks: Horizontal scrollable with copy button
  - Charts: Compact versions, tap to expand to full screen
  - Action buttons stack vertically: "Execute as workflow", "Save to report"
- Feedback: Thumbs up/down below each AI response
- Guardrail banner: Compact orange stripe with expand for details

Input bar (sticky bottom):
- Multi-line input (expands up to 4 lines)
- Attach button (left)
- Send button (right, violet)
- Above input: Horizontal scrollable suggestion chips

Conversation list (accessible from three-dot menu > "All Conversations"):
- Full-screen overlay list
- Search at top
- Conversation items: Title, preview, timestamp
- Swipe left to delete

States: Empty conversation with example prompts, typing indicator, long message with "Read more" truncation, error with retry, offline queue indicator.
```

#### Prompt F-011: Mobile Pipeline Monitor

```
Design the ML Pipeline monitoring page for ERP-AI at 390px mobile width.

Layout:
- Sticky top bar: Hamburger menu, "ML Pipelines" title, "+" button to create
- Bottom navigation bar: Catalog, Orchestrator, Copilot, Pipelines (active), More

Content:
1) Metrics summary (horizontal scrollable, 2 visible at a time):
   - Active Pipelines: 12
   - Success Rate: 98.7%
   - Runs Today: 847
   - Avg Duration: 4m 23s

2) Pipeline cards (single column):
   Each card:
   - Pipeline name (bold), status badge (right-aligned)
   - Progress bar if running with step indicator
   - Last run time and duration
   - Quick actions row: View, Run, Pause (icon buttons)
   - Tap to navigate to detail page

3) Pipeline detail page (separate screen):
   - Vertical step timeline (instead of DAG): Each step as a card with status, duration, expand for logs
   - Model metrics: Compact stat tiles (2x2)
   - "View Full Logs" button opens bottom sheet with scrollable log viewer

4) Pull-to-refresh for live status updates

States: No pipelines (setup CTA), running pipeline with live step progress, failed pipeline with error summary card, all complete.
```

#### Prompt F-012: Mobile Guardrail Alerts

```
Design the Guardrail alerts and violations page for ERP-AI at 390px mobile width.

Layout:
- Sticky top bar: Back arrow, "Guardrail Alerts" title, filter icon
- Content: Chronological feed of violations and alerts

Content:
1) Summary strip (sticky below top bar):
   - "12 violations today" with severity breakdown: 1 Critical (red), 4 High (orange), 7 Medium (amber)

2) Alert feed (single column):
   Each alert card:
   - Severity icon and color bar (left edge): Critical (red), High (orange), Medium (amber), Low (gray)
   - Rule name: "PII Detection & Redaction"
   - Agent name: "Customer Sentiment Analyzer"
   - Timestamp: "10 min ago"
   - Preview: "Detected SSN pattern in output..."
   - Quick actions: "Review", "Dismiss", "Escalate"
   - Tap to expand with full violation context

3) Filter bottom sheet (triggered by filter icon):
   - Severity: Critical, High, Medium, Low
   - Rule category: Content Safety, PII, Compliance, Quality
   - Time range: Last 1h, 6h, 24h, 7d
   - "Apply" sticky button

4) Bulk action mode: Long-press to select, floating action bar with "Mark Reviewed", "Escalate Selected"

States: No violations (green success illustration with "All clear"), critical alert with vibration/haptic hint notation, loading feed, offline with last-synced timestamp.
```

### 3.4 Tablet/Responsive (1024px)

```
Tablet adaptations for ERP-AI at 1024px:

General rules:
- Left sidebar collapses to icon-only mode (56px) by default; expand on hover or tap
- Content area uses full remaining width
- Agent catalog grid: 2 columns instead of 3
- Copilot layout: Hide conversation list sidebar; access via header dropdown; chat + context panel remain as two columns (60/40 split)
- Pipeline DAG: Horizontal scroll for wide workflows; fit-to-view default
- Tables: Horizontally scrollable with sticky first column (name)
- KPI metric tiles: 2x2 grid instead of 4-across row
- Guardrail rule list: Card view instead of table for better readability
- Modals: Max width 640px, centered
- Command palette: Full width minus 64px padding on each side
- Charts: Maintain full fidelity; reduce legend to icon-only with tooltip
- Bottom navigation bar: Hidden; all navigation via collapsible sidebar

Specific adaptations:
- Orchestrator workflow builder: Canvas takes 65%, config panel 35%
- Copilot: Chat thread gets 60%, context panel 40%, conversation list becomes a slide-out drawer
- NLP Analytics: Charts stack to 1 column below metrics row; embed explorer scatter plot maintains interactivity
- Pipeline cards: 2-column grid
- Guardrail table: Switch to card-based layout with sort/filter dropdown above
```

---

## 4. Make Automation Prompts

### Prompt M-001: Agent Health Degradation Response

```
Build a Make scenario for automated agent health monitoring in ERP-AI:

Trigger: Webhook — receives event from Redpanda topic "erp.ai.agent-catalog.health.changed"
Payload: { "agent_id": "agent_lead_scoring", "agent_name": "Lead Scoring Agent", "previous_status": "healthy", "current_status": "degraded", "latency_p95_ms": 890, "error_rate": 0.04, "timestamp": "2026-02-23T10:15:00Z", "tenant_id": "tenant_acme" }

Steps:
1. Parse JSON payload: Extract agent_name, current_status, latency_p95_ms, error_rate, tenant_id.
2. Router:
   a. If current_status == "degraded":
      - POST to /v1/guardrail to activate "Degraded Agent Disclaimer" guardrail for this agent.
      - Send Slack notification to #ai-ops: "Agent '{agent_name}' degraded. Latency: {latency_p95_ms}ms, Error rate: {error_rate * 100}%. Investigating."
      - Create incident in PagerDuty (P3 severity).
   b. If current_status == "offline":
      - POST to /v1/agent-orchestrator to reroute workflows using this agent to fallback agent.
      - Send Slack alert to #ai-critical: "CRITICAL: Agent '{agent_name}' is OFFLINE. Workflows rerouted to fallback."
      - Create incident in PagerDuty (P1 severity).
      - Send email to ai-platform-owners@company.com with incident details.
   c. If current_status == "healthy" AND previous_status != "healthy":
      - POST to /v1/guardrail to deactivate "Degraded Agent Disclaimer".
      - Send Slack notification to #ai-ops: "Agent '{agent_name}' recovered. Status: Healthy."
      - Resolve PagerDuty incident.
3. Log event to Google Sheets "Agent Health Audit Log": timestamp, agent_name, previous_status, current_status, actions_taken.

Guardrails: Deduplicate by agent_id + tenant_id within 5-minute window. Rate limit Slack messages to 1 per agent per 10 minutes. Preserve audit trail for compliance.
```

### Prompt M-002: Guardrail Violation Escalation Pipeline

```
Build a Make scenario for guardrail violation escalation in ERP-AI:

Trigger: Webhook — receives event from Redpanda topic "erp.ai.guardrail.violation.detected"
Payload: { "violation_id": "viol_8f3a", "rule_name": "PII Detection & Redaction", "rule_category": "PII Protection", "severity": "critical", "agent_id": "agent_sentiment", "agent_name": "Customer Sentiment Analyzer", "input_hash": "sha256:abc123", "violation_detail": "SSN pattern detected in agent output", "tenant_id": "tenant_acme", "timestamp": "2026-02-23T10:22:00Z" }

Steps:
1. Parse and validate payload.
2. Router by severity:
   a. If severity == "critical":
      - Immediately POST to /v1/agent-orchestrator to pause all active workflows containing this agent.
      - POST to /v1/copilot to inject advisory message into active copilot sessions referencing this agent.
      - Send Slack alert to #guardrail-critical with violation details, agent link, and "Investigate" deep link.
      - Create Jira ticket: Summary "CRITICAL Guardrail Violation: {rule_name} on {agent_name}", Priority P1, assigned to AI Safety team.
      - Send email to compliance-officer@company.com and dpo@company.com.
   b. If severity == "high":
      - Log to violation database via POST /v1/guardrail/violations.
      - Send Slack notification to #guardrail-alerts.
      - If violation count for this agent + rule > 10 in 24h: Auto-escalate to critical path.
   c. If severity == "medium" or "low":
      - Log to violation database.
      - Aggregate into daily digest (store in Google Sheets, send at 09:00 UTC).
3. Update agent risk score: POST to /v1/agent-catalog/{agent_id}/risk-score with incremented score.
4. Log all actions to immutable audit trail.

Guardrails: Never expose actual PII in Slack/email messages — only hashes and references. Idempotent by violation_id. Manual override path for false positives.
```

### Prompt M-003: ML Pipeline Run Completion Processor

```
Build a Make scenario for ML pipeline run completion processing in ERP-AI:

Trigger: Webhook — receives event from Redpanda topic "erp.ai.ml-pipeline.run.completed"
Payload: { "pipeline_id": "pipe_churn_v3", "pipeline_name": "Customer Churn Prediction v3", "run_id": "run_9e2b", "status": "success", "duration_seconds": 372, "model_metrics": { "accuracy": 0.942, "f1_score": 0.91, "auc_roc": 0.96 }, "previous_metrics": { "accuracy": 0.938, "f1_score": 0.89, "auc_roc": 0.95 }, "artifact_url": "s3://erp-ai-models/churn_v3/run_9e2b/", "triggered_by": "scheduled", "tenant_id": "tenant_acme", "timestamp": "2026-02-23T06:12:00Z" }

Steps:
1. Parse payload and compute metric deltas (current vs previous).
2. If status == "success":
   a. Compare model_metrics to previous_metrics:
      - If accuracy improved by > 0.5%: Flag as "significant improvement".
      - If accuracy degraded by > 1%: Flag as "regression alert".
   b. If significant improvement:
      - Send Slack to #ml-wins: "Pipeline '{pipeline_name}' improved! Accuracy: {accuracy} (+{delta}%). Ready for review."
      - Create deployment review ticket in Jira: "Review and deploy {pipeline_name} run {run_id}".
   c. If regression alert:
      - Send Slack alert to #ml-alerts: "REGRESSION: Pipeline '{pipeline_name}' accuracy dropped to {accuracy} ({delta}%). Do NOT auto-deploy."
      - Block auto-deployment via POST /v1/ml-pipeline/{pipeline_id}/deployments with hold flag.
      - Create investigation ticket in Jira.
   d. Store metrics snapshot in Google Sheets "ML Model Registry" for historical tracking.
3. If status == "failed":
   - Send Slack to #ml-alerts with error details.
   - If 3+ consecutive failures: Create PagerDuty incident and pause schedule.
4. Generate daily model performance summary at 08:00 UTC: Aggregate all pipeline runs, compute trends, send executive summary to #ml-daily.

Guardrails: Idempotent by run_id. Never auto-deploy models that triggered regression alerts. Retain all metric snapshots for audit compliance.
```

### Prompt M-004: Copilot Conversation Quality Monitor

```
Build a Make scenario for monitoring copilot conversation quality in ERP-AI:

Trigger: Schedule — Every 1 hour

Steps:
1. HTTP GET /v1/copilot/conversations/metrics?window=1h
   Response includes: { "total_conversations": 234, "avg_messages_per_conversation": 6.2, "positive_feedback_rate": 0.87, "negative_feedback_rate": 0.05, "no_feedback_rate": 0.08, "guardrail_interventions": 3, "avg_response_latency_ms": 1240, "escalation_to_human_rate": 0.02, "top_topics": ["revenue forecasting", "lead scoring", "compliance"], "low_confidence_responses": 12 }

2. Evaluate quality thresholds:
   a. If negative_feedback_rate > 0.10: Alert — "Copilot negative feedback rate elevated: {rate}%"
   b. If avg_response_latency_ms > 3000: Alert — "Copilot response latency degraded: {latency}ms"
   c. If escalation_to_human_rate > 0.05: Alert — "High escalation rate: {rate}% of conversations need human intervention"
   d. If guardrail_interventions > 10: Alert — "Elevated guardrail interventions: {count} in last hour"

3. For each triggered alert:
   - Send Slack notification to #copilot-quality with metric details and trend vs previous hour.
   - If 3 consecutive hours with same alert: Escalate to PagerDuty and send email to ai-product-team@company.com.

4. Regardless of alerts, append hourly metrics row to Google Sheets "Copilot Quality Metrics" for dashboarding.

5. Daily at 09:00 UTC: Aggregate 24h metrics, compute daily trends, generate summary report, send to #copilot-daily and stakeholder email list.

Guardrails: No PII in metrics or alerts. Rate limit PagerDuty escalations to 1 per alert type per 6 hours. Cache previous hour metrics for trend comparison.
```

### Prompt M-005: Embedding Index Health and Sync

```
Build a Make scenario for embedding index health monitoring in ERP-AI:

Trigger: Schedule — Every 15 minutes

Steps:
1. HTTP GET /v1/embedding/indices
   Response: Array of embedding index objects with { "index_id", "name", "vector_count", "dimension", "status", "last_sync", "sync_lag_seconds", "similarity_search_p95_ms" }

2. For each index, evaluate:
   a. If status != "healthy": Flag as unhealthy.
   b. If sync_lag_seconds > 300 (5 min): Flag as sync-delayed.
   c. If similarity_search_p95_ms > 200: Flag as slow search.

3. For unhealthy indices:
   - Send Slack alert to #embedding-ops: "Embedding index '{name}' is {status}. Vector count: {vector_count}. Last sync: {last_sync}."
   - POST /v1/embedding/indices/{index_id}/rebuild to trigger rebuild.
   - If rebuild fails: Create PagerDuty incident.

4. For sync-delayed indices:
   - Send Slack notification to #embedding-ops: "Index '{name}' sync lag: {sync_lag_seconds}s."
   - POST /v1/embedding/indices/{index_id}/sync to trigger manual sync.

5. For slow search indices:
   - Log to Google Sheets "Embedding Performance Log".
   - If slow for 3+ consecutive checks: Send alert to #embedding-ops with investigation link.

6. Aggregate health summary: "{healthy_count}/{total_count} indices healthy, {total_vectors} total vectors across {total_indices} indices."
   Post to #embedding-daily at 08:00 UTC.

Guardrails: Deduplicate rebuild triggers per index within 1 hour. Rate limit Slack to 1 alert per index per 30 minutes. Log all sync and rebuild actions for audit.
```

---

## 5. Prompt Usage Guidelines

### 5.1 Figma Prompt Best Practices
1. **Copy-paste directly** into Figma Make's prompt input. Each prompt is self-contained.
2. **Generate at the specified breakpoint** first (1440px for desktop, 390px for mobile), then use Figma's responsive features to verify at intermediate widths.
3. **Always generate both light and dark variants** unless the prompt specifies otherwise.
4. **Review AIDD guardrails** against each generated output before accepting — use the handoff gate template in Section 8.
5. **Iterate on component-level prompts** (F-001, F-002) before page-level prompts to establish a consistent design system.
6. **Use realistic data** from the sample values provided; avoid lorem ipsum.
7. **Verify accessibility** after generation: Run Figma contrast checker plugins and verify focus order annotations.

### 5.2 Make Automation Best Practices
1. **Set up webhook endpoints** in your Make environment before deploying scenarios.
2. **Use environment variables** for API keys, Slack tokens, and endpoint URLs — never hardcode credentials.
3. **Test with sample payloads** provided in each prompt before connecting to live Redpanda topics.
4. **Configure error handling** on every HTTP module: Retry 3x with exponential backoff (30s, 60s, 120s).
5. **Set up a dead-letter queue** scenario to capture events that fail all retries.
6. **Rate limit external notifications** (Slack, PagerDuty, email) as specified in each scenario's guardrails section.
7. **Monitor Make scenario execution logs** weekly for silent failures or quota exhaustion.

---

## 6. Output Packaging Convention

Request Figma Make outputs under these pages:

```
00_Context — Project brief, personas, architecture diagram reference
01_IA_Flows — Information architecture, navigation map, user flow diagrams
02_Design_Tokens — Color, typography, spacing, elevation, motion tokens (light + dark)
03_Components — Full component library with all state variants
04_Desktop_1440 — All desktop page designs
05_Tablet_1024 — Tablet-adapted layouts
06_Mobile_390 — All mobile page designs
07_States_And_Errors — Empty, loading, error, success, offline, degraded states for every surface
08_Observability_And_Testability — Analytics event map, performance instrumentation surfaces, test state matrix
09_Handoff_AIDD_Checklist — Completed AIDD gate template per page
```

---

## 7. Performance Acceptance Targets

| Metric | Target | Measurement |
|--------|--------|-------------|
| LCP | <= 2.5s at p75 | Mid-tier mobile on 4G |
| INP | <= 200ms at p75 | All interactive surfaces |
| CLS | <= 0.10 | All pages |
| Time to first interaction | <= 1.8s | Key routes (Catalog, Copilot, Pipelines) |
| JS route budget | <= 220KB gzip | Per initial route payload |
| API timeout feedback | 2-3s | User-visible spinner then retry path |
| Skeleton render | < 100ms | Before real content loads |
| Agent catalog search | <= 300ms | Client-side filter + API response |
| Copilot response display | Streaming | First token visible < 500ms |
| Pipeline DAG render | <= 1s | For workflows up to 20 nodes |

---

## 8. AIDD Handoff Gate Template

Attach this completed checklist to every generated design before engineering handoff:

```
[ ] 1. Accessibility gate passed
      - Color contrast ratios verified (4.5:1 normal text, 3:1 large text)
      - Focus order annotated for all interactive elements
      - Keyboard navigation paths documented
      - Screen-reader announcements specified for dynamic content (agent status changes, pipeline progress, guardrail alerts)
      - Tap targets >= 44x44px on touch surfaces

[ ] 2. Performance gate passed
      - Route-level lazy loading boundaries defined
      - JS bundle budget per route documented
      - Skeleton/placeholder dimensions match final layout (no CLS)
      - Heavy components (DAG visualizer, embedding scatter plot, charts) marked for deferred loading
      - Image strategy: responsive srcset, WebP/AVIF, lazy below fold

[ ] 3. Reliability gate passed
      - Error states defined for every API-dependent surface
      - Retry paths available for failed agent executions, pipeline runs, copilot messages
      - Offline/degraded behavior specified (cached catalog, queued messages)
      - Guardrail intervention UI handles graceful content redaction

[ ] 4. Observability gate passed
      - Analytics events mapped: page views, agent executions, pipeline runs, copilot interactions, guardrail evaluations
      - Error events instrumented with trace ID correlation
      - Performance marks (LCP, INP, CLS) identified per page
      - Conversion/engagement funnels documented

[ ] 5. Testability gate passed
      - State matrix (default, loading, error, empty, success, degraded) mapped to test cases
      - Component variants documented for visual regression testing
      - API mock requirements specified per page
      - E2E critical paths identified: Agent execute, Pipeline run, Copilot conversation, Guardrail violation flow

[ ] 6. Security and privacy gate passed
      - PII handling: Redaction rules enforced in all UI surfaces
      - Tenant isolation: No cross-tenant data leakage in any component
      - Auth: JWT verification required before rendering protected content
      - Audit: All destructive actions logged with user, timestamp, and action detail
```

---

## 9. Revision History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2026-02-23 | AIDD System | Initial prompt pack covering all 7 ERP-AI services: agent-catalog, agent-orchestrator, copilot-service, embedding-service, guardrail-service, ml-pipeline-service, nlp-service |
