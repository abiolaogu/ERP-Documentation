# Figma & Make Prompts — ERP-Assistant
> Version: 1.0 | Last Updated: 2026-02-23 | Status: Draft
> Classification: Internal | Author: AIDD System

---

## 1. Purpose

This prompt pack provides production-ready Figma Make design prompts and Make automation scenario prompts for **ERP-Assistant**, the AI-powered personal assistant module of the ERP platform. ERP-Assistant comprises six microservices — action-engine, assistant-core, briefing-service, connector-hub, memory-service, and voice-service — connected through a Gateway/API layer authenticated via ERP-IAM (OIDC/JWT). The module integrates with external tools including Google Workspace, Microsoft 365, Notion, Slack, Jira, Trello, Asana, WhatsApp, Telegram, and storage providers.

Designers use the Figma prompts to generate screens, components, and flows for the assistant experience. Automation engineers use the Make prompts to build operational workflows that enhance the assistant's capabilities.

---

## 2. AIDD Guardrails (Apply To All Prompts)

These are non-negotiable constraints that must be enforced in every generated output.

### 2.1 User Experience And Accessibility
- Meet WCAG 2.1 AA for color contrast, keyboard access, focus visibility, and semantic structure.
- Keep tap targets at least 44x44px on touch devices.
- Provide clear empty, loading, error, and success states for every asynchronous surface.
- Maintain plain-language labels and error messages; the assistant interface must feel conversational, not technical.
- Voice interactions must provide visual feedback equivalents for deaf and hard-of-hearing users.
- Briefings and summaries must support text-to-speech readback with proper ARIA live regions.
- Memory recall surfaces must clearly indicate data freshness and source provenance.

### 2.2 Performance And Frontend Efficiency
- Design for route-level lazy loading and code splitting by default.
- Keep initial route JS budget under 220KB gzip for user-facing routes.
- Avoid heavy always-mounted UI primitives; defer advanced widgets (connector configuration panels, memory timeline) until needed.
- Keep animation subtle (150-200ms) and non-blocking.
- Enforce skeleton UIs and progressive content rendering for briefing feeds and conversation history.
- Voice waveform visualizations must not block the main thread.

### 2.3 Reliability, Trust, And Safety
- Build explicit recovery paths (retry, fallback, escalation) for voice commands, action executions, and connector syncs.
- Show clear provenance for every assistant recommendation: which data source, which connector, what confidence level.
- Memory surface must expose what the assistant remembers and allow users to delete specific memories.
- Connector failures must degrade gracefully with clear user messaging and manual fallback options.

### 2.4 Observability And Testability
- Every critical flow must define analytics events, error events, and performance checkpoints.
- Produce handoff notes that map UI states to test cases (unit, integration, E2E).
- Include performance instrumentation surfaces (LCP, INP, CLS, API latency markers).
- Voice interactions must log transcription accuracy metrics and latency.
- Action engine executions must expose trace IDs for distributed tracing.

---

## 3. Figma Design Prompts

### 3.1 Design System Foundation

#### Prompt F-001: Core Design Tokens

```
Create a comprehensive design token system for an AI personal assistant platform (ERP-Assistant).

Token categories:

1) Color Roles:
   - Primary: Deep teal (#0F766E) for navigation, headers, and primary actions
   - Accent: Bright cyan (#06B6D4) for assistant-related highlights, active states, interactive elements
   - Surface: White (#FFFFFF) cards on warm gray (#F5F5F4) background
   - Success: Emerald (#059669) for completed actions, successful connections
   - Warning: Amber (#D97706) for connection issues, stale data indicators
   - Error: Rose (#DC2626) for failed actions, disconnected integrations
   - Info: Sky blue (#0284C7) for informational badges, tips
   - Assistant Glow: Soft teal-to-cyan gradient (#0F766E to #06B6D4) for assistant UI surfaces, voice indicator
   - Voice Active: Pulsing cyan (#22D3EE) for active voice listening state
   - Memory Accent: Warm violet (#7C3AED) for memory-related surfaces and recalls
   - Dark mode variants for every color role

2) Typography Scale:
   - Font family: Inter, system-ui, sans-serif
   - Display: 36px/2.25rem, weight 700, letter-spacing -0.02em (page titles)
   - H1: 30px/1.875rem, weight 600 (section headers)
   - H2: 24px/1.5rem, weight 600 (card titles, briefing headers)
   - H3: 20px/1.25rem, weight 500 (subsection headers)
   - Body: 16px/1rem, weight 400, line-height 1.5 (default text, briefing content)
   - Body Small: 14px/0.875rem, weight 400 (metadata, timestamps, secondary info)
   - Caption: 12px/0.75rem, weight 500 (badges, labels, connector status)
   - Conversational: 16px/1rem, weight 400, line-height 1.6 (chat messages, optimized for readability)

3) Spacing Scale:
   - 4px (xs) | 8px (sm) | 12px (md) | 16px (lg) | 24px (xl) | 32px (2xl) | 48px (3xl) | 64px (4xl)

4) Border Radius:
   - Subtle: 4px (inputs, inline badges)
   - Default: 8px (cards, modals)
   - Rounded: 16px (chat bubbles, assistant panels)
   - Pill: 9999px (status badges, voice button, tags)

5) Elevation:
   - Level 1: 0 1px 3px rgba(0,0,0,0.08) (cards at rest)
   - Level 2: 0 4px 12px rgba(0,0,0,0.10) (hover cards, dropdowns)
   - Level 3: 0 8px 24px rgba(0,0,0,0.12) (modals, assistant panel)
   - Level 4: 0 16px 48px rgba(0,0,0,0.16) (command palette overlay)

6) Motion:
   - Fast: 100ms ease-out (button press, toggle)
   - Default: 150ms ease-in-out (card hover, panel slide)
   - Smooth: 250ms ease-in-out (modal open, briefing card expand)
   - Voice Pulse: 600ms ease-in-out infinite (voice listening indicator)
   - Typing: 300ms steps(3) infinite (assistant typing indicator)

Deliver both light and dark theme token sets. Include contrast verification annotations for all foreground/background pairs.
```

#### Prompt F-002: Component Library

