# Figma & Make Prompts — ERP-Autonomous-Coding
> Version: 1.0 | Last Updated: 2026-02-23 | Status: Draft
> Classification: Internal | Author: AIDD System

---

## 1. Purpose

This prompt pack provides production-ready Figma Make design prompts and Make automation scenario prompts for **ERP-Autonomous-Coding**, the AI autonomous coding platform of the ERP suite. ERP-Autonomous-Coding comprises six microservices — agent-core, git-bridge, ide-server, review-engine, sandbox-runtime, and task-planner — connected through a Gateway/API layer authenticated via ERP-IAM (OIDC/JWT). The module integrates with source control providers (GitHub, GitLab, Bitbucket, Azure DevOps) and IDE plugins (JetBrains, VS Code, Vim/Neovim, Emacs).

Designers use the Figma prompts to generate screens, components, and flows for the autonomous coding experience. Automation engineers use the Make prompts to build operational workflows that support the coding platform.

---

## 2. AIDD Guardrails (Apply To All Prompts)

These are non-negotiable constraints that must be enforced in every generated output.

### 2.1 User Experience And Accessibility
- Meet WCAG 2.1 AA for color contrast, keyboard access, focus visibility, and semantic structure.
- Keep tap targets at least 44x44px on touch devices.
- Provide clear empty, loading, error, and success states for every asynchronous surface.
- Code editor and diff viewer components must support high-contrast themes for readability.
- Keyboard navigation must be comprehensive — developers expect full keyboard operability.
- Syntax-highlighted code must maintain WCAG-compliant contrast ratios against both light and dark backgrounds.
- Screen readers must announce task status changes, review verdicts, and sandbox execution results.

### 2.2 Performance And Frontend Efficiency
- Design for route-level lazy loading and code splitting by default.
- Keep initial route JS budget under 220KB gzip for dashboard routes.
- Code editor, diff viewer, and file tree are heavy components — defer loading until route is active.
- Keep animation subtle (150-200ms) and non-blocking; code execution progress uses minimal animation.
- Enforce skeleton UIs for task lists, review queues, and file trees.
- Sandbox terminal output must stream efficiently without blocking the UI thread.
- Syntax highlighting must be lazy-applied per visible viewport, not on entire files.

### 2.3 Reliability, Trust, And Safety
- Build explicit recovery paths for sandbox execution failures, git operations, and review engine errors.
- All AI-generated code must display clear provenance: which model, what prompt, confidence level.
- Sandbox execution must show clear isolation indicators — users must know code runs in a safe environment.
- Review engine verdicts must include rationale, not just pass/fail.
- Git operations (commit, push, branch) must have confirmation dialogs for destructive actions.
- Never auto-push code without explicit user approval.

### 2.4 Observability And Testability
- Every critical flow must define analytics events, error events, and performance checkpoints.
- Produce handoff notes that map UI states to test cases (unit, integration, E2E).
- Include performance instrumentation surfaces (LCP, INP, CLS, API latency markers).
- Task execution flows must expose trace IDs for distributed tracing.
- Sandbox execution logs must be fully downloadable and searchable.

---

## 3. Figma Design Prompts

### 3.1 Design System Foundation

#### Prompt F-001: Core Design Tokens

```
Create a comprehensive design token system for an AI autonomous coding platform (ERP-Autonomous-Coding).

Token categories:

1) Color Roles:
   - Primary: Deep slate (#1E293B) for navigation, headers, and primary surfaces
   - Accent: Bright emerald (#10B981) for primary actions, success states, "Run" buttons
   - Secondary Accent: Electric blue (#3B82F6) for links, secondary actions, information
   - Surface: White (#FFFFFF) cards on cool gray (#F1F5F9) background
   - Editor Background: Near-black (#0F172A) for code editor dark mode, off-white (#FAFAFA) for light mode
   - Success: Green (#22C55E) for passed reviews, successful builds, tests passing
   - Warning: Amber (#F59E0B) for review warnings, partial test failures, sandbox alerts
   - Error: Red (#EF4444) for failed builds, review rejections, syntax errors, broken tests
   - Info: Blue (#3B82F6) for informational annotations, code suggestions
   - AI Indicator: Violet (#8B5CF6) for AI-generated content markers, agent activity indicators
   - Diff Added: Green (#BBF7D0 bg, #166534 text) for added lines in diffs
   - Diff Removed: Red (#FECACA bg, #991B1B text) for removed lines in diffs
   - Diff Modified: Blue (#BFDBFE bg, #1E40AF text) for modified lines in diffs
   - Dark mode variants for every color role

2) Typography Scale:
   - Font family (UI): Inter, system-ui, sans-serif
   - Font family (Code): JetBrains Mono, Fira Code, Cascadia Code, monospace
   - Display: 32px/2rem, weight 700 (page titles)
   - H1: 28px/1.75rem, weight 600 (section headers)
   - H2: 22px/1.375rem, weight 600 (card titles)
   - H3: 18px/1.125rem, weight 500 (subsection headers)
   - Body: 15px/0.9375rem, weight 400, line-height 1.5 (UI text)
   - Body Small: 13px/0.8125rem, weight 400 (metadata, timestamps)
   - Caption: 11px/0.6875rem, weight 500 (badges, labels, file sizes)
   - Code: JetBrains Mono, 14px/0.875rem, line-height 1.6 (code editor, diffs, terminal)
   - Code Small: JetBrains Mono, 12px/0.75rem (inline code references, file paths)

3) Spacing Scale:
   - 2px (2xs) | 4px (xs) | 8px (sm) | 12px (md) | 16px (lg) | 24px (xl) | 32px (2xl) | 48px (3xl)

4) Border Radius:
   - None: 0px (code blocks, terminal output)
   - Subtle: 4px (inputs, inline badges, code annotations)
   - Default: 6px (cards, panels)
   - Rounded: 8px (modals, floating panels)
   - Pill: 9999px (status badges, tags)

5) Elevation:
   - Level 1: 0 1px 3px rgba(0,0,0,0.06) (cards at rest)
   - Level 2: 0 4px 8px rgba(0,0,0,0.08) (hover cards, dropdowns)
   - Level 3: 0 8px 16px rgba(0,0,0,0.10) (modals, command palette)
   - Level 4: 0 16px 32px rgba(0,0,0,0.14) (IDE overlay panels)
   - Code focus: inset 0 0 0 2px rgba(59, 130, 246, 0.5) (active editor border)

6) Motion:
   - Instant: 50ms (cursor blink, character input)
   - Fast: 100ms ease-out (button press, toggle, tab switch)
   - Default: 150ms ease-in-out (panel slide, card hover)
   - Smooth: 200ms ease-in-out (modal open, drawer expand)
   - Progress: 300ms linear (build progress, test runner bar)

Deliver both light and dark theme token sets. Dark theme is the default for this developer-focused tool. Include contrast verification annotations for all foreground/background pairs in both themes.
```

#### Prompt F-002: Component Library

