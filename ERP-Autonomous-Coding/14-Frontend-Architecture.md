# ERP-Autonomous-Coding -- Frontend Architecture (Dashboard)

## Document Information

| Field | Value |
|-------|-------|
| Module | ERP-Autonomous-Coding |
| Version | 1.0.0 |
| Last Updated | 2026-02-23 |
| Framework | Next.js 14 |

---

## 1. Dashboard Architecture

```mermaid
flowchart TB
    subgraph "Next.js 14 Dashboard"
        subgraph "Pages (App Router)"
            DASH["/ (Dashboard)"]
            SESS["/sessions"]
            SESS_DET["/sessions/[id]"]
            REPOS["/repositories"]
            REVIEWS["/reviews"]
            REVIEWS_DET["/reviews/[id]"]
            SANDBOX["/sandboxes"]
            SETTINGS["/settings"]
            APPROVALS["/approvals"]
        end

        subgraph "Shared Components"
            NAV["Navigation Sidebar"]
            HEADER["Header (Auth, Tenant)"]
            BREADCRUMB["Breadcrumb"]
            TOAST["Toast Notifications"]
        end

        subgraph "Feature Components"
            SESSION_LIST["Session List Table"]
            SESSION_DETAIL["Session Detail View"]
            DIFF_VIEWER["Code Diff Viewer"]
            TRACE_VIEWER["Reasoning Trace"]
            REVIEW_PANEL["Review Findings Panel"]
            REPO_CONNECT["Repository Connection Wizard"]
            SANDBOX_LOGS["Sandbox Log Viewer"]
            APPROVAL_QUEUE["AIDD Approval Queue"]
            METRICS_DASH["Metrics Dashboard"]
        end

        subgraph "Data Layer"
            SSE_CLIENT["SSE Client (EventSource)"]
            API_CLIENT["API Client (fetch)"]
            SWR["SWR (data fetching + caching)"]
            ZUSTAND["Zustand (client state)"]
        end
    end

    subgraph "Backend"
        API["Autonomous Coding API"]
        EVENTS["SSE Event Stream"]
    end

    DASH --> METRICS_DASH
    SESS --> SESSION_LIST
    SESS_DET --> SESSION_DETAIL
    SESS_DET --> DIFF_VIEWER
    SESS_DET --> TRACE_VIEWER
    REVIEWS --> REVIEW_PANEL
    SANDBOX --> SANDBOX_LOGS
    REPOS --> REPO_CONNECT
    APPROVALS --> APPROVAL_QUEUE

    API_CLIENT --> API
    SSE_CLIENT --> EVENTS
    SWR --> API_CLIENT
```

---

## 2. Page Hierarchy

```mermaid
flowchart TB
    ROOT["/ (Layout)"] --> DASH2["/ (Dashboard Home)<br/>KPI cards, recent activity"]
    ROOT --> SESSIONS["sessions/"]
    ROOT --> REPOS2["repositories/"]
    ROOT --> REV["reviews/"]
    ROOT --> SAND["sandboxes/"]
    ROOT --> APPROV["approvals/"]
    ROOT --> SET["settings/"]

    SESSIONS --> SESS_LIST["(list)<br/>Filterable table"]
    SESSIONS --> SESS_ID["[id]/<br/>Session detail"]
    SESS_ID --> SESS_TRACE["trace/<br/>Reasoning trace"]
    SESS_ID --> SESS_DIFF["diff/<br/>Code diff viewer"]

    REPOS2 --> REPO_LIST["(list)<br/>Connected repos"]
    REPOS2 --> REPO_ADD["connect/<br/>Connection wizard"]

    REV --> REV_LIST["(list)<br/>Review history"]
    REV --> REV_ID["[id]/<br/>Review detail"]

    SAND --> SAND_LIST["(list)<br/>Active sandboxes"]
    SAND --> SAND_ID["[id]/logs<br/>Sandbox logs"]

    APPROV --> APPROV_LIST["(list)<br/>Pending approvals"]

    SET --> SET_GEN["general/<br/>General settings"]
    SET --> SET_SAND["sandbox/<br/>Sandbox config"]
    SET --> SET_REV2["review/<br/>Review rules"]
    SET --> SET_TEAM["team/<br/>Team management"]
```

---

## 3. Key UI Components

### 3.1 Dashboard Home (KPI Cards)