```
Create an AI personal assistant component library for ERP-Assistant with these component families:

Navigation Components:
- Global top bar: Logo "ERP-Assistant", breadcrumb trail, notification bell with unread count, voice activation button (microphone icon with pulse ring), user avatar menu
- Left sidebar: Collapsible, icon + label items for Home/Briefings, Chat, Actions, Connectors, Memory, Voice, Settings; active state with accent bar; collapse to icon-only mode
- Command palette: Centered overlay (Cmd+K trigger), natural language input "Ask your assistant anything...", categorized results (actions, connectors, briefings, memories, settings)

Chat/Conversation Components:
- Chat bubble (user): Right-aligned, teal background, white text, timestamp, edit icon
- Chat bubble (assistant): Left-aligned, white with teal left accent border, includes:
  - Markdown rendering (headers, lists, tables)
  - Confidence indicator when making recommendations
  - Source provenance tags (e.g., "From Jira", "From Google Calendar", "From Memory")
  - Inline action buttons ("Schedule this", "Create Jira ticket", "Save to memory")
  - Feedback row: Thumbs up, thumbs down, copy, regenerate
- Typing indicator: Three animated dots with "Assistant is thinking..." text
- Voice transcription bubble: Dashed border, real-time text appearing as speech is recognized, waveform indicator

Briefing Components:
- Daily briefing card: Date header, priority-ranked items list, source icons, "Mark as read" action
- Briefing item: Icon (calendar, email, task, alert), title, 2-line summary, source connector badge, time indicator, priority color bar (left edge)
- Briefing section: Collapsible header ("Today's Calendar", "Pending Approvals", "Key Emails"), item count badge, expand/collapse animation
- Morning digest header: Greeting ("Good morning, Sarah"), date, weather (optional), quick action buttons ("Start my day", "Show priorities")

Action Components:
- Action card: Action name ("Schedule Meeting"), category badge ("Calendar"), connected service icon (Google Calendar), "Execute" button, parameter preview
- Action execution form: Dynamic form fields based on action schema, parameter suggestions from memory, "Run Action" primary CTA, progress indicator
- Action result card: Status badge (Success/Failed/Pending), result summary, affected items list, "Undo" button (if applicable), execution time
- Action history item: Timestamp, action name, status badge, parameter summary, expand for full details

Connector Components:
- Connector card: Service logo (Google, Microsoft, Slack, Jira, etc.), service name, connection status (Connected/Disconnected/Syncing/Error), last sync time, "Configure" button, data scope tags
- Connector setup wizard: Step indicator (Authorize > Configure > Test > Activate), OAuth flow placeholder, permission scope checklist, test connection button
- Sync status indicator: Animated spinner for syncing, green check for synced, red X for error, timestamp of last successful sync

Memory Components:
- Memory timeline: Vertical timeline with memory entries, grouped by date, filterable by type
- Memory card: Memory type icon (fact, preference, context, relationship), content preview, source, created date, confidence score, "Forget" button (delete with confirmation)
- Memory search: Input with placeholder "Search what I remember about...", results with highlighted match terms
- Memory category filter: Chips for Facts, Preferences, Contacts, Projects, Decisions, Custom

Voice Components:
- Voice activation button: Large circular button with microphone icon, resting state (outlined), listening state (filled with pulse animation), processing state (spinning ring)
- Voice waveform: Real-time audio waveform visualization, compact bar style
- Voice transcript overlay: Semi-transparent overlay showing real-time transcription, with "Cancel" and "Send" buttons
- Voice feedback: "I heard: [transcribed text]" confirmation bar with "Edit" and "Send" options

State variants for every component:
- Default, hover, active, focus-visible, loading (skeleton), disabled, error, success, empty state, offline/degraded, syncing
```

### 3.2 Desktop Pages (1440px)

#### Prompt F-003: Assistant Home — Daily Briefing

```
Design the Assistant Home and Daily Briefing page for ERP-Assistant at 1440px width.

Layout:
- Global top bar with "ERP-Assistant" logo, breadcrumb "Home", voice button, notifications, avatar
- Left sidebar with "Home" active
- Main content area with 24px padding

Content sections:

1) Greeting header:
   - "Good morning, Sarah" (personalized greeting based on time of day)
   - Date: "Sunday, February 23, 2026"
   - Quick stat row: 6 meetings today, 14 unread emails, 3 pending approvals, 2 overdue tasks
   - Quick actions: "Start My Day" primary button (triggers voice briefing), "Show Priorities" secondary

2) Morning briefing card (full width, prominent):
   - Header: "Your Daily Briefing" with "Listen" button (voice readback) and "Refresh" icon
   - Auto-generated executive summary (3-4 sentences): "You have a busy day ahead. The Q4 budget review with Finance is at 10 AM — the latest reports show a 3% variance that needs your attention. Your Jira sprint has 4 items due today, and Sarah from Marketing is waiting on your campaign approval since Friday."
   - Source badges inline: Google Calendar, Jira, Gmail, Slack

3) Briefing sections (2-column layout):

   Left column (60%):
   a) "Today's Schedule" (from Google Calendar connector):
      - Timeline view: 8:00 AM Stand-up (Zoom), 10:00 AM Budget Review (Conference Room B), 12:00 PM Lunch with Client (Restaurant), 2:30 PM Sprint Planning (Teams), 4:00 PM 1:1 with Manager
      - Each item: Time, title, location/link, attendees count, "Join" quick action for virtual meetings
      - Conflict indicator: Yellow highlight if overlapping

   b) "Pending Actions" (from action-engine):
      - Approve marketing campaign budget ($45,000) — from Slack, 2 days waiting
      - Review PR #847 on ERP-Platform — from GitHub, assigned yesterday
      - Sign vendor contract — from DocuSign, due today
      - Each: Priority badge, source icon, age indicator, "Take Action" button

   Right column (40%):
   c) "Key Emails" (from Gmail/Outlook connector):
      - 3-5 most important emails with sender, subject, 1-line AI summary, timestamp
      - "CFO Budget Alert: Q4 variance exceeded threshold — Action required"
      - "Client Follow-up: Acme Corp requesting updated timeline"
      - "HR: Open enrollment deadline reminder — February 28"
      - Quick actions per email: Reply, Archive, Snooze, Flag

   d) "Task Updates" (from Jira/Asana connector):
      - Sprint progress bar: "Sprint 24: 67% complete (8/12 items done)"
      - Overdue items highlighted in red
      - Items due today with status badges
      - "View Full Board" link

4) Assistant suggestion strip (bottom, full width):
   - "Based on your schedule, I suggest blocking 3:30-4:00 PM for budget review prep"
   - "You haven't responded to the CFO's email from Friday — would you like me to draft a response?"
   - Each suggestion: Text, confidence indicator, "Do it" and "Dismiss" actions

5) Floating assistant chat bubble (bottom-right corner):
   - Circular button with assistant avatar
   - Unread indicator if assistant has proactive suggestions
   - Click to expand chat panel

States: First-time user onboarding (connect your first service CTA), no connectors configured (setup wizard prompt), all caught up (celebration illustration), loading briefing (skeleton cards), connector sync error (partial briefing with warning badges).

Include dark mode variant.
```

#### Prompt F-004: Conversational Chat Interface

