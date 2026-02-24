# ERP-Projects -- Workflow Engine

## Document Control

| Field         | Value                                          |
|---------------|------------------------------------------------|
| Module        | ERP-Projects                                   |
| Version       | 1.0                                            |
| Date          | 2026-02-23                                     |

---

## 1. Workflow Architecture

ERP-Projects implements a state-machine-based workflow engine that governs entity lifecycle transitions, approval processes, and automated actions. Each major entity (Project, Task, Sprint, Timesheet, Invoice) has a defined state machine with guarded transitions.

```mermaid
flowchart TB
    subgraph "Workflow Engine"
        SM["State Machine<br/>Definitions"]
        GUARD["Transition Guards<br/>(Preconditions)"]
        ACT["Side Effects<br/>(Actions on transition)"]
        EVT["Event Publisher<br/>(Notify subscribers)"]
    end

    subgraph "Triggers"
        USER["User Action"]
        SCHED["Scheduled Job"]
        EVENT["Event Handler"]
        AI["AI Recommendation"]
    end

    USER & SCHED & EVENT & AI --> SM
    SM --> GUARD
    GUARD -->|"Pass"| ACT --> EVT
    GUARD -->|"Fail"| ERR["Transition Denied"]
```

---

## 2. Project Lifecycle State Machine

```mermaid
stateDiagram-v2
    [*] --> PLANNING: Create project
    PLANNING --> ACTIVE: Start project
    ACTIVE --> ON_HOLD: Pause
    ON_HOLD --> ACTIVE: Resume
    ACTIVE --> COMPLETED: Complete
    ACTIVE --> CANCELLED: Cancel
    PLANNING --> CANCELLED: Cancel
    COMPLETED --> ARCHIVED: Auto-archive (90 days)
    CANCELLED --> ARCHIVED: Auto-archive (30 days)
    ARCHIVED --> [*]

    note right of PLANNING
        Guards:
        - Name, dates, budget required
        - Owner assigned
    end note

    note right of ACTIVE
        Guards:
        - At least one task exists
        - Resources allocated
        Actions:
        - Start baseline tracking
        - Begin health monitoring
    end note

    note right of COMPLETED
        Guards:
        - All tasks Done or Cancelled
        - Timesheets approved
        Actions:
        - Calculate final metrics
        - Generate completion report
    end note
```

### 2.1 Project Transition Guards

| From      | To        | Guards                                       | Side Effects                          |
|-----------|-----------|----------------------------------------------|---------------------------------------|
| PLANNING  | ACTIVE    | Owner assigned, at least 1 task, budget > 0  | Save initial baseline, start health monitor |
| ACTIVE    | ON_HOLD   | None                                         | Pause all timers, notify team         |
| ON_HOLD   | ACTIVE    | None                                         | Resume timers, notify team            |
| ACTIVE    | COMPLETED | All tasks Done/Cancelled                     | Generate final report, archive timers |
| *         | CANCELLED | User has ADMIN or MANAGER role               | Cancel active timers, notify team     |
| COMPLETED | ARCHIVED  | 90 days elapsed (auto)                       | Move to cold storage                  |
| CANCELLED | ARCHIVED  | 30 days elapsed (auto)                       | Move to cold storage                  |

---

## 3. Task Lifecycle State Machine

```mermaid
stateDiagram-v2
    [*] --> TODO: Create task
    TODO --> IN_PROGRESS: Start work
    IN_PROGRESS --> IN_REVIEW: Submit for review
    IN_PROGRESS --> BLOCKED: Blocker detected
    IN_REVIEW --> IN_PROGRESS: Changes requested
    IN_REVIEW --> DONE: Approved
    BLOCKED --> IN_PROGRESS: Blocker resolved
    TODO --> DONE: Skip (trivial tasks)
    IN_PROGRESS --> DONE: Complete (no review needed)

    note right of DONE
        Guards:
        - No open blocking dependencies
        - Checklist items completed (if any)
        Actions:
        - Update parent task completion %
        - Recalculate project completion %
        - Notify assignees and watchers
    end note
```

### 3.1 Task Transition Guards

| From        | To           | Guards                                    | Side Effects                       |
|-------------|-------------|-------------------------------------------|------------------------------------|
| TODO        | IN_PROGRESS | All FS predecessors Done                  | Start timer if auto-track enabled  |
| IN_PROGRESS | IN_REVIEW   | At least one reviewer assigned            | Notify reviewers                   |
| IN_REVIEW   | DONE        | Reviewer approves                         | Stop timer, update project metrics |
| *           | BLOCKED     | Blocking dependency identified            | Notify PM, pause timer             |
| BLOCKED     | IN_PROGRESS | Blocker resolved                          | Resume timer, notify assignee      |
| *           | DONE        | No open blockers, checklist 100% (if any) | Cascade: unblock dependent tasks   |