```
Create an AI autonomous coding platform component library for ERP-Autonomous-Coding with these component families:

Navigation Components:
- Global top bar: Logo "ERP-Coding", breadcrumb trail, active project/repo selector dropdown, notification bell, user avatar menu
- Left sidebar: Collapsible, icon + label items for Dashboard, Task Planner, Code Editor, Reviews, Sandbox, Git, Settings; active state with accent bar; collapse to icon-only mode
- Command palette: Centered overlay (Cmd+K), search with categorized results (tasks, files, branches, reviews, commands)
- Tab bar: Horizontal tabs for open files/views with close button, modified indicator dot, drag-to-reorder

Code Components:
- Code editor: Syntax-highlighted editor area with line numbers, gutter annotations (AI suggestions, breakpoints, errors), minimap sidebar, word wrap toggle
- Diff viewer: Side-by-side and unified diff views with line-level annotations, added/removed/modified color coding, expand collapsed context, comment anchors
- File tree: Collapsible directory tree with file type icons, modified status indicators, search filter, AI-generated file badges
- Terminal/Console: Dark background terminal with monospace output, ANSI color support, scrollback buffer, copy button, clear button, resize handle
- Code suggestion inline: Ghost text (dimmed) showing AI suggestion, Tab to accept, Esc to dismiss, confidence indicator

Task Components:
- Task card: Task title, description preview, status badge (Planned/In Progress/In Review/Complete/Failed), assigned agent badge, priority indicator, estimated duration, subtask progress bar
- Task timeline: Horizontal Gantt-style view showing task dependencies, parallel execution, estimated vs actual duration
- Task detail panel: Full description, acceptance criteria checklist, subtasks list, agent assignments, git branch link, execution logs

Review Components:
- Review summary card: PR title, branch names (source -> target), file count, additions/deletions count, review verdict badge (Approved/Changes Requested/Pending), reviewer avatar
- Review comment: Inline code annotation with author, timestamp, severity (suggestion/warning/issue/critical), resolved/unresolved toggle, reply thread
- Review checklist: Automated checks list (tests pass, no security issues, code style, performance impact) with pass/fail badges

Sandbox Components:
- Sandbox environment card: Runtime name (Node 20, Python 3.12, Go 1.22), status (Running/Stopped/Building), resource usage bars (CPU, memory), execution time, "Open Terminal" button
- Execution output panel: Streaming terminal output with timestamps, error highlighting, collapsible stack traces, "Copy All" button
- Resource monitor: Real-time CPU, memory, disk usage gauges for active sandbox

Git Components:
- Branch selector: Dropdown with search, branch list grouped by (Active, Stale, Merged), create new branch option
- Commit history: Vertical list with commit hash, message, author avatar, timestamp, branch/tag decorations, AI-generated badge
- Merge/PR status: Visual pipeline showing checks (Build, Test, Lint, Review, Deploy) with pass/fail/pending for each

Status and Agent Components:
- Agent activity indicator: Avatar with animated ring (idle/thinking/coding/reviewing), current task label
- Build/test status bar: Horizontal progress bar with step labels (Install, Build, Test, Lint), pass/fail color segments
- AI confidence badge: Percentage with color gradient (green >90%, amber 70-90%, red <70%)

State variants for every component:
- Default, hover, active, focus-visible, loading (skeleton), disabled, error, success, empty state, running, queued
```

### 3.2 Desktop Pages (1440px)

#### Prompt F-003: Coding Dashboard

```
Design the main Coding Dashboard page for ERP-Autonomous-Coding at 1440px width.

Layout:
- Global top bar with "ERP-Coding" logo, breadcrumb "Dashboard", project selector "ERP-Platform / main", notifications, avatar
- Left sidebar with "Dashboard" active
- Main content area with 20px padding

Content sections:

1) Header row:
   - Greeting: "Welcome back, Sarah" with current project context "ERP-Platform"
   - Quick actions: "New Task" primary button (emerald), "Open Editor" secondary, "View Reviews" secondary
   - Branch indicator: "main" with branch selector dropdown

2) Activity overview (4 KPI tiles):
   - Active Tasks: 5 (2 in progress, 3 planned) with progress ring
   - Reviews Pending: 3 PRs awaiting review
   - Build Status: Passing (green badge with last build time "2m ago")
   - Agent Activity: 2 agents coding, 1 reviewing (animated indicators)

3) Active tasks (left column, 60%):
   Section header: "Active Tasks" with view toggle (Cards | List | Timeline)
   Task cards (stacked):
   - "Implement checkout page timeout handler" — In Progress — Agent: CodeBot Alpha — Branch: feature/checkout-timeout — 67% complete — Subtasks: 4/6 done — Est: 2h remaining
   - "Add unit tests for payment service" — In Progress — Agent: TestBot — Branch: test/payment-unit — 40% complete — Subtasks: 2/5 done — Est: 3h remaining
   - "Refactor user authentication middleware" — Planned — Priority: High — Est: 4h — Assigned to: CodeBot Beta
   - "Fix CORS configuration for API gateway" — Planned — Priority: Critical — Est: 1h — Unassigned
   - "Update OpenAPI specs for v2 endpoints" — In Review — PR #892 open — Review requested from CodeReview Agent

   Each card: Click to expand with subtask list, agent logs, diff preview

4) Review queue (right column, 40%):
   Section header: "Review Queue" with "View All" link
   Review cards:
   - PR #891: "feat: Add checkout timeout handler" — +347 / -42 lines — 8 files — Agent: CodeBot Alpha — Status: Awaiting Review — Review score: AI pre-review 87/100
   - PR #889: "test: Payment service unit tests" — +523 / -12 lines — 15 files — Agent: TestBot — Status: Changes Requested — 2 issues found
   - PR #887: "fix: CORS headers on OPTIONS requests" — +23 / -8 lines — 2 files — Human author — Status: Approved, merge ready

   Each card: File count badge, additions/deletions, review status, time since opened

5) Agent status panel (bottom, full width):
   - Horizontal cards for each active agent:
     - "CodeBot Alpha" — Status: Coding — Current: checkout-timeout — CPU: 34% — Memory: 512MB — Active for 47m
     - "TestBot" — Status: Coding — Current: payment-tests — CPU: 28% — Memory: 384MB — Active for 1h 23m
     - "CodeReview Agent" — Status: Reviewing — Current: PR #891 — Processing: file 4/8 — Active for 12m
   - Each agent: Avatar with animated status ring, real-time activity, resource usage mini-bars, "View Logs" link

6) Recent activity feed (collapsible bottom panel):
   - Chronological feed: "CodeBot Alpha committed 'Add timeout middleware' to feature/checkout-timeout (3m ago)", "TestBot created 2 new test files for PaymentService (15m ago)", "Build #1247 passed on feature/checkout-timeout (5m ago)"

States: No tasks (create first task CTA), all tasks complete (celebration), agent offline, build failing (red alert banner), sandbox resource limit warning.

Include dark mode variant (default theme).
```

#### Prompt F-004: Task Planner

```
Design the Task Planner page for ERP-Autonomous-Coding at 1440px width.

Layout:
- Global top bar with breadcrumb "ERP-Coding > Task Planner"
- Left sidebar with "Task Planner" active
- Main content area

Content sections:

1) Header:
   - Title "Task Planner" with subtitle "Plan, decompose, and assign coding tasks to AI agents"
   - Actions: "Create Task" primary button, "Import from Issue Tracker" secondary (GitHub Issues, Jira), "Bulk Import" tertiary
   - View toggle: Board (Kanban) | List | Timeline (Gantt) | Dependency Graph

2) Board view (default, Kanban columns):
   Columns: Backlog | Planned | In Progress | In Review | Complete | Failed

   Backlog:
   - "Implement SSO integration for enterprise clients" — Priority: Medium — Est: 8h — Tags: auth, enterprise
   - "Add GraphQL subscriptions for real-time updates" — Priority: Low — Est: 12h — Tags: api, graphql

   Planned:
   - "Refactor user authentication middleware" — Priority: High — Est: 4h — Agent: CodeBot Beta — Branch: (will create) — Starts: Tomorrow
   - "Fix CORS configuration for API gateway" — Priority: Critical — Est: 1h — Agent: (unassigned) — Blocked by: Auth middleware refactor

   In Progress:
   - "Implement checkout page timeout handler" — Agent: CodeBot Alpha — Branch: feature/checkout-timeout — Progress: 67% — Subtasks: 4/6
   - "Add unit tests for payment service" — Agent: TestBot — Branch: test/payment-unit — Progress: 40% — Subtasks: 2/5

   In Review:
   - "Update OpenAPI specs for v2 endpoints" — PR #892 — Review: Pending — Files: 12

   Complete (last 5):
   - "Add rate limiting to public API" — Completed 2h ago — PR #886 merged
   - "Migrate user table to new schema" — Completed yesterday — PR #884 merged

   Drag-and-drop between columns. Each card is draggable.

3) Task creation/edit modal (triggered by "Create Task"):
   - Title input: "Describe what you want to build..."
   - Description: Rich text editor with markdown support
   - AI decomposition: "Let AI break this down into subtasks" button
     - Shows decomposed subtasks with estimates, dependencies, suggested agent assignments
     - User can edit, reorder, remove subtasks
   - Configuration:
     - Priority: Critical | High | Medium | Low
     - Estimated hours (AI-suggested based on description)
     - Agent assignment: Auto (let system choose) | Specific agent dropdown
     - Git configuration: Target branch, base branch
     - Sandbox requirements: Runtime, resource limits
     - Review requirements: Auto-review | Human review required | Both
   - Dependencies: Link to other tasks (blocks/blocked-by)
   - "Create & Start" primary CTA, "Create as Draft" secondary

4) Task detail view (slide-over panel, 600px):
   - Task header: Title, status badge, priority, agent avatar
   - Tabs: Overview | Subtasks | Code Changes | Agent Logs | Timeline
   - Overview: Description, acceptance criteria, dependencies, git branch, sandbox info
   - Subtasks: Checklist with status per subtask, auto-updated by agent
   - Code Changes: Embedded diff viewer showing all changes made for this task
   - Agent Logs: Streaming log output from the agent working on this task, including:
     - "Analyzing codebase for relevant files..." (thinking)
     - "Creating file: src/middleware/timeout.ts" (coding)
     - "Running tests: 14/14 passed" (testing)
     - "Preparing pull request..." (finalizing)
   - Timeline: Visual timeline of task lifecycle events

5) Dependency graph view (alternative view):
   - DAG visualization showing task dependencies
   - Color-coded by status
   - Critical path highlighted
   - Click node to see task detail
   - Zoom and pan controls

States: Empty planner (import tasks from GitHub/Jira CTA), task decomposition in progress (AI thinking animation), agent selection with capacity indicators, task blocked by dependency, task failed with error detail and retry option.

Include dark mode variant (default theme).
```