```
Design the full Conversational Chat interface for ERP-Assistant at 1440px width.

Layout:
- Global top bar with breadcrumb "ERP-Assistant > Chat"
- Left sidebar with "Chat" active
- Three-column layout: Conversation list (280px) | Chat thread (flexible) | Context panel (320px)

Conversation list (left, 280px):
- "New Conversation" button at top
- Search conversations input
- Pinned conversations section (collapsible)
- Conversation items: Auto-generated title, preview text, timestamp, unread badge, source icons (which connectors were referenced)
- Grouped by: Today, Yesterday, This Week, Older
- Sample conversations:
  - "Q4 Budget Review Prep" (pinned, Google Calendar + Sheets)
  - "Acme Corp Deal Status" (Jira + Salesforce)
  - "Team Sprint Planning Notes" (Jira + Slack)
  - "Travel Booking for NYC Conference" (Gmail + Calendar)
  - "Marketing Campaign Approval" (Slack + Asana)

Chat thread (center):
- Message exchange:
  User: "What's the status of the Acme Corp deal?"
  Assistant: "Based on your Jira board and last week's Slack discussions, here's the Acme Corp status:
  - **Deal stage**: Proposal Sent (moved from Negotiation on Feb 18)
  - **Value**: $240,000 ARR
  - **Next step**: Follow-up call scheduled for Tuesday 10 AM
  - **Risk**: Client requested a 15% discount — Sarah from Sales flagged this in #deals channel on Friday
  - **Action needed**: Approve the discount or propose alternative packaging
  Sources: Jira CRM-1847, Slack #deals (Feb 21), Google Calendar"
  [Action buttons: "Approve discount", "Draft counter-proposal", "Schedule follow-up"]

  User: "Draft a counter-proposal email suggesting a 10% discount with extended contract"
  Assistant: "Here's a draft email to John at Acme Corp:
  [Formatted email preview with subject, greeting, proposal details, closing]
  Confidence: High — based on your previous email tone and company discount policy.
  [Action buttons: "Send via Gmail", "Edit in Gmail", "Copy to clipboard"]"

- Message features:
  - Source provenance badges on every assistant response (connector icons + links)
  - Inline data cards: Calendar event preview, Jira ticket summary, email thread preview
  - Action buttons contextually generated based on conversation
  - Code blocks for technical content with syntax highlighting
  - Tables for structured data (deal comparison, schedule overview)
  - Feedback: Thumbs up/down on every assistant response

Voice input mode:
- Voice button in input bar — click to activate
- Listening state: Waveform animation, real-time transcription appearing in input field
- "I heard: [text]" confirmation before sending

Input area (bottom):
- Multi-line text input: "Ask your assistant anything..."
- Buttons: Attach file, Voice input, Send
- Above input: Suggestion chips — "Check my calendar", "Summarize unread emails", "Show overdue tasks"

Context panel (right, 320px):
- "Conversation Context" header
- Referenced connectors with sync status
- Referenced entities: People, projects, deals mentioned in conversation
- Memory items recalled for this conversation
- Action history: Actions executed during this conversation with status
- "Export Conversation" button

States: New conversation with example prompts, multi-turn conversation with various content types, voice input active, assistant error with retry, connector unavailable warning, memory recall indicator.

Include dark mode variant.
```

#### Prompt F-005: Connector Hub Management

```
Design the Connector Hub management page for ERP-Assistant at 1440px width.

Layout:
- Global top bar with breadcrumb "ERP-Assistant > Connectors"
- Left sidebar with "Connectors" active
- Main content area

Content sections:

1) Header:
   - Title "Connector Hub" with subtitle "Connect your tools to give your assistant superpowers"
   - Actions: "Add Connector" primary button
   - Filter: Category (Productivity, Communication, Project Management, CRM, Storage, Custom), Status (All, Connected, Disconnected, Error)

2) Connected services (top section):
   - Section header: "Active Connections" with count badge "7 connected"
   - Grid of connected connector cards (3 columns):
     Each card:
     - Service logo (large, recognizable): Google Workspace, Microsoft 365, Slack, Jira, Notion, Trello, Asana
     - Service name and account: "Google Workspace — sarah@company.com"
     - Status: Green "Connected" badge with last sync time "Synced 2 min ago"
     - Data scope tags: "Calendar", "Gmail", "Drive", "Contacts"
     - Sync indicator: Animated if actively syncing
     - Actions: "Configure" (gear icon), "Disconnect" (with confirmation), "Force Sync" (refresh icon)
     - Health mini-chart: Sync success rate sparkline over 7 days

   Sample connected services:
   - Google Workspace (Calendar, Gmail, Drive, Contacts) — Connected, synced 2 min ago
   - Slack (Messages, Channels, Files) — Connected, synced 5 min ago
   - Jira (Projects, Issues, Sprints) — Connected, synced 1 min ago
   - Microsoft 365 (Outlook, Teams, OneDrive) — Connected, synced 10 min ago
   - Notion (Pages, Databases) — Connected, syncing now
   - WhatsApp Business — Connected, real-time
   - GitHub (Repos, PRs, Issues) — Error, last sync failed 30 min ago

3) Available connectors (bottom section):
   - Section header: "Available Connectors" with search input
   - Grid of available connector cards (4 columns, smaller cards):
     Each: Service logo, name, category badge, "Connect" button, brief description
   - Categories with dividers:
     Productivity: Trello, Asana, Monday.com, ClickUp, Basecamp
     Communication: Telegram, Discord, Microsoft Teams (standalone), Zoom
     CRM: Salesforce, HubSpot, Pipedrive
     Storage: Dropbox, Box, OneDrive (standalone), AWS S3
     Custom: "Build Custom Connector" card with API documentation link

4) Connector detail panel (slide-over, 520px, when card clicked):
   - Service header: Logo, name, connection status, account info
   - Tabs: Overview | Configuration | Sync History | Permissions
   - Overview tab: What data this connector provides, last 5 sync events, health chart
   - Configuration tab:
     - Sync frequency: Real-time, Every 5 min, Every 15 min, Every hour, Manual
     - Data scope toggles: Which data types to sync (e.g., Calendar events: On, Email: On, Drive files: Off)
     - Filter rules: "Only sync emails from last 30 days", "Only sync starred calendar events"
     - Webhook URL for real-time connectors
   - Sync History tab: Table of sync events (timestamp, direction, records synced, status, duration)
   - Permissions tab: OAuth scopes granted, "Re-authorize" button, data deletion options

5) Connection wizard (modal, triggered by "Add Connector"):
   - Step 1: Select service (search + grid of service logos)
   - Step 2: Authorize (OAuth redirect with explanation of requested permissions)
   - Step 3: Configure data scope (toggle which data types to sync)
   - Step 4: Test connection (automated test with success/failure result)
   - Step 5: Activate ("Your assistant can now access your Google Calendar!")

States: No connectors (onboarding wizard), all healthy, connector error (red status with retry), OAuth expired (re-authorization prompt), sync in progress, rate limit warning from external service.

Include dark mode variant.
```

#### Prompt F-006: Memory Explorer