---

## 4. Sprint Lifecycle State Machine

```mermaid
stateDiagram-v2
    [*] --> PLANNED: Create sprint
    PLANNED --> ACTIVE: Start sprint
    ACTIVE --> REVIEW: Sprint end date
    REVIEW --> COMPLETED: Review done
    COMPLETED --> [*]

    note right of PLANNED
        Guards:
        - Start/end dates set
        - At least 1 story in sprint
    end note

    note right of ACTIVE
        Actions:
        - Lock sprint scope
        - Begin burndown tracking
    end note

    note right of COMPLETED
        Actions:
        - Calculate velocity
        - Return incomplete stories to backlog
        - Prompt retrospective creation
    end note
```

---

## 5. Timesheet Approval Workflow

```mermaid
stateDiagram-v2
    [*] --> DRAFT: Enter time
    DRAFT --> SUBMITTED: Submit
    SUBMITTED --> APPROVED: Manager approves
    SUBMITTED --> REJECTED: Manager rejects
    REJECTED --> DRAFT: Revise entries
    APPROVED --> SYNCED: Payroll sync complete
    SYNCED --> [*]

    note right of SUBMITTED
        Guards:
        - Total hours within limits
        - All entries have project/task
        Actions:
        - Lock entries from editing
        - Notify manager
    end note

    note right of APPROVED
        Guards:
        - Manager has approval permission
        Actions:
        - Mark entries as approved
        - Update project cost tracking
        - Queue for ERP-HCM sync
    end note
```

---

## 6. Invoice Lifecycle

```mermaid
stateDiagram-v2
    [*] --> DRAFT: Create invoice
    DRAFT --> SENT: Send to client
    SENT --> PAID: Payment received
    SENT --> OVERDUE: Past due date
    OVERDUE --> PAID: Payment received
    DRAFT --> CANCELLED: Cancel
    SENT --> CANCELLED: Cancel

    note right of SENT
        Actions:
        - Email invoice to client
        - Mark time entries as billed
        - Sync to ERP-Finance AR
    end note

    note right of PAID
        Actions:
        - Record payment
        - Update project revenue
        - Sync to ERP-Finance
    end note
```

---

## 7. Automated Workflows

### 7.1 Scheduled Automations

| Automation                        | Schedule  | Description                            |
|-----------------------------------|-----------|----------------------------------------|
| Overdue task escalation          | Hourly    | Escalate priority if overdue > 3 days  |
| Health score recalculation       | Every 15m | Recalculate project health scores      |
| Timesheet submission reminder    | Friday PM | Remind users to submit weekly timesheet|
| Sprint auto-close                | Daily     | Complete sprints past end date         |
| Budget alert check               | Hourly    | Check budget thresholds                |
| Recurring task creation          | Daily     | Create instances of recurring tasks    |
| Project auto-archive             | Daily     | Archive completed/cancelled projects   |
| Stale timer cleanup              | Every 12h | Auto-stop timers running > 12 hours   |

### 7.2 Event-Driven Automations

| Trigger Event                    | Automated Action                          |
|----------------------------------|-------------------------------------------|
| Task completed                   | Recalculate parent task completion %      |
| All tasks in project completed   | Prompt project completion                 |
| Time entry created               | Update task actual hours                  |
| Time entry approved              | Update project spent amount               |
| Resource allocation changed      | Recalculate timeline conflicts            |
| Dependency added                 | Check for circular dependencies           |
| Project health drops to CRITICAL | Notify PMO and executive sponsor          |
| Sprint completed                 | Update velocity, prompt retrospective     |

---

## 8. Workflow Configuration

### 8.1 Custom Status Workflows (Planned)

Tenants will be able to define custom task status workflows:

```json
{
  "workflowName": "Software Development",
  "statuses": [
    { "name": "BACKLOG", "category": "TODO" },
    { "name": "READY", "category": "TODO" },
    { "name": "IN_DEVELOPMENT", "category": "IN_PROGRESS" },
    { "name": "CODE_REVIEW", "category": "IN_REVIEW" },
    { "name": "TESTING", "category": "IN_REVIEW" },
    { "name": "DONE", "category": "DONE" }
  ],
  "transitions": [
    { "from": "BACKLOG", "to": ["READY"] },
    { "from": "READY", "to": ["IN_DEVELOPMENT"] },
    { "from": "IN_DEVELOPMENT", "to": ["CODE_REVIEW", "BLOCKED"] },
    { "from": "CODE_REVIEW", "to": ["IN_DEVELOPMENT", "TESTING"] },
    { "from": "TESTING", "to": ["IN_DEVELOPMENT", "DONE"] },
    { "from": "BLOCKED", "to": ["IN_DEVELOPMENT"] }
  ]
}
```