#### Prompt F-005: Code Editor and IDE View

```
Design the Code Editor / IDE view for ERP-Autonomous-Coding at 1440px width.

Layout:
- Global top bar with breadcrumb "ERP-Coding > Editor", active task indicator "Working on: checkout-timeout"
- Left sidebar collapsed to icon-only mode for maximum editor space
- Main area: Three-panel IDE layout — File Explorer (240px) | Editor (flexible) | Right Panel (320px, collapsible)

File Explorer (left, 240px):
- Project name and branch: "ERP-Platform / feature/checkout-timeout"
- Search files input with keyboard shortcut hint (Cmd+P)
- Directory tree:
  src/
    middleware/
      timeout.ts (modified, green dot) [AI badge]
      auth.ts
      cors.ts
    services/
      payment/
        payment.service.ts
        payment.test.ts (modified, green dot) [AI badge]
      checkout/
        checkout.controller.ts (modified, green dot) [AI badge]
        checkout.service.ts
    utils/
      retry.ts [AI badge, new file]
  tests/
    integration/
      checkout.integration.test.ts (modified) [AI badge]
  package.json (modified)

- File icons by type (TS, JS, JSON, MD, etc.)
- Modified files: Green dot indicator
- AI-generated files: Small violet "AI" badge
- Right-click context menu: Open, Open Diff, View History, Copy Path

Editor Area (center, flexible):
- Tab bar: Open files as tabs with close buttons
  - timeout.ts (active), checkout.controller.ts, retry.ts (new)
  - Modified indicator (dot) on unsaved files
  - AI badge on tabs for AI-generated content

- Code editor:
  - Syntax-highlighted TypeScript code
  - Line numbers with gutter
  - AI suggestion ghost text (dimmed, showing next suggested code block)
    - "Tab to accept | Esc to dismiss | Alt+] for next suggestion"
  - Inline AI annotations:
    - Line 24: Violet left-margin marker with tooltip "AI: Added timeout configuration with 30s default based on project patterns"
    - Line 47: Amber warning marker "AI: Consider adding error type narrowing here for better type safety"
  - Error/warning squiggles from linter
  - Minimap (right edge of editor)
  - Breadcrumb path above editor: src > middleware > timeout.ts > TimeoutMiddleware > configure()

- Editor toolbar:
  - "Ask AI" button: Opens inline AI chat for code questions
  - "Explain" button: AI explains selected code
  - "Refactor" button: AI suggests refactoring for selection
  - "Generate Tests" button: Auto-generate tests for current file
  - Format, Undo, Redo, Find & Replace

Right Panel (320px, collapsible tabs):
- Tab: Agent Activity
  - Real-time feed of what the AI agent is doing:
    - "Reading checkout.service.ts to understand request flow..."
    - "Generating timeout middleware with configurable thresholds..."
    - "Running linter on timeout.ts — 0 issues"
    - "Creating test file: checkout.integration.test.ts"
  - Agent confidence for current task: 91%
  - "Pause Agent" and "Redirect Agent" buttons

- Tab: AI Chat (inline Copilot)
  - Chat thread for code-specific questions:
    User: "Why did you choose 30 seconds as the default timeout?"
    AI: "Based on the p99 latency data in your performance tests (avg 12s) and the existing timeout patterns in auth.ts (45s) and cors.ts (15s), 30s provides a safe margin. The checkout flow involves 3 sequential API calls averaging 4s each. Would you like me to make this configurable via environment variable?"
  - Input: "Ask about this code..."

- Tab: Test Runner
  - Test results tree: File > Suite > Test
  - Pass/fail indicators with execution time
  - Failed test detail with stack trace and diff
  - "Run Tests" button, "Run Current File Tests" button
  - Coverage summary: 87% statements, 92% branches

- Tab: Git Changes
  - Changed files list with additions/deletions count
  - Staged vs unstaged grouping
  - "Stage All", "Commit", "Push" buttons
  - Commit message input with AI-suggested message

Bottom Panel (collapsible, 200px):
- Tabs: Terminal | Output | Problems | Debug Console
- Terminal: Full terminal emulator with sandbox environment
- Problems: Linter errors and warnings with file:line links
- Output: Build and test output streams

States: Empty editor (open a file), AI writing code (live cursor animation in editor), AI suggestion pending (ghost text), test running (progress bar), build failing (error banner), merge conflict (highlighted conflict markers).

Include dark mode variant (default theme for this view).
```

#### Prompt F-006: Code Review Interface

```
Design the Code Review interface for ERP-Autonomous-Coding at 1440px width.

Layout:
- Global top bar with breadcrumb "ERP-Coding > Reviews > PR #891"
- Left sidebar with "Reviews" active
- Main content area

Content sections:

1) PR Header:
   - Title: "feat: Add checkout page timeout handler" with PR number #891
   - Branch: feature/checkout-timeout -> main
   - Author: CodeBot Alpha (AI agent avatar with violet badge)
   - Status: Open | Review in Progress | Changes Requested | Approved | Merged
   - Review checks bar: Build (Pass, green), Tests (Pass, green, 147/147), Lint (Pass, green), Security (Pass, green), AI Review (87/100, amber)
   - Actions: "Approve" (green), "Request Changes" (amber), "Merge" (blue, disabled until approved), "Close" (red outline)
   - Metadata: Opened 2h ago, 8 files changed, +347 / -42 lines, 3 commits

2) PR description (collapsible):
   - AI-generated summary: "This PR implements a configurable timeout middleware for the checkout flow. It adds request-level timeout handling with graceful degradation, retry logic for transient failures, and integration tests covering timeout scenarios."
   - Acceptance criteria checklist:
     [x] Timeout middleware created with configurable thresholds
     [x] Graceful degradation on timeout (return cached response or error)
     [x] Retry logic for transient failures (max 3 retries, exponential backoff)
     [x] Integration tests covering timeout scenarios
     [ ] Load testing with simulated slow responses (pending)
   - Test results summary: 147 tests passed, 0 failed, 87% coverage
   - AI confidence: 91% — "All acceptance criteria met except load testing. Code follows project patterns and passes all automated checks."

3) File browser (left, 280px):
   - Changed files tree:
     - src/middleware/timeout.ts (+187, new file) [AI generated]
     - src/services/checkout/checkout.controller.ts (+23, -8)
     - src/services/checkout/checkout.service.ts (+45, -12)
     - src/utils/retry.ts (+67, new file) [AI generated]
     - tests/integration/checkout.integration.test.ts (+89, new file) [AI generated]
     - tests/unit/timeout.test.ts (+58, new file) [AI generated]
     - package.json (+3, -1)
     - tsconfig.json (+2, -0)
   - Click file to navigate to its diff
   - Checkmark icon for reviewed files
   - Comment count badge per file

4) Diff viewer (center, flexible):
   - View toggle: Side-by-side | Unified | Rich diff
   - File header: File path, change summary (+187 lines), "Viewed" checkbox
   - Side-by-side diff:
     - Left: Previous version (or empty for new files)
     - Right: New version with syntax highlighting
     - Line numbers on both sides
     - Green background for additions, red for removals, blue for modifications
     - Collapsible unchanged context sections with "Show 20 more lines" expander
   - AI annotations (right margin, violet markers):
     - Line 15: "AI: Using exponential backoff pattern consistent with retry.ts in ERP-Platform"
     - Line 34: "AI: Timeout value sourced from environment variable with 30s fallback"
   - Inline comments:
     - Click any line number to add comment
     - Existing comments shown as expandable threads anchored to specific lines
     - Comment severity: Suggestion (blue), Warning (amber), Issue (red), Praise (green)

5) Review conversation (right panel, 360px, or below diff):
   - Chronological thread of review comments:
     - CodeReview Agent: "Overall well-structured. Consider adding request ID to timeout error responses for tracing." (Suggestion, Line 45 of timeout.ts)
     - CodeReview Agent: "The retry delay calculation on line 67 of retry.ts could overflow for attempt counts > 30. Add a max delay cap." (Issue, Line 67 of retry.ts)
     - Human reviewer (Sarah): "Good point on the overflow. Also, can we add a circuit breaker pattern for repeated timeouts?"
     - CodeBot Alpha: "Added max delay cap of 30s and circuit breaker with 5-failure threshold. See latest commit."
   - Comment input with markdown support, severity selector, file/line reference auto-fill
   - "Submit Review" button with verdict: Approve / Request Changes / Comment only

6) Commit history tab:
   - 3 commits in this PR:
     - "feat: Add timeout middleware and retry utility" — 2h ago
     - "test: Add integration and unit tests for timeout" — 1h ago
     - "fix: Add max delay cap and circuit breaker per review feedback" — 15m ago
   - Click commit to see its specific diff

States: New PR awaiting review, review in progress (AI reviewing with progress indicator showing "Analyzing file 4/8"), changes requested with unresolved comments, approved and ready to merge, merge conflict detected, merged successfully.

Include dark mode variant (default theme).
```