```
Design the Memory Explorer page for ERP-Assistant at 1440px width.

Layout:
- Global top bar with breadcrumb "ERP-Assistant > Memory"
- Left sidebar with "Memory" active
- Main content area: Two-column layout — Memory timeline (65%) | Memory details (35%)

Content sections:

1) Header:
   - Title "Memory" with subtitle "What your assistant remembers about you and your work"
   - Search input: "Search memories..." with natural language support
   - Actions: "Add Memory Manually" secondary button, "Memory Settings" gear icon
   - Stats row: 1,247 memories stored, 23 recalled today, 6 categories

2) Filter and category bar:
   - Category chips: All, Facts (brain icon), Preferences (heart icon), Contacts (people icon), Projects (folder icon), Decisions (gavel icon), Dates (calendar icon)
   - Source filter: All Sources, User-Stated, Inferred from Chat, From Connectors, Manually Added
   - Time filter: All Time, Last 7 Days, Last 30 Days, Last 90 Days
   - Confidence filter: All, High (>90%), Medium (60-90%), Low (<60%)

3) Memory timeline (left, 65%):
   - Vertical timeline with date headers
   - Memory cards grouped by date:

   Today, February 23, 2026:
   - [Preference] "Prefers morning meetings before 11 AM for deep work in afternoons" — Source: Inferred from calendar patterns — Confidence: 92% — Recalled 3 times
   - [Fact] "Q4 budget variance is 3% over target" — Source: Google Sheets sync — Confidence: 98% — Recalled today in briefing
   - [Contact] "John Miller at Acme Corp: key decision maker, prefers email over calls" — Source: Chat conversation — Confidence: 87% — Recalled 5 times

   February 22, 2026:
   - [Decision] "Approved marketing campaign with $45K budget, pending CFO sign-off" — Source: Slack #approvals — Confidence: 95%
   - [Project] "Sprint 24 goal: Complete checkout redesign by Feb 28" — Source: Jira sprint planning — Confidence: 99%
   - [Preference] "Uses dark mode in all applications" — Source: User-stated — Confidence: 100%

   February 21, 2026:
   - [Fact] "Company all-hands meeting moved to every other Thursday" — Source: Google Calendar — Confidence: 94%
   - [Contact] "Sarah Chen: Marketing Director, reports to VP of Growth, working on Q1 campaign" — Source: Inferred from email + Slack — Confidence: 81%

   Each memory card:
   - Type icon and color-coded left bar
   - Content text (2-3 lines max, expand on click)
   - Source badge with connector icon
   - Confidence percentage with visual indicator (green/amber/gray)
   - "Recalled N times" usage indicator
   - Quick actions: Edit, Forget (delete), Pin, Share with assistant context

4) Memory detail panel (right, 35%):
   - Shown when a memory is selected
   - Full memory content (no truncation)
   - Metadata:
     - Type, category, source, confidence score
     - Created date, last recalled date, recall count
     - Related memories (linked by topic/entity)
   - Provenance trail: "How did I learn this?"
     - "Inferred from 3 calendar events where you declined afternoon meetings"
     - Links to source conversations/events
   - Edit form: Modify content, adjust category, override confidence
   - "Forget This" button with confirmation dialog explaining implications
   - "Related Memories" section: Other memories about the same entity/topic

5) Memory insights (collapsible bottom panel):
   - "Memory Health" summary: Distribution by category (pie chart), confidence distribution, growth trend
   - "Stale Memories" alert: "12 memories haven't been recalled in 90+ days — review?"
   - "Low Confidence" alert: "7 memories below 60% confidence — verify or remove?"

States: Empty memory (first-use onboarding with explanation of how memory works), rich memory timeline, search results with highlighted matches, memory conflict (two contradictory memories flagged), bulk memory management mode, memory export/download.

Include dark mode variant.
```

#### Prompt F-007: Action Engine Dashboard

```
Design the Action Engine dashboard page for ERP-Assistant at 1440px width.

Layout:
- Global top bar with breadcrumb "ERP-Assistant > Actions"
- Left sidebar with "Actions" active
- Main content area

Content sections:

1) Header:
   - Title "Action Engine" with subtitle "Automate tasks across your connected services"
   - Actions: "Create Custom Action" primary button, "Browse Templates" secondary
   - Search: "Search actions..." input

2) Quick action bar (horizontal scrollable):
   - Frequently used actions as pill buttons: "Schedule Meeting", "Send Email", "Create Jira Ticket", "Post to Slack", "Set Reminder", "Create Document"
   - Each pill: Icon + label, tap to open execution form

3) Action metrics (4 KPI tiles):
   - Actions Today: 23 (trend +5 vs yesterday)
   - Success Rate: 97.8%
   - Time Saved: 2.4 hours (estimated)
   - Most Used: "Schedule Meeting" (8 times today)

4) Available actions grid (3 columns):
   Grouped by connector:

   Google Workspace:
   - "Schedule Meeting" — Create calendar event with smart time suggestions
   - "Send Email" — Compose and send via Gmail with AI drafting
   - "Create Document" — Generate Google Doc from template or prompt
   - "Share File" — Share Drive file with specified permissions

   Slack:
   - "Post Message" — Send to channel or DM
   - "Set Status" — Update Slack status with duration
   - "Create Channel" — Set up new channel with description and members

   Jira:
   - "Create Issue" — New ticket with AI-suggested fields
   - "Update Issue Status" — Transition issue through workflow
   - "Log Time" — Record work hours on issue
   - "Add Comment" — Comment on issue with context

   Composite Actions (multi-service):
   - "Schedule & Notify" — Create calendar event + send Slack notification + email invite
   - "Standup Summary" — Pull Jira updates + create Slack post + schedule follow-up
   - "Meeting Prep" — Pull agenda from calendar + gather related docs + create summary

   Each action card: Service icon, action name, description, required connector badge, "Execute" button, execution count

5) Recent action history (bottom table):
   Columns: Timestamp, Action, Service(s), Parameters Summary, Status, Duration, Initiated By (Chat/Voice/Schedule/Manual)
   Sample rows:
   - "10:15 AM | Schedule Meeting | Google Calendar | 'Budget Review, Feb 24, 10-11 AM, 5 attendees' | Success | 1.2s | Voice"
   - "09:42 AM | Create Issue | Jira | 'Bug: Checkout page timeout, Priority: High' | Success | 0.8s | Chat"
   - "09:30 AM | Send Email | Gmail | 'Re: Acme Corp Proposal to john@acme.com' | Success | 1.5s | Chat"
   - "09:15 AM | Post Message | Slack | '#standup: Sprint 24 update...' | Success | 0.3s | Scheduled"
   - "08:00 AM | Standup Summary | Jira+Slack | 'Daily standup for Team Alpha' | Failed | 3.2s | Scheduled"

6) Action execution modal (when "Execute" clicked):
   - Action name and service icon header
   - Dynamic form based on action schema:
     - For "Schedule Meeting": Title, Date/Time (with AI suggestion), Duration, Attendees (auto-complete from memory), Location/Link, Description
   - AI assistance: "Based on attendee availability, I suggest Tuesday 2-3 PM" with explanation
   - Memory-assisted autofill: Fields pre-populated from recent conversations and memory
   - "Execute" primary button, "Execute & Remember" (saves preferences), "Cancel"
   - Progress indicator during execution
   - Result display: Success confirmation or error with retry

States: No actions available (connect services first), action execution in progress, action failed with retry/alternative, action queue (multiple pending), undo confirmation.

Include dark mode variant.
```

#### Prompt F-008: Voice Interface