```mermaid
flowchart TB
    subgraph "Dashboard Home"
        subgraph "KPI Row"
            K1["Active Sessions<br/>Real-time count"]
            K2["Success Rate<br/>Last 7 days"]
            K3["Avg Cycle Time<br/>PR creation speed"]
            K4["Coverage Delta<br/>Agent contribution"]
            K5["Pending Approvals<br/>AIDD queue"]
        end

        subgraph "Charts Row"
            C1["Sessions Over Time<br/>(Area chart)"]
            C2["Review Score Distribution<br/>(Histogram)"]
        end

        subgraph "Activity Feed"
            A1["Recent Sessions<br/>(live-updating list)"]
            A2["Recent PRs<br/>(status badges)"]
        end
    end
```

### 3.2 Code Diff Viewer

The diff viewer renders agent-generated code changes with syntax highlighting, inline annotations, and approval controls:

- Split or unified diff view toggle
- Syntax-highlighted code (via Shiki or Prism)
- Inline review comments from the Review Engine
- Accept/reject controls per file or per hunk
- Link to full reasoning trace for each change

### 3.3 Reasoning Trace Viewer

The trace viewer presents the agent's step-by-step reasoning as a timeline:

- Expandable step cards showing reasoning text
- Tool invocation details (file reads, writes, terminal commands)
- Token usage per step
- Duration per step
- Error states highlighted in red

---

## 4. Technology Stack

| Layer | Technology | Purpose |
|-------|-----------|---------|
| Framework | Next.js 14 (App Router) | Server/client rendering, routing |
| Language | TypeScript | Type safety |
| Styling | Tailwind CSS + shadcn/ui | Component library |
| State (server) | SWR | Data fetching, caching, revalidation |
| State (client) | Zustand | Client-side global state |
| Real-time | EventSource (SSE) | Live session updates |
| Charts | Recharts | Data visualization |
| Diff Viewer | react-diff-viewer-continued | Code diff rendering |
| Syntax Highlighting | Shiki | Code syntax colors |
| Forms | React Hook Form + Zod | Form validation |
| Tables | TanStack Table | Sortable, filterable tables |
| Icons | Lucide React | Iconography |
| Testing | Vitest + Testing Library | Unit + component tests |
| E2E Testing | Playwright | End-to-end tests |

---

## 5. Data Fetching Pattern

```mermaid
sequenceDiagram
    participant Page as Next.js Page
    participant SWR as SWR Hook
    participant API as API Client
    participant Backend as Backend API
    participant SSE as SSE Stream

    Note over Page: Initial Load (Server)
    Page->>Backend: Server-side fetch (RSC)
    Backend-->>Page: Initial data

    Note over Page: Client Hydration
    Page->>SWR: useSWR("/v1/sessions")
    SWR->>API: fetch with JWT
    API->>Backend: GET /v1/sessions
    Backend-->>API: {sessions: [...]}
    API-->>SWR: Data
    SWR-->>Page: Render

    Note over Page: Real-time Updates
    Page->>SSE: new EventSource("/v1/events")
    SSE->>Page: session.started event
    Page->>SWR: mutate (update cache)
    SWR-->>Page: Re-render
```

---

## 6. Authentication Flow

```mermaid
sequenceDiagram
    participant User
    participant Dash as Dashboard
    participant IAM as ERP-IAM
    participant API as Backend API

    User->>Dash: Navigate to dashboard
    Dash->>Dash: Check session cookie
    alt No session
        Dash->>IAM: Redirect to OIDC login
        IAM->>User: Login form
        User->>IAM: Credentials
        IAM->>Dash: Redirect with auth code
        Dash->>IAM: Exchange code for tokens
        IAM-->>Dash: {access_token, id_token, refresh_token}
        Dash->>Dash: Set HTTP-only session cookie
    end
    Dash->>API: API requests with Bearer token
    API-->>Dash: Authenticated responses
```

---

## 7. Responsive Design

| Breakpoint | Layout | Sidebar | Details |
|-----------|--------|---------|---------|
| Desktop (>= 1280px) | Full | Expanded sidebar | Split panels |
| Tablet (768-1279px) | Compact | Collapsed sidebar (icons) | Stacked panels |
| Mobile (< 768px) | Single column | Hidden (hamburger menu) | Single panel |

---

## 8. Accessibility

| Standard | Implementation |
|----------|---------------|
| WCAG 2.1 AA | Color contrast, keyboard navigation, screen reader support |
| Keyboard Navigation | All interactive elements focusable, logical tab order |
| Screen Reader | ARIA labels, roles, live regions for real-time updates |
| Motion | Reduced motion media query respected |
| Color Blindness | Icons + text labels (not color alone) for status indicators |