#### Prompt F-007: Sandbox Runtime Manager

```
Design the Sandbox Runtime manager page for ERP-Autonomous-Coding at 1440px width.

Layout:
- Global top bar with breadcrumb "ERP-Coding > Sandbox"
- Left sidebar with "Sandbox" active
- Main content area

Content sections:

1) Header:
   - Title "Sandbox Environments" with subtitle "Isolated runtimes for safe AI code execution"
   - Actions: "Create Sandbox" primary button, "Templates" secondary
   - Resource summary: "Using 3.2 / 8.0 CPU cores, 4.8 / 16 GB memory across 4 active sandboxes"

2) Active sandboxes (top section, 2-column grid):
   Each sandbox card:
   - Name and ID: "checkout-timeout-sandbox" (sandbox_7a3f)
   - Runtime badge: "Node.js 20.11" with runtime icon
   - Status: Running (green pulse) | Building (amber spinner) | Stopped (gray) | Error (red)
   - Linked task: "Implement checkout timeout handler" (clickable link)
   - Linked branch: "feature/checkout-timeout"
   - Resource usage: CPU bar (34%), Memory bar (512MB / 2GB), Disk (1.2GB / 5GB)
   - Uptime: "Active for 47 minutes"
   - Actions: "Open Terminal" (primary), "View Logs", "Stop", "Destroy"
   - Network indicator: "Outbound restricted to: npm registry, company-internal APIs"

   Sample sandboxes:
   - "checkout-timeout-sandbox" — Node.js 20.11 — Running — CPU 34% — 47m active — Agent: CodeBot Alpha
   - "payment-tests-sandbox" — Node.js 20.11 — Running — CPU 28% — 1h 23m active — Agent: TestBot
   - "auth-refactor-sandbox" — Node.js 20.11 — Building (installing dependencies...) — Agent: CodeBot Beta
   - "api-docs-sandbox" — Python 3.12 — Stopped — Last used 3h ago — No agent assigned

3) Terminal view (expanded when "Open Terminal" clicked):
   - Full-width terminal emulator with dark background
   - Command output streaming in real-time:
     ```
     $ npm test -- --coverage
     PASS src/middleware/__tests__/timeout.test.ts (2.4s)
       TimeoutMiddleware
         ✓ should timeout after configured duration (124ms)
         ✓ should return cached response on timeout (89ms)
         ✓ should retry on transient failure (201ms)
         ✓ should trigger circuit breaker after threshold (156ms)

     Test Suites: 1 passed, 1 total
     Tests:       4 passed, 4 total
     Coverage:    87% Statements | 92% Branches | 84% Functions
     ```
   - Input line at bottom for commands
   - "Copy All", "Clear", "Download Log" buttons in toolbar

4) Sandbox creation modal:
   - Runtime selector: Node.js (18, 20, 22), Python (3.10, 3.11, 3.12), Go (1.21, 1.22), Rust (1.75), Java (17, 21), Custom Dockerfile
   - Resource allocation: CPU (0.5-4 cores slider), Memory (256MB-8GB slider), Disk (1GB-20GB slider)
   - Network policy: Full access | Restricted (allowlist) | No network
   - Network allowlist editor (when Restricted): Add URLs/domains
   - Link to task (optional): Dropdown of active tasks
   - Link to branch (optional): Branch selector
   - Environment variables: Key-value editor with secret masking
   - "Create" primary CTA, estimated startup time: "~15 seconds"

5) Sandbox logs and audit (bottom section):
   - Table: Sandbox Name, Event, Timestamp, Details
   - Events: Created, Started, Package Installed, Tests Run, Build Completed, Stopped, Destroyed
   - Filter by sandbox, event type, date range
   - Export logs button

6) Resource quota visualization (sidebar widget):
   - Organization limits: CPU, Memory, Disk, Active sandboxes
   - Stacked bar showing usage by sandbox
   - "Request Quota Increase" link

States: No sandboxes (create first CTA), sandbox building with progress (dependency install log streaming), sandbox running with live terminal, sandbox resource limit hit (warning with suggestion to increase or stop idle sandboxes), sandbox error with crash log.

Include dark mode variant (default theme).
```

#### Prompt F-008: Git Bridge Operations

```
Design the Git Bridge operations page for ERP-Autonomous-Coding at 1440px width.

Layout:
- Global top bar with breadcrumb "ERP-Coding > Git"
- Left sidebar with "Git" active
- Main content area

Content sections:

1) Header:
   - Title "Git Bridge" with subtitle "Manage branches, commits, and pull requests across providers"
   - Repository selector: "ERP-Platform" dropdown (connected repos list)
   - Provider badge: GitHub icon with "github.com/company/erp-platform"
   - Actions: "Create Branch" button, "Sync" button (force sync with remote)

2) Branch overview (top section):
   - Active branches table:
     Branch | Base | Status | Agent | PR | Last Activity | Actions
     - feature/checkout-timeout | main | Active (3 commits ahead) | CodeBot Alpha | PR #891 Open | 15m ago | View, Switch, Delete
     - test/payment-unit | main | Active (5 commits ahead) | TestBot | PR #889 Changes Requested | 1h ago | View, Switch, Delete
     - feature/auth-refactor | main | Planned (no commits) | CodeBot Beta | — | Created 30m ago | View, Switch, Delete
     - fix/cors-config | main | Merged | Human | PR #887 Merged | 3h ago | View, Delete
   - Visual branch graph: Simplified git graph showing branch points and merge points
   - "Show merged branches" toggle

3) Commit history (center section):
   - Branch selector: "feature/checkout-timeout" with diff comparison "...vs main"
   - Commit list:
     - "fix: Add max delay cap and circuit breaker" — CodeBot Alpha (AI) — 15m ago — sha: a3f7b2c
     - "test: Add integration and unit tests for timeout" — CodeBot Alpha (AI) — 1h ago — sha: e8d9f1a
     - "feat: Add timeout middleware and retry utility" — CodeBot Alpha (AI) — 2h ago — sha: 4c2b7e8
   - Each commit: Hash (truncated, copyable), message, author with AI badge, timestamp, files changed count, click to view diff
   - AI-generated commits clearly badged with violet "AI" indicator

4) Pull request summary (right panel, 380px):
   - Open PRs list:
     - PR #891: "feat: Add checkout timeout handler" — AI Review: 87/100 — Checks: 4/4 passing — Reviewers: Pending
     - PR #889: "test: Payment service unit tests" — AI Review: 72/100 — Checks: 3/4 (lint warning) — Reviewers: Changes Requested
     - PR #892: "docs: Update OpenAPI specs v2" — AI Review: 95/100 — Checks: 4/4 — Reviewers: Approved
   - Each PR card: Title, review score, check status, click to navigate to full review interface
   - "Create PR" button from any branch

5) Git operations panel (bottom, collapsible):
   - Recent git operations log:
     - "CodeBot Alpha pushed 1 commit to feature/checkout-timeout" — 15m ago
     - "TestBot created branch test/payment-unit from main" — 1h 30m ago
     - "PR #887 merged into main by Sarah" — 3h ago
     - "Main branch updated: 2 new commits from merged PRs" — 3h ago
   - Manual operations: Cherry-pick, Rebase, Reset (with confirmation dialogs and safety warnings)
   - Conflict resolution: List of any merge conflicts with affected files and "Resolve" action

States: No repositories connected (setup provider CTA), repository syncing (progress indicator), merge conflict detected (amber alert with affected files), branch protection violations (error with explanation), all PRs merged (clean state).

Include dark mode variant (default theme).
```