```
Design the Voice interaction interface for ERP-Assistant at 1440px width.

Layout:
- This is a modal overlay that appears on top of any current page when voice is activated
- Semi-transparent dark backdrop

Voice activation states:

1) Resting state (voice button in top bar):
   - Microphone icon, outlined, teal color
   - Tooltip: "Click or press Ctrl+Space to talk"

2) Listening state (full overlay):
   - Centered circular voice indicator (160px diameter):
     - Outer ring: Pulsing cyan animation (expanding/contracting rings)
     - Inner circle: Filled teal with microphone icon
     - Waveform visualization around the circle (real-time audio bars)
   - Below circle: "Listening..." text with animated dots
   - Real-time transcription: Large text appearing character by character below the waveform
     - Example: "Schedule a meeting with John from Acme for Tuesday at..."
   - Cancel button: "Tap anywhere or press Escape to cancel"
   - Language indicator: "EN" badge (supports multiple languages)

3) Processing state:
   - Circle shrinks to processing spinner
   - Transcription finalized: "Schedule a meeting with John from Acme for Tuesday at 2 PM"
   - "Processing your request..." text with progress indicator
   - Below: "I heard:" confirmation with "Edit" link to correct transcription

4) Response state:
   - Assistant response displayed as a card below the transcription:
     - "I'll schedule a meeting with John Miller (Acme Corp) for Tuesday, February 25 at 2:00 PM"
     - Confirmation details: Duration (30 min default), Location (Zoom link), Attendees
     - Action buttons: "Confirm" (primary, teal), "Modify" (secondary), "Cancel"
   - Voice readback: Assistant reads response aloud, with playback control (pause, replay)
   - Visual indicator: Sound wave animation from assistant avatar

5) Confirmation state:
   - Success checkmark animation
   - "Done! Meeting scheduled for Tuesday at 2 PM"
   - "Anything else?" prompt
   - Auto-dismiss after 3 seconds if no response

Voice command examples (shown during first-use onboarding):
- "What's on my calendar today?"
- "Send a Slack message to the team about the deadline change"
- "Create a Jira ticket for the login bug, high priority"
- "What did John from Acme say in his last email?"
- "Remind me to follow up with Sarah on Friday"

Continuous conversation mode:
- After response, assistant stays in listening mode for follow-up
- Conversation history scrolls above, showing previous exchanges
- "End conversation" button or "Goodbye" voice command to close

Accessibility:
- All voice interactions have full visual text equivalents
- Captions for assistant speech
- Visual-only mode toggle for users who prefer not to use voice

States: Microphone permission request, no microphone detected error, poor audio quality warning, transcription confidence low (amber highlight on uncertain words), voice recognition error with text input fallback, network offline (voice unavailable, text only).

Include dark mode variant.
```

### 3.3 Mobile Pages (390px)

#### Prompt F-009: Mobile Assistant Home

```
Design the Assistant Home and Daily Briefing page for ERP-Assistant at 390px mobile width.

Layout:
- Sticky top bar: Hamburger menu (left), "Assistant" title (center), voice button (right)
- Bottom navigation bar: Home (active), Chat, Actions, Connectors, More

Content:

1) Greeting card (full width):
   - "Good morning, Sarah" with date
   - Quick stats row (horizontal scrollable): 6 meetings, 14 emails, 3 approvals, 2 overdue
   - "Start My Day" button (full width, teal)

2) Briefing summary card:
   - AI-generated 2-3 sentence summary
   - "Listen" button for voice readback (with headphones icon)
   - Expand for full briefing

3) Briefing sections (stacked, full width):
   a) Today's Schedule: Compact timeline with time, title, location; tap to expand
   b) Pending Actions: Priority-sorted cards with "Take Action" button
   c) Key Emails: Sender, subject, AI summary, swipe actions (reply, archive, snooze)
   d) Task Updates: Sprint progress bar, overdue items highlighted

4) Suggestion cards (horizontal scrollable):
   - AI suggestions with "Do it" quick action
   - Swipe to dismiss

5) Floating voice button (bottom-right, above nav bar):
   - Large (56px) circular teal button with microphone
   - Pulse animation when assistant has proactive suggestions

Pull-to-refresh for live briefing update.

States: Loading skeleton, no connectors (setup CTA), all caught up, offline mode with cached briefing, partial connector sync error.

Touch targets: All interactive elements minimum 44x44px. Cards have 16px horizontal padding.
```

#### Prompt F-010: Mobile Chat Interface

```
Design the Chat interface for ERP-Assistant at 390px mobile width.

Layout:
- Sticky top bar: Back arrow (to conversation list), conversation title (truncated), voice button, three-dot menu
- Full-screen chat thread
- Sticky bottom input bar with voice toggle

Chat thread:
- User messages: Right-aligned, teal, max width 85%
- Assistant responses: Left-aligned, white with teal border, max width 90%
  - Source badges: Compact icons (Google, Slack, Jira) with tooltip on tap
  - Inline data cards: Tap to expand (calendar event, Jira ticket, email preview)
  - Action buttons: Stack vertically, full card width
  - Tables: Horizontally scrollable
- Typing indicator with animated dots

Input bar (sticky bottom):
- Text input expanding up to 3 lines
- Attach button (left)
- Voice button (right, switch between typing and voice mode)
- Send button (appears when text entered, replacing voice button)
- Above input: Horizontal scrollable suggestion chips

Conversation list (separate screen, accessible via back arrow):
- Search input at top
- Pinned conversations section
- All conversations: Title, preview, timestamp, source icons
- Swipe left: Delete, Archive
- Swipe right: Pin
- "New Conversation" floating action button

Voice mode in chat:
- Tap voice button: Input bar transforms to waveform + transcription
- Real-time text appearing in input area
- "Cancel" (left) and "Send" (right) buttons flanking waveform

States: Empty conversation list, new conversation with example prompts, long conversation with scroll, voice input active, error with retry, offline queued messages indicator.
```

#### Prompt F-011: Mobile Connector Management

```
Design the Connector management page for ERP-Assistant at 390px mobile width.

Layout:
- Sticky top bar: Hamburger menu, "Connectors" title, "+" button
- Bottom navigation bar: Home, Chat, Actions, Connectors (active), More

Content:

1) Connected services (section header with count):
   - Vertical list of connected service cards (full width):
     Each card:
     - Service logo (left, 40px), name and account (center), status badge (right)
     - Below name: Last sync time, data scope tags (compact)
     - Tap to expand: Sync history, configure, disconnect actions
     - Swipe left: Quick disconnect (with confirmation)

   Sample:
   - Google Workspace — sarah@company.com — Connected, 2m ago
   - Slack — company.slack.com — Connected, 5m ago
   - Jira — company.atlassian.net — Connected, 1m ago
   - GitHub — @sarah-dev — Error, sync failed 30m ago (red highlight)

2) Available connectors (section):
   - Section header: "Add More Connectors"
   - Search input
   - Compact grid (2 columns): Service logo + name, "Connect" button
   - Category tabs (horizontal scroll): All, Productivity, Communication, CRM, Storage

3) Connector detail (separate screen, slide from right):
   - Large service logo and name
   - Connection status with detail
   - Tabs (horizontal scroll): Overview, Config, Sync History, Permissions
   - All tab content stacked vertically, full width

4) Connection wizard (full-screen modal):
   - Step indicator at top
   - One step per screen, "Next" and "Back" buttons
   - OAuth step: Full-screen webview for authorization

States: No connectors (illustrated onboarding), adding first connector, sync in progress, connector error with troubleshooting, OAuth expired re-auth.
```