### 3.3 Mobile Pages (390px)

#### Prompt F-009: Mobile Coding Dashboard

```
Design the Coding Dashboard for ERP-Autonomous-Coding at 390px mobile width.

Layout:
- Sticky top bar: Hamburger menu (left), "Dashboard" title (center), notifications icon (right)
- Bottom navigation bar: Dashboard (active), Tasks, Reviews, Sandbox, More

Content:

1) Project context bar (below top bar):
   - Current project: "ERP-Platform / main" with branch indicator
   - Build status badge: "Passing" (green) or "Failing" (red)

2) Metrics (horizontal scrollable, 2 visible):
   - Active Tasks: 5
   - Pending Reviews: 3
   - Build: Passing
   - Agents: 2 active

3) Active tasks (single column):
   - Compact task cards:
     - Task title (truncated 1 line)
     - Status badge and agent avatar (right)
     - Progress bar
     - Tap to navigate to task detail
   - "View All Tasks" link at bottom

4) Review queue (single column):
   - Compact PR cards:
     - PR title and number
     - Branch names (truncated)
     - Review status badge
     - Files and lines changed
     - Tap to navigate to review

5) Agent status (horizontal scrollable cards):
   - Mini agent cards: Avatar, name, status, current task (truncated)
   - Animated status ring

6) Activity feed (collapsible section):
   - Recent events in compact list format
   - Timestamp, event description, agent/user avatar

States: Loading skeleton, no tasks (create first task CTA), agents all idle, build failing alert banner.

Touch targets minimum 44x44px. Code text uses 13px JetBrains Mono for readability on mobile.
```

#### Prompt F-010: Mobile Code Review

```
Design the Code Review interface for ERP-Autonomous-Coding at 390px mobile width.

Layout:
- Sticky top bar: Back arrow, "PR #891" title, three-dot menu (approve, request changes, merge)
- No bottom navigation (full-screen review context)

Content:

1) PR summary header:
   - Title: "feat: Add checkout timeout handler" (2-line wrap)
   - Branch: feature/checkout-timeout -> main (truncated with tooltip)
   - Author: CodeBot Alpha avatar + "AI Generated" badge
   - Stats: 8 files, +347 / -42
   - Check status: 4 green check icons in row

2) Tab bar (horizontal scroll):
   - Description | Files | Conversation | Commits

3) Files tab:
   - Changed files list (full width):
     - File path (truncated from left: .../timeout.ts)
     - Change indicator: +187 (green), -0 (red)
     - "Viewed" checkbox
     - Tap to open diff viewer

4) Diff viewer (full screen when file tapped):
   - Unified diff view only (side-by-side too narrow for mobile)
   - Syntax highlighted with added/removed coloring
   - Line numbers in gutter
   - Horizontal scroll for long lines
   - Tap line number to add comment
   - AI annotations as expandable inline cards
   - Swipe left/right to navigate between files
   - "Back to file list" button at top

5) Conversation tab:
   - Thread of review comments
   - Each comment: Author, timestamp, severity badge, code context preview
   - Reply input at bottom
   - Tap code reference to jump to diff location

6) Action bar (sticky bottom when on Description tab):
   - "Approve" (green), "Request Changes" (amber), "Comment" (gray) buttons
   - Full-width layout, 48px height

States: PR loading, review in progress (AI analyzing indicator), changes requested with unresolved count badge, approved with merge button, merge conflict.
```

#### Prompt F-011: Mobile Task Planner

```
Design the Task Planner for ERP-Autonomous-Coding at 390px mobile width.

Layout:
- Sticky top bar: Hamburger menu, "Tasks" title, "+" button (create task)
- Bottom navigation bar: Dashboard, Tasks (active), Reviews, Sandbox, More

Content:

1) View selector (segmented control below top bar):
   - List (default) | Board | Timeline
   - Status filter chips (horizontal scroll): All, In Progress, Planned, In Review, Complete

2) List view (default):
   - Grouped by status with section headers and count
   - Task items (full width card):
     - Title (bold, 1 line truncated)
     - Status badge (left) + Priority indicator (right)
     - Agent avatar + name (if assigned)
     - Progress bar (if in progress)
     - Subtask count: "4/6 subtasks"
     - Tap to open detail

3) Board view:
   - Horizontal scrollable Kanban columns
   - Each column: Status header with count
   - Compact task cards within columns
   - Drag-and-drop within columns (with haptic feedback notation)
   - Tap card to open detail

4) Task detail (separate screen, slide from right):
   - Full task header with title, status, priority, agent
   - Tabs (horizontal scroll): Overview, Subtasks, Code, Logs
   - Overview: Description, acceptance criteria, branch, dependencies
   - Subtasks: Checklist with toggle
   - Code: Changed files list (tap to view diff)
   - Logs: Agent activity feed (scrollable)
   - Actions at bottom: "Start" / "Pause" / "Retry" depending on status

5) Create task (full-screen modal):
   - Title input (large)
   - Description (expandable textarea)
   - "AI Decompose" button (generates subtasks)
   - Priority selector (segmented)
   - Agent assignment dropdown
   - "Create" sticky bottom button

States: Empty tasks (import from GitHub/Jira CTA), task creation with AI decomposition in progress, task in progress with live agent activity, task failed with retry option.
```

#### Prompt F-012: Mobile Sandbox Terminal

```
Design the Sandbox terminal interface for ERP-Autonomous-Coding at 390px mobile width.

Layout:
- Sticky top bar: Back arrow, "Sandbox: checkout-timeout" title, three-dot menu (stop, destroy, logs)
- Full-screen terminal (no bottom navigation)

Content:

1) Sandbox info strip (compact, below top bar):
   - Runtime: "Node.js 20.11" badge
   - Status: "Running" green badge
   - Resources: CPU 34% | Mem 512MB (compact inline)
   - Linked task: "checkout-timeout" (tap to navigate)

2) Terminal area (full screen, dark background):
   - Monospace output with ANSI color support
   - Font: JetBrains Mono 13px
   - Scrollable output with smooth scroll
   - Touch scroll (no keyboard shortcuts on mobile)
   - Pinch-to-zoom for readability
   - Long-press to select and copy text
   - Auto-scroll to bottom (toggle with "scroll lock" button)

3) Input area (sticky bottom):
   - Command input (monospace font)
   - Send button (right)
   - Quick command chips above input (horizontal scroll): "npm test", "npm run build", "git status", "ls -la"
   - Keyboard: Show code-optimized keyboard if available

4) Quick actions menu (three-dot menu):
   - Stop sandbox
   - Restart sandbox
   - View full logs (opens log viewer)
   - Download logs
   - Sandbox settings
   - Destroy sandbox (with confirmation)

5) Resource alert (overlay banner):
   - Shown when approaching limits: "Memory usage at 90% (1.8GB / 2GB). Consider stopping idle processes."

States: Sandbox starting (build log streaming), running (active terminal), command executing (loading indicator next to input), resource limit warning, sandbox crashed (error with restart option), sandbox stopped (start button).
```

### 3.4 Tablet/Responsive (1024px)