#### Prompt F-012: Mobile Voice Assistant

```
Design the Voice assistant full-screen interface for ERP-Assistant at 390px mobile width.

Layout:
- Full-screen overlay (no navigation bars visible)
- Activated from voice button anywhere in the app

Content:

1) Top area:
   - Close button (X, top-left)
   - "Voice Assistant" label (center)
   - Language indicator "EN" (top-right)

2) Center area (listening state):
   - Large voice indicator circle (120px):
     - Pulsing cyan rings animation
     - Microphone icon center
   - Real-time waveform bars (horizontal, full width, below circle)
   - "Listening..." animated text

3) Transcription area (below center):
   - Large text (18px) showing real-time transcription
   - Full width with 24px padding
   - Scrollable if long command

4) Response area (after processing):
   - Assistant response card (full width, 16px margins):
     - Response text with formatting
     - Source badges
     - Action buttons (stacked, full width)
   - Voice readback controls: Play/Pause, speed selector

5) Bottom area:
   - "Tap to speak again" hint
   - Text input fallback: "Or type here..." text field
   - Conversation history (scroll up to see previous voice exchanges)

6) Quick commands (first-use overlay):
   - Grid of example commands: "My schedule", "Unread emails", "Create meeting", "Send message"
   - Tap to execute

Haptic feedback notes:
- Vibrate briefly when entering listening state
- Vibrate on successful action completion
- Double vibrate on error

States: Microphone permission dialog, listening (waveform active), processing (spinner), response with actions, continuous conversation (multiple exchanges), error (recognition failed — "I didn't catch that, try again"), offline (voice unavailable).
```

### 3.4 Tablet/Responsive (1024px)

```
Tablet adaptations for ERP-Assistant at 1024px:

General rules:
- Left sidebar collapses to icon-only mode (56px) by default; expand on hover or tap
- Content area uses full remaining width
- Bottom navigation bar: Hidden; all navigation via sidebar

Specific adaptations:
- Home/Briefing: Briefing sections switch to single column; schedule timeline and email list stack vertically; quick stats remain in a 4-across row
- Chat: Two-column layout — Chat thread (65%) + Context panel (35%); Conversation list becomes a slide-out drawer triggered by header button
- Connectors: Connected services grid becomes 2 columns; available connectors grid becomes 3 columns; detail panel is a slide-over at 480px width
- Memory: Timeline takes full width; detail panel becomes a slide-over from right at 400px width; memory cards remain full fidelity
- Actions: Action grid becomes 2 columns; action metrics tiles arrange 2x2; execution modal is centered at 560px max width
- Voice overlay: Same centered design as desktop but circle indicator scales to 140px; transcription text uses 20px font; action buttons full width
- Tables: Horizontally scrollable with sticky first column
- KPI tiles: 2x2 grid instead of 4-across
- Modals: Max width 600px, centered with backdrop
- Command palette: Full width minus 48px padding on each side
```

---

## 4. Make Automation Prompts

### Prompt M-001: Morning Briefing Generator

```
Build a Make scenario for automated morning briefing generation in ERP-Assistant:

Trigger: Schedule — Daily at 06:00 AM local time per user (use timezone from user profile)

Steps:
1. HTTP GET /v1/briefing/users/active — Get list of users with briefing enabled.
2. Iterator: For each user:
   a. HTTP GET /v1/briefing/data?user_id={user_id}&date=today
      - Fetches aggregated data from all connected services via connector-hub:
        - Calendar events for today (Google Calendar / Outlook)
        - Unread emails prioritized by AI (Gmail / Outlook)
        - Pending tasks and overdue items (Jira / Asana / Trello)
        - Unread Slack messages in priority channels
        - Pending approvals from any workflow
   b. HTTP POST /v1/copilot/generate with prompt:
      "Generate a concise morning briefing for {user_name}. Prioritize by urgency and relevance. Include: schedule overview, top 3 emails requiring attention, overdue tasks, pending approvals. Tone: professional and helpful. Max 200 words for summary."
   c. HTTP POST /v1/briefing/{user_id} — Store generated briefing.
   d. HTTP POST /v1/briefing/{user_id}/notify — Trigger push notification and email:
      Subject: "Your Daily Briefing — {date}"
      Body: HTML-formatted briefing with deep links to each item
   e. If user has voice_briefing_enabled: Queue voice synthesis via /v1/voice/synthesize with briefing text.

3. Aggregate: Count successful briefings, failed briefings, total items surfaced.
4. Log to Google Sheets "Briefing Generation Log": timestamp, users_processed, success_count, failure_count, avg_generation_time.
5. If failure_count > 0: Send Slack alert to #assistant-ops with failed user IDs and error details.

Guardrails: Timeout per user: 30 seconds. Skip users with no connected services. Respect user's do-not-disturb settings. Deduplicate: Only generate one briefing per user per day. Retry failed users once after 15-minute delay.
```

### Prompt M-002: Connector Sync Health Monitor

```
Build a Make scenario for monitoring connector sync health in ERP-Assistant:

Trigger: Schedule — Every 10 minutes

Steps:
1. HTTP GET /v1/connector-hub/connections?status=active
   Response: Array of active connections with { "connection_id", "user_id", "service", "last_sync", "sync_status", "error_count_24h", "data_freshness_seconds" }

2. For each connection, evaluate:
   a. If sync_status == "error": Flag as failed connection.
   b. If data_freshness_seconds > 900 (15 min) for real-time connectors: Flag as stale.
   c. If error_count_24h > 10: Flag as chronically failing.

3. For failed connections:
   - HTTP POST /v1/connector-hub/connections/{connection_id}/retry — Attempt re-sync.
   - If retry fails:
     - Send push notification to user: "Your {service} connection needs attention. Last sync failed. [Reconnect]"
     - If error suggests OAuth expired: Direct user to re-authorization deep link.
   - Log failure to Google Sheets "Connector Health Log".

4. For stale connections:
   - HTTP POST /v1/connector-hub/connections/{connection_id}/sync — Force sync.
   - If sync takes > 60 seconds: Log as slow sync.

5. For chronically failing connections:
   - Send Slack alert to #assistant-ops: "Connection {service} for user {user_id} has {error_count_24h} errors in 24h."
   - Create support ticket if error_count_24h > 50.

6. Aggregate health summary:
   - Total connections, healthy count, stale count, error count.
   - Post to #assistant-daily at 09:00 UTC.

Guardrails: Rate limit re-sync attempts to 1 per connection per 30 minutes. Never expose user credentials in logs or alerts. Respect API rate limits of external services (Google: 100 req/min, Slack: 50 req/min). Idempotent retry by connection_id + timestamp window.
```

### Prompt M-003: Memory Lifecycle Manager

```
Build a Make scenario for managing memory lifecycle in ERP-Assistant:

Trigger: Schedule — Daily at 02:00 UTC (low-traffic window)

Steps:
1. HTTP GET /v1/memory/stale?days=90
   - Returns memories not recalled in 90+ days.
   Response: Array of { "memory_id", "user_id", "content_preview", "type", "confidence", "created_at", "last_recalled", "recall_count" }

2. For each stale memory:
   a. If recall_count == 0 AND confidence < 70%:
      - Auto-archive: POST /v1/memory/{memory_id}/archive with reason "Never recalled, low confidence".
   b. If recall_count > 0 AND last_recalled > 180 days:
      - Flag for user review: POST /v1/memory/{memory_id}/flag with reason "Not recalled in 6 months".
   c. All other stale memories:
      - Reduce confidence by 10%: PATCH /v1/memory/{memory_id} with confidence_delta: -0.10.

3. HTTP GET /v1/memory/conflicts
   - Returns pairs of contradictory memories.
   - For each conflict pair:
     - If one memory has significantly higher confidence (>20% difference): Keep higher, archive lower.
     - Otherwise: Flag both for user review with "Please confirm which is correct" notification.

4. HTTP GET /v1/memory/duplicates
   - Returns near-duplicate memory pairs (>90% semantic similarity).
   - For each duplicate pair:
     - Merge into single memory with combined context: POST /v1/memory/merge.
     - Retain the memory with higher recall_count as primary.

5. Generate memory health report:
   - Total memories per category, stale count, archived count, conflicts resolved, duplicates merged.
   - Send to user (if opted-in) as weekly digest: "Your assistant's memory health: 1,247 memories, 12 archived this week, 3 conflicts need your review."

6. Log operations to Google Sheets "Memory Lifecycle Log": Date, user_id, memories_reviewed, archived, flagged, merged, confidence_adjusted.

Guardrails: Never auto-delete memories — only archive (recoverable for 30 days). User can override any automated decision. Respect "pinned" memories — never archive or reduce confidence. Log all operations for audit. Process maximum 500 memories per user per run to avoid timeouts.
```

### Prompt M-004: Action Failure Recovery and Learning

```
Build a Make scenario for handling action execution failures in ERP-Assistant:

Trigger: Webhook — receives event from Redpanda topic "erp.assistant.action.failed"
Payload: { "action_id": "act_schedule_meeting", "action_name": "Schedule Meeting", "user_id": "user_sarah", "service": "google_calendar", "error_type": "auth_expired", "error_message": "OAuth token expired", "parameters": { "title": "Budget Review", "date": "2026-02-25", "attendees": ["john@acme.com"] }, "attempt_count": 1, "timestamp": "2026-02-23T10:15:00Z" }

Steps:
1. Parse payload and classify error_type:
   a. "auth_expired" / "permission_denied":
      - Send push notification to user: "Your {service} connection needs re-authorization to complete '{action_name}'. [Fix Now]"
      - Queue action for retry after re-authorization: POST /v1/action-engine/queue with retry_after_auth flag.
      - Update connector status: PATCH /v1/connector-hub/connections/{service} with status "auth_required".

   b. "rate_limited":
      - Calculate backoff: next_retry = timestamp + (attempt_count * 60 seconds).
      - Queue action for delayed retry: POST /v1/action-engine/queue with scheduled_at = next_retry.
      - If attempt_count >= 3: Notify user "Action delayed due to {service} rate limits. Will retry automatically."

   c. "service_unavailable":
      - Check service health: GET /v1/connector-hub/health/{service}.
      - If service is down: Notify user "{service} is currently unavailable. Your action '{action_name}' has been queued and will execute when service recovers."
      - Queue action with auto-retry on service recovery event.

   d. "validation_error":
      - Notify user with specific fix: "Your action '{action_name}' failed: {error_message}. Would you like to fix the parameters and retry?"
      - Include deep link to action form pre-populated with original parameters.

   e. "unknown":
      - Log full error to Google Sheets "Action Error Log".
      - If same action + error_type occurs 5+ times in 24h: Create ops alert in Slack #assistant-alerts.
      - Notify user with generic retry option.

2. Store failure for pattern analysis: POST /v1/memory/{user_id} with type "action_failure_context" for learning.

3. If user retries and succeeds: POST /v1/memory/{user_id} with corrective pattern for future error prevention.

Guardrails: Maximum 5 retry attempts per action. Never retry destructive actions (delete, revoke) without user confirmation. Do not expose raw error messages containing credentials or tokens to users. Rate limit user notifications to 1 per action per 10 minutes.
```

### Prompt M-005: Proactive Assistant Suggestions