```
Tablet adaptations for ERP-Autonomous-Coding at 1024px:

General rules:
- Left sidebar collapses to icon-only mode (48px) by default
- Dark theme is default for developer tool
- Bottom navigation bar: Hidden; all navigation via sidebar

Specific adaptations:
- Dashboard: KPI tiles in 2x2 grid; Active tasks and review queue stack vertically (full width each); Agent status remains horizontal scrollable
- Task Planner: Kanban board shows 3 visible columns (scroll for rest); Task detail panel is a slide-over at 480px; Task creation modal is centered at 560px
- Code Editor: File explorer collapses to icon buttons (toggle to expand); Editor takes full width; Right panel (agent activity, tests, git) becomes a bottom sheet or toggle panel at 320px height; Terminal panel at bottom remains at 180px
- Code Review: Diff viewer uses unified view by default (side-by-side available but narrow); File list becomes a collapsible header dropdown; Review conversation panel becomes a bottom sheet; PR header becomes more compact (2-line layout)
- Sandbox: Sandbox grid becomes single column; Terminal takes full width when open; Resource monitor becomes a compact strip
- Git Bridge: Branch table horizontally scrollable; Commit history takes full width; PR panel becomes a bottom row of cards
- Tables: Horizontally scrollable with sticky first column
- Modals: Max width 600px, centered
- Code fonts: JetBrains Mono remains at 14px; minimum 13px for readability
- Command palette: Full width minus 48px padding
```

---

## 4. Make Automation Prompts

### Prompt M-001: Task Completion and PR Pipeline

```
Build a Make scenario for automated task-to-PR pipeline in ERP-Autonomous-Coding:

Trigger: Webhook — receives event from Redpanda topic "erp.coding.task.status.changed"
Payload: { "task_id": "task_checkout_timeout", "task_name": "Implement checkout page timeout handler", "status": "complete", "previous_status": "in_progress", "agent_id": "agent_codebot_alpha", "branch": "feature/checkout-timeout", "commits": 3, "files_changed": 8, "lines_added": 347, "lines_removed": 42, "test_results": { "total": 147, "passed": 147, "failed": 0, "coverage": 87.2 }, "timestamp": "2026-02-23T10:30:00Z", "tenant_id": "tenant_acme" }

Steps:
1. Parse payload and validate task completion.

2. If status == "complete":
   a. Trigger automated code review:
      - POST /v1/review-engine/reviews with { branch, base_branch: "main", review_type: "comprehensive" }
      - Wait for review completion (webhook callback or polling with 30s intervals, max 10 min).

   b. If review score >= 85 AND no critical issues:
      - Create Pull Request via git-bridge:
        POST /v1/git-bridge/pull-requests with {
          title: "feat: {task_name}",
          body: Auto-generated from task description + test results + review summary,
          source_branch: branch,
          target_branch: "main",
          reviewers: ["human-reviewer-from-team-rotation"],
          labels: ["ai-generated", "auto-reviewed"]
        }
      - Send Slack notification to #dev-team: "PR created for '{task_name}' by {agent_id}. Review score: {review_score}/100. [View PR]"

   c. If review score < 85 OR critical issues found:
      - Send task back for revision: POST /v1/task-planner/tasks/{task_id}/revise with review feedback.
      - Notify agent to address issues.
      - Send Slack notification: "Task '{task_name}' needs revision. AI review found {issue_count} issues. Agent will address automatically."

3. If status == "failed":
   - Log failure details: POST /v1/task-planner/tasks/{task_id}/logs.
   - If failure_count < 3: Auto-retry with adjusted parameters.
   - If failure_count >= 3: Escalate to human developer via Slack DM + Jira ticket.
   - Send alert to #dev-ops: "Task '{task_name}' failed {failure_count} times. Escalating."

4. Update task tracking dashboard via POST /v1/task-planner/metrics.

Guardrails: Never auto-merge PRs — always require human approval. Rate limit PR creation to 1 per branch. Preserve all review feedback for audit. Maximum 3 automated retries before human escalation.
```

### Prompt M-002: Build and Test Pipeline Monitor

```
Build a Make scenario for monitoring build and test pipelines in ERP-Autonomous-Coding:

Trigger: Webhook — receives event from Redpanda topic "erp.coding.sandbox.build.completed"
Payload: { "sandbox_id": "sandbox_7a3f", "task_id": "task_checkout_timeout", "agent_id": "agent_codebot_alpha", "branch": "feature/checkout-timeout", "build_status": "success", "test_status": "passed", "test_results": { "total": 147, "passed": 147, "failed": 0, "skipped": 0, "coverage": 87.2, "duration_seconds": 34 }, "lint_results": { "errors": 0, "warnings": 2, "info": 5 }, "security_scan": { "critical": 0, "high": 0, "medium": 1, "low": 3 }, "build_duration_seconds": 67, "timestamp": "2026-02-23T10:25:00Z" }

Steps:
1. Parse payload and evaluate quality gates:
   a. Tests: All must pass (failed == 0). Coverage must be >= 80%.
   b. Lint: Zero errors. Warnings <= 5.
   c. Security: Zero critical or high vulnerabilities.
   d. Build: Duration must be < 300s (5 min).

2. If all gates pass:
   - Update task status: PATCH /v1/task-planner/tasks/{task_id} with quality_gates: "all_passed".
   - Log metrics to Google Sheets "Build Quality Log": branch, coverage, test count, lint score, security score, duration.

3. If any gate fails:
   a. Test failures:
      - Send Slack notification to agent channel: "Tests failed on {branch}: {failed_count} failures. See sandbox {sandbox_id} for details."
      - POST /v1/task-planner/tasks/{task_id}/subtasks/add with "Fix failing tests: [test names]".
   b. Coverage below threshold:
      - POST /v1/task-planner/tasks/{task_id}/subtasks/add with "Increase test coverage from {coverage}% to >=80%".
   c. Security vulnerabilities:
      - Send Slack alert to #security-dev: "Security scan found {severity} vulnerability in {branch}. Details: {description}."
      - If critical: Block PR creation, create urgent Jira ticket.
   d. Lint errors:
      - Auto-fix if possible: POST /v1/sandbox/{sandbox_id}/exec with "npx eslint --fix".
      - If auto-fix succeeds: Re-run build pipeline.

4. Track build quality trends:
   - Aggregate daily: avg coverage, avg build time, common failure patterns.
   - Weekly report to #dev-metrics: "This week: 47 builds, 94% pass rate, avg coverage 89%, avg build time 54s."

Guardrails: Never skip security gate for critical/high vulnerabilities. Rate limit auto-fix attempts to 2 per build. Preserve all build logs for 90 days. Do not retry builds more than 3 times automatically.
```

### Prompt M-003: Agent Resource and Performance Monitor

```
Build a Make scenario for monitoring AI agent resources and performance in ERP-Autonomous-Coding:

Trigger: Schedule — Every 5 minutes

Steps:
1. HTTP GET /v1/agent-core/agents?status=active
   Response: Array of active agents with { "agent_id", "agent_name", "status", "current_task", "sandbox_id", "cpu_percent", "memory_mb", "active_duration_minutes", "lines_written", "commits_count", "error_count" }

2. For each active agent, evaluate:
   a. Resource health:
      - If cpu_percent > 80: Flag as high CPU.
      - If memory_mb > (allocated_memory * 0.90): Flag as high memory.
      - If active_duration_minutes > 240 (4 hours): Flag as long-running.
   b. Productivity health:
      - If lines_written == 0 AND active_duration > 30 min: Flag as stalled.
      - If error_count > 10 in last hour: Flag as error-prone.

3. For flagged agents:
   a. High resource usage:
      - POST /v1/sandbox/{sandbox_id}/optimize — Request garbage collection and process cleanup.
      - If still high after 5 min: Alert to #dev-ops "Agent '{agent_name}' consuming {cpu}% CPU, {memory}MB RAM. Consider restarting sandbox."

   b. Stalled agent:
      - POST /v1/agent-core/{agent_id}/health-check — Verify agent is responsive.
      - If unresponsive: Auto-restart agent with POST /v1/agent-core/{agent_id}/restart.
      - Notify task owner: "Agent '{agent_name}' was stalled on task '{task_name}' and has been restarted."
      - If stalled 3+ times on same task: Escalate to human developer.

   c. Error-prone agent:
      - Analyze error patterns: POST /v1/agent-core/{agent_id}/error-analysis.
      - If errors are environment-related: Rebuild sandbox.
      - If errors are code-related: Add error context to task and request agent retry.
      - Send Slack alert with error pattern summary.

   d. Long-running agent:
      - Check progress: GET /v1/task-planner/tasks/{task_id}/progress.
      - If progress < 30% after 4h: Flag task as potentially blocked.
      - Notify team: "Agent '{agent_name}' has been working on '{task_name}' for {duration}. Progress: {progress}%. May need human assistance."

4. Aggregate health metrics:
   - Total agents active, total CPU, total memory, total lines written, total commits.
   - Post to #agent-metrics every 30 minutes (aggregate 6 checks).

5. Daily summary at 09:00 UTC:
   - Agent productivity: Lines written, commits, tasks completed, average task duration.
   - Resource efficiency: Avg CPU, avg memory, peak usage times.
   - Issue summary: Stalls, restarts, escalations.

Guardrails: Maximum 2 auto-restarts per agent per hour. Never kill an agent mid-commit (check git status first). Preserve agent logs before restart. Rate limit Slack alerts to 1 per agent per 15 minutes.
```