```
Build a Make scenario for generating proactive assistant suggestions in ERP-Assistant:

Trigger: Schedule — Every 30 minutes during user's active hours (08:00-18:00 local time)

Steps:
1. HTTP GET /v1/briefing/users/active?online=true — Get users currently active.

2. For each active user:
   a. HTTP GET /v1/memory/{user_id}/context?window=4h — Get recent activity context.
   b. HTTP GET /v1/connector-hub/changes/{user_id}?since=30m — Get recent changes across connectors.
      Examples: New email from VIP contact, calendar change, Jira status update, Slack mention.

   c. Evaluate suggestion triggers:
      - Upcoming meeting in 15 min with no prep: Suggest "You have 'Budget Review' in 15 min. Want me to pull the latest budget report?"
      - Unresponded email from VIP older than 4 hours: Suggest "You haven't replied to CFO's email about Q4 budget. Would you like me to draft a response?"
      - Task approaching deadline: Suggest "Sprint item 'Checkout Redesign' is due tomorrow and 40% complete. Want to reschedule or request help?"
      - Calendar conflict detected: Suggest "You have overlapping meetings at 2 PM. Want me to reschedule the less critical one?"
      - Follow-up due (from memory): Suggest "You planned to follow up with John at Acme today. Want me to send a check-in email?"

   d. For each suggestion:
      - Score relevance (0-100) based on urgency, recency, user response history.
      - If score > 70: POST /v1/assistant-core/{user_id}/suggestions with suggestion text, action, and confidence.
      - Limit to max 3 suggestions per 30-minute window.

3. Track suggestion acceptance/dismissal rates per user:
   - HTTP GET /v1/assistant-core/{user_id}/suggestion-metrics
   - If dismissal_rate > 80% for a suggestion type: Reduce frequency for that type.
   - If acceptance_rate > 60% for a pattern: Increase proactivity for that pattern.

4. Weekly: Aggregate suggestion metrics, send to #assistant-product for UX optimization.

Guardrails: Respect user's "Do Not Disturb" and "Focus Mode" settings. Maximum 3 suggestions per 30-minute window. Never repeat a dismissed suggestion within 24 hours. No suggestions for sensitive topics (HR issues, personal health, salary) unless explicitly opted-in. User can disable proactive suggestions entirely.
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
8. **Voice interaction screens** require extra care: ensure all voice states have visual text equivalents and the transitions between states are clearly annotated.

### 5.2 Make Automation Best Practices
1. **Set up webhook endpoints** in your Make environment before deploying scenarios.
2. **Use environment variables** for API keys, Slack tokens, and endpoint URLs — never hardcode credentials.
3. **Test with sample payloads** provided in each prompt before connecting to live Redpanda topics.
4. **Configure error handling** on every HTTP module: Retry 3x with exponential backoff (30s, 60s, 120s).
5. **Set up a dead-letter queue** scenario to capture events that fail all retries.
6. **Rate limit external notifications** (Slack, push, email) as specified in each scenario's guardrails section.
7. **Respect external API rate limits**: Google (100 req/min), Slack (50 req/min), Jira (100 req/min).
8. **Monitor Make scenario execution logs** weekly for silent failures or quota exhaustion.
9. **User timezone awareness**: All scheduled scenarios must account for user's local timezone from their profile.

---

## 6. Output Packaging Convention

Request Figma Make outputs under these pages:

```
00_Context — Project brief, personas (knowledge worker, executive, team lead), architecture diagram reference
01_IA_Flows — Information architecture, navigation map, user flow diagrams (briefing flow, chat flow, action flow, voice flow)
02_Design_Tokens — Color, typography, spacing, elevation, motion tokens (light + dark)
03_Components — Full component library with all state variants (chat, briefing, connector, memory, voice, action)
04_Desktop_1440 — All desktop page designs (Home, Chat, Connectors, Memory, Actions, Voice)
05_Tablet_1024 — Tablet-adapted layouts
06_Mobile_390 — All mobile page designs
07_Voice_States — Complete voice interaction state machine (resting, listening, processing, response, error)
08_States_And_Errors — Empty, loading, error, success, offline, degraded, syncing states for every surface
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
| Time to first interaction | <= 1.8s | Key routes (Home, Chat, Actions) |
| JS route budget | <= 220KB gzip | Per initial route payload |
| Briefing generation | <= 5s | End-to-end briefing render from API call |
| Chat response display | Streaming | First token visible < 500ms |
| Voice transcription latency | <= 300ms | From speech end to text display |
| Voice-to-action latency | <= 2s | From voice command to action execution start |
| Connector sync indicator | <= 100ms | Status badge update after sync event |
| Memory search | <= 500ms | Natural language query to results |
| Action execution feedback | <= 1s | From button press to progress indicator |

---

## 8. AIDD Handoff Gate Template

Attach this completed checklist to every generated design before engineering handoff:

```
[ ] 1. Accessibility gate passed
      - Color contrast ratios verified (4.5:1 normal text, 3:1 large text)
      - Focus order annotated for all interactive elements
      - Keyboard navigation paths documented
      - Screen-reader announcements specified for dynamic content (chat messages, briefing updates, connector status, voice transcription)
      - Tap targets >= 44x44px on touch surfaces
      - Voice interactions have complete visual text equivalents
      - ARIA live regions specified for real-time updates (voice transcript, sync status)

[ ] 2. Performance gate passed
      - Route-level lazy loading boundaries defined
      - JS bundle budget per route documented
      - Skeleton/placeholder dimensions match final layout (no CLS)
      - Heavy components (voice waveform, memory timeline, connector config panels) marked for deferred loading
      - Image strategy: responsive srcset, WebP/AVIF, lazy below fold

[ ] 3. Reliability gate passed
      - Error states defined for every API-dependent surface
      - Retry paths available for failed actions, connector syncs, voice commands, chat messages
      - Offline/degraded behavior specified (cached briefing, queued actions, offline memory access)
      - Connector failure graceful degradation defined
      - Voice fallback to text input specified

[ ] 4. Observability gate passed
      - Analytics events mapped: page views, chat interactions, action executions, voice commands, connector usage, memory recalls
      - Error events instrumented with trace ID correlation
      - Performance marks (LCP, INP, CLS) identified per page
      - Voice quality metrics: transcription accuracy, latency, error rate
      - Engagement funnels documented: briefing read-through, action completion, suggestion acceptance

[ ] 5. Testability gate passed
      - State matrix (default, loading, error, empty, success, degraded, syncing) mapped to test cases
      - Component variants documented for visual regression testing
      - API mock requirements specified per page
      - E2E critical paths identified: Daily briefing flow, Chat conversation with action, Voice command to action, Connector setup, Memory recall in context

[ ] 6. Security and privacy gate passed
      - PII handling: User data from connectors displayed only to authorized users
      - Memory privacy: Users can view, edit, and delete any memory
      - Connector credentials: Never displayed after initial setup
      - Tenant isolation: No cross-tenant data leakage in any component
      - Auth: JWT verification required before rendering protected content
      - Voice data: Audio not stored beyond transcription unless user opts in
      - Audit: All actions logged with user, timestamp, and detail
```

---

## 9. Revision History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2026-02-23 | AIDD System | Initial prompt pack covering all 6 ERP-Assistant services: action-engine, assistant-core, briefing-service, connector-hub, memory-service, voice-service |

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
Design a control plane view for ERP-Assistant that shows:
1. Hasura GraphQL connectivity (latency, success %, schema drift status)
2. ERP-DBaaS health (connection pool, replica lag, failover posture)
3. ERP-IAM integration (OIDC issuer health, token validation errors, session invalidations)
4. ERP-Observability status (OTLP export rate, tracing availability, alert health)

Include: command surface, timeline, runbook links, and one-click diagnostics panel.
Add states for: fully healthy, degraded IAM, degraded DB, degraded GraphQL, global outage.
```

### Prompt F-901: Role-Aware Workspace + Guardrail Surface
```
Generate a role-aware workspace for ERP-Assistant with:
- Role lenses: Operator, Manager, Auditor, Admin
- Guardrail panel with mode badge (Protected / Supervised / Autonomous)
- Inline policy rationale for blocked/supervised actions
- Approval request flow for supervised actions

Ensure each role sees distinct navigation and action priorities.
```

### Prompt F-902: Incident + Recovery UX
```
Design a full incident-response flow for ERP-Assistant:
- Alert ingestion to triage board
- Impacted tenant view and blast-radius visualization
- Action timeline with runbook steps and rollback controls
- Post-incident report generator modal

Must include: trace-id copy, audit evidence export, and SLA breach indicators.
```

### Prompt F-903: Mobile-Responsive Executive Snapshot
```
Create mobile and tablet executive dashboards for ERP-Assistant:
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
Create a Make scenario for ERP-Assistant that:
1. Triggers on deployment/webhook events
2. Validates GraphQL, IAM, DB, and OTLP health in sequence
3. Creates incident ticket when thresholds are breached
4. Posts status updates to operations channels
5. Writes audit trail to persistent store

Add branching for: transient failures, policy violations, and hard-stop conditions.
```

### Prompt M-901: Tenant Onboarding Orchestration
```
Generate a Make scenario for tenant onboarding in ERP-Assistant:
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