### Prompt M-004: Code Review Quality Assurance

```
Build a Make scenario for ensuring code review quality in ERP-Autonomous-Coding:

Trigger: Webhook — receives event from Redpanda topic "erp.coding.review.completed"
Payload: { "review_id": "rev_8f2a", "pr_id": "pr_891", "pr_title": "feat: Add checkout timeout handler", "branch": "feature/checkout-timeout", "author_type": "ai_agent", "author_id": "agent_codebot_alpha", "reviewer_type": "ai_engine", "review_score": 87, "issues_found": [ { "severity": "suggestion", "file": "timeout.ts", "line": 45, "message": "Consider adding request ID" }, { "severity": "issue", "file": "retry.ts", "line": 67, "message": "Overflow risk in delay calculation" } ], "coverage_delta": +2.1, "timestamp": "2026-02-23T10:35:00Z" }

Steps:
1. Parse review results and classify:
   a. Score tiers: Excellent (>=95), Good (85-94), Needs Work (70-84), Poor (<70).

2. Cross-check review against quality standards:
   a. If author_type == "ai_agent":
      - Verify all AI-generated files are flagged in PR metadata.
      - Check that AI confidence scores are included in commit messages.
      - Ensure test coverage delta is positive or neutral (not decreasing).
   b. For all PRs:
      - Verify no secrets detected (scan for API keys, tokens, passwords).
      - Verify no hardcoded URLs or environment-specific values.
      - Check that breaking changes are documented.

3. Based on score tier:
   a. Excellent (>=95):
      - Auto-assign to human reviewer from team rotation for quick rubber-stamp.
      - Mark as "Fast Track Review" in PR labels.
      - Send Slack: "PR #{pr_id} scored {score}/100 — fast-track review recommended."

   b. Good (85-94):
      - Assign to human reviewer with review notes highlighting the AI-found issues.
      - Send Slack with summary of issues to address.

   c. Needs Work (70-84):
      - Send PR back to agent for revisions: POST /v1/task-planner/tasks/{task_id}/revise with issues list.
      - Do not assign human reviewer until agent addresses issues.
      - Send Slack: "PR #{pr_id} scored {score}/100 — agent will address {issue_count} issues before human review."

   d. Poor (<70):
      - Block PR and escalate: Send Slack alert to #code-quality with full review details.
      - Create Jira ticket for human investigation.
      - Notify agent owner/team lead.

4. Track review quality metrics:
   - Store in Google Sheets "Review Quality Log": PR, score, issue types, coverage delta, author type.
   - Weekly: Compute average review scores by agent, most common issue types, coverage trends.
   - Send weekly report to #engineering-metrics.

5. Review engine calibration:
   - If human reviewers override AI review decisions >20% of the time: Flag for review engine retraining.
   - Track false positive rate per review rule.

Guardrails: Never auto-merge any PR regardless of score. Human review is always required. Preserve all review artifacts for audit. Rate limit agent revisions to 3 per PR before human escalation.
```

### Prompt M-005: Git Sync and Conflict Resolution

```
Build a Make scenario for git synchronization and conflict detection in ERP-Autonomous-Coding:

Trigger: Schedule — Every 10 minutes + Webhook on "erp.coding.git.push" events

Steps:
1. HTTP GET /v1/git-bridge/branches?status=active
   Response: Active branches with { "branch", "base_branch", "commits_ahead", "commits_behind", "last_activity", "author_type", "merge_status" }

2. For each active branch:
   a. If commits_behind > 0:
      - Attempt rebase: POST /v1/git-bridge/branches/{branch}/rebase with { base: "main" }.
      - If rebase succeeds: Log success, continue.
      - If conflict detected:
        - Identify conflicting files: GET /v1/git-bridge/branches/{branch}/conflicts.
        - If author_type == "ai_agent" AND conflict is in AI-generated files only:
          - Auto-resolve using agent: POST /v1/agent-core/{agent_id}/resolve-conflict with conflict details.
          - Run tests after resolution: POST /v1/sandbox/{sandbox_id}/exec with "npm test".
          - If tests pass: Commit resolution and continue.
          - If tests fail: Revert resolution, notify human.
        - If conflict involves human-authored files:
          - Never auto-resolve. Send Slack DM to branch owner: "Merge conflict detected on '{branch}' with main. {file_count} files in conflict. [Resolve Now]"
          - Create Jira ticket for conflict resolution if unresolved after 4 hours.

   b. If no activity for > 24 hours AND branch is not "planned":
      - Send reminder to branch owner: "Branch '{branch}' has been inactive for {hours}h. Still working on it?"
      - If inactive > 72 hours: Flag for cleanup review.

3. Monitor protected branch (main):
   a. If direct push detected (not via PR): Alert to #dev-security "Direct push to main detected! Commit: {sha}, Author: {author}."
   b. If build fails on main: P1 alert to #dev-critical "Main branch build is BROKEN. Last passing commit: {sha}."

4. Daily git health report at 08:00 UTC:
   - Active branches: count, average age, branches behind main.
   - Merged branches pending cleanup.
   - Conflict frequency trends.
   - PR merge velocity: avg time from PR open to merge.

Guardrails: Never force-push to any branch. Never auto-resolve conflicts in human-authored files. Rate limit rebase attempts to 1 per branch per 30 minutes. Preserve all conflict resolution attempts in audit log. Maximum 3 auto-resolve attempts before human escalation.
```

---

## 5. Prompt Usage Guidelines

### 5.1 Figma Prompt Best Practices
1. **Copy-paste directly** into Figma Make's prompt input. Each prompt is self-contained.
2. **Generate dark theme first** — this is a developer tool and dark mode is the primary theme. Then generate light mode as the variant.
3. **Generate at the specified breakpoint** first (1440px for desktop, 390px for mobile), then verify at intermediate widths.
4. **Review AIDD guardrails** against each generated output before accepting — use the handoff gate template in Section 8.
5. **Iterate on component-level prompts** (F-001, F-002) before page-level prompts to establish a consistent design system.
6. **Use realistic code samples** in all editor and diff views — avoid placeholder text. TypeScript is the primary language.
7. **Verify code readability**: All code text must be legible at the designed font sizes, with proper syntax highlighting contrast in both themes.
8. **Verify accessibility** after generation: Run Figma contrast checker, especially on syntax-highlighted code against editor backgrounds.

### 5.2 Make Automation Best Practices
1. **Set up webhook endpoints** in your Make environment before deploying scenarios.
2. **Use environment variables** for API keys, Slack tokens, git provider tokens, and endpoint URLs — never hardcode.
3. **Test with sample payloads** provided in each prompt before connecting to live Redpanda topics.
4. **Configure error handling** on every HTTP module: Retry 3x with exponential backoff (30s, 60s, 120s).
5. **Set up a dead-letter queue** scenario to capture events that fail all retries.
6. **Rate limit external notifications** as specified in each scenario's guardrails section.
7. **Git safety is paramount**: Never auto-merge, never force-push, never resolve human-authored conflicts automatically.
8. **Monitor Make scenario execution logs** weekly for silent failures or quota exhaustion.
9. **Preserve audit trails**: All code changes, reviews, and agent actions must be logged for compliance.

---

## 6. Output Packaging Convention

Request Figma Make outputs under these pages:

```
00_Context — Project brief, personas (developer, tech lead, DevOps engineer), architecture diagram reference
01_IA_Flows — Information architecture, navigation map, user flow diagrams (task creation flow, code review flow, sandbox lifecycle, git operations)
02_Design_Tokens — Color, typography, spacing, elevation, motion tokens (dark + light themes)
03_Components — Full component library with all state variants (code, task, review, sandbox, git, agent)
04_Desktop_1440 — All desktop page designs (Dashboard, Task Planner, Code Editor, Reviews, Sandbox, Git Bridge)
05_Tablet_1024 — Tablet-adapted layouts
06_Mobile_390 — All mobile page designs
07_Code_Views — Dedicated page for code editor states, diff viewer states, terminal states, syntax highlighting samples
08_States_And_Errors — Empty, loading, error, success, running, queued, conflict, build-fail states for every surface
09_Observability_And_Testability — Analytics event map, performance instrumentation surfaces, test state matrix
10_Handoff_AIDD_Checklist — Completed AIDD gate template per page
```

---

## 7. Performance Acceptance Targets

| Metric | Target | Measurement |
|--------|--------|-------------|
| LCP | <= 2.5s at p75 | Mid-tier mobile on 4G |
| INP | <= 200ms at p75 | All interactive surfaces |
| CLS | <= 0.10 | All pages |
| Time to first interaction | <= 1.8s | Key routes (Dashboard, Task Planner, Reviews) |
| JS route budget | <= 220KB gzip | Per initial route payload (excluding code editor) |
| Code editor load | <= 2s | From route navigation to editable state |
| Diff viewer render | <= 1s | For diffs up to 2000 lines |
| File tree render | <= 500ms | For repositories up to 10,000 files |
| Terminal output streaming | Real-time | < 50ms latency between output and display |
| Syntax highlighting | <= 200ms | Per visible viewport (lazy-apply for large files) |
| Task planner board render | <= 1s | For boards with up to 100 tasks |
| Search results | <= 300ms | File search and command palette |
| Git branch list | <= 500ms | For repositories with up to 500 branches |

---

## 8. AIDD Handoff Gate Template

Attach this completed checklist to every generated design before engineering handoff:

```
[ ] 1. Accessibility gate passed
      - Color contrast ratios verified for both light and dark themes (4.5:1 normal text, 3:1 large text)
      - Syntax highlighting contrast verified against editor backgrounds in both themes
      - Focus order annotated for all interactive elements, including code editor tab navigation
      - Keyboard navigation comprehensive — developers expect full keyboard operability
      - Screen-reader announcements for task status changes, build results, review verdicts, agent activity
      - Tap targets >= 44x44px on touch surfaces

[ ] 2. Performance gate passed
      - Route-level lazy loading boundaries defined (code editor, diff viewer, terminal are heavy)
      - JS bundle budget per route documented
      - Code editor, diff viewer, and terminal marked for deferred loading
      - Skeleton/placeholder dimensions match final layout (no CLS)
      - Syntax highlighting applied lazily per visible viewport
      - Terminal output streaming does not block UI thread

[ ] 3. Reliability gate passed
      - Error states defined for every API-dependent surface
      - Retry paths for sandbox failures, git operations, build errors, review engine errors
      - Offline/degraded behavior: Cached task list, read-only code view, queued git operations
      - Destructive git operations (push, merge, delete branch) have confirmation dialogs
      - Sandbox isolation indicators clearly visible

[ ] 4. Observability gate passed
      - Analytics events: task creation, code changes, review submissions, sandbox operations, git operations
      - Error events with trace ID correlation across agent-core, sandbox-runtime, and git-bridge
      - Performance marks (LCP, INP, CLS) per page
      - Agent productivity metrics: lines written, commits, task completion rate, error rate

[ ] 5. Testability gate passed
      - State matrix mapped to test cases for all components
      - Code editor states: empty, loaded, editing, AI suggestion visible, error, read-only
      - Diff viewer states: small diff, large diff, conflict markers, comment threads
      - API mock requirements specified per page
      - E2E critical paths: Task create -> agent code -> build -> review -> PR -> merge

[ ] 6. Security and privacy gate passed
      - No credentials displayed in code views (masked in environment variable displays)
      - Sandbox network isolation clearly indicated
      - Git operations require authentication confirmation for destructive actions
      - AI-generated code clearly attributed and auditable
      - Tenant isolation: No cross-tenant repository or code access
      - Audit: All code changes, reviews, and agent actions logged
```

---

## 9. Revision History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2026-02-23 | AIDD System | Initial prompt pack covering all 6 ERP-Autonomous-Coding services: agent-core, git-bridge, ide-server, review-engine, sandbox-runtime, task-planner |

<!-- SOVEREIGN_FIGMA_MAKE_EXPANSION_2026_03 -->
## 2026-03 Shared Infra + AIDD + GA Expansion Pack

### Scope
This section upgrades the prompt pack to align with:
- Shared infrastructure (`Hasura + ERP-DBaaS + ERP-IAM + ERP-Observability`)
- AIDD guardrails (Protected, Supervised, Autonomous execution modes)
- GA deployment quality expectations (accessibility, performance, observability, rollback-safe UX)

### Global Prompt Rules (Apply To Every Screen)
- Use production-safe copy and deterministic states for loading/error/empty/success.
- Include tenant context, role context, and policy context in all critical admin flows.
- Require explicit confirmation UX for destructive actions; show blast radius and rollback hint.
- Include instrumentation notes: event name, trace ID propagation, KPI target, and SLO linkage.
- Ensure WCAG 2.1 AA contrast and keyboard focus maps in every frame set.

### Prompt F-900: Shared Infra Control Plane Screen
```
Design a control plane view for ERP-Autonomous-Coding that shows:
1. Hasura GraphQL connectivity (latency, success %, schema drift status)
2. ERP-DBaaS health (connection pool, replica lag, failover posture)
3. ERP-IAM integration (OIDC issuer health, token validation errors, session invalidations)
4. ERP-Observability status (OTLP export rate, tracing availability, alert health)

Include: command surface, timeline, runbook links, and one-click diagnostics panel.
Add states for: fully healthy, degraded IAM, degraded DB, degraded GraphQL, global outage.
```

### Prompt F-901: Role-Aware Workspace + Guardrail Surface
```
Generate a role-aware workspace for ERP-Autonomous-Coding with:
- Role lenses: Operator, Manager, Auditor, Admin
- Guardrail panel with mode badge (Protected / Supervised / Autonomous)
- Inline policy rationale for blocked/supervised actions
- Approval request flow for supervised actions

Ensure each role sees distinct navigation and action priorities.
```

### Prompt F-902: Incident + Recovery UX
```
Design a full incident-response flow for ERP-Autonomous-Coding:
- Alert ingestion to triage board
- Impacted tenant view and blast-radius visualization
- Action timeline with runbook steps and rollback controls
- Post-incident report generator modal

Must include: trace-id copy, audit evidence export, and SLA breach indicators.
```

### Prompt F-903: Mobile-Responsive Executive Snapshot
```
Create mobile and tablet executive dashboards for ERP-Autonomous-Coding:
- KPI cards (availability, p95 latency, error budget burn, throughput)
- AI-generated narrative summary with confidence score
- Risk highlights and next-best-action CTA

Use adaptive layout breakpoints and preserve semantic heading structure.
```

### Prompt F-904: Figma Make -> Engineering Handoff Packet
```
For each completed flow, generate a handoff packet containing:
- Component inventory and token usage map
- API contract touchpoints (GraphQL operations and IAM claims)
- Test matrix (unit/integration/e2e/accessibility/performance)
- Observability event catalog and dashboard mapping
- GA release checklist references
```

### Prompt M-900: Make Automation Blueprint (Shared Infra)
```
Create a Make scenario for ERP-Autonomous-Coding that:
1. Triggers on deployment/webhook events
2. Validates GraphQL, IAM, DB, and OTLP health in sequence
3. Creates incident ticket when thresholds are breached
4. Posts status updates to operations channels
5. Writes audit trail to persistent store

Add branching for: transient failures, policy violations, and hard-stop conditions.
```

### Prompt M-901: Tenant Onboarding Orchestration
```
Generate a Make scenario for tenant onboarding in ERP-Autonomous-Coding:
- Validate tenant metadata and compliance profile
- Provision IAM roles and claims mapping
- Apply Hasura metadata/migrations checks
- Seed tenant defaults and dashboards
- Emit onboarding completion event and SLA timer
```

### Completion Criteria
- Every screen has loading/error/empty/success/access-denied variants.
- Every critical action includes telemetry mapping and guardrail annotation.
- Every flow includes desktop + tablet + mobile-responsive frames.
- Every high-risk flow includes rollback and operator escalation UX.
