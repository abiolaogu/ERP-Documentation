# ERP-CRM Workflow Diagrams

## 1. Lead-to-Close Workflow

This is the primary revenue workflow, tracking a lead from initial capture through deal closure.

```mermaid
flowchart TB
    START([Lead Source<br/>Web Form / Chat / Import / Manual]) --> CAPTURE["Create Contact<br/>email validated, UUID v7 generated"]
    CAPTURE --> SCORE["AI Lead Scoring<br/>Demographics + Behavior"]
    SCORE --> CLASSIFY{Score<br/>Classification}

    CLASSIFY -->|"Hot (80-100)"| ROUTE_HOT["Auto-Route to<br/>Top Rep (immediate)"]
    CLASSIFY -->|"Warm (50-79)"| ROUTE_WARM["Route to<br/>Sales Rep (48h)"]
    CLASSIFY -->|"Cold (0-49)"| NURTURE["Add to<br/>Nurture Campaign"]

    NURTURE --> RESCORE{"Score<br/>Improved?"}
    RESCORE -->|Yes| ROUTE_WARM
    RESCORE -->|No| NURTURE

    ROUTE_HOT --> QUALIFY["Sales Rep<br/>Qualifies Lead"]
    ROUTE_WARM --> QUALIFY

    QUALIFY --> QUALIFIED{Qualification<br/>Decision}
    QUALIFIED -->|Qualified| CREATE_DEAL["Create Deal<br/>in Pipeline"]
    QUALIFIED -->|Not Ready| NURTURE
    QUALIFIED -->|Disqualified| DISQUALIFY["Disqualify<br/>with Reason"]
    DISQUALIFY --> ARCHIVE["Archive<br/>Contact"]

    CREATE_DEAL --> STAGE1["Stage: Qualified<br/>25% probability"]
    STAGE1 --> PROPOSAL["Stage: Proposal<br/>50% probability"]
    PROPOSAL --> NEGOTIATE["Stage: Negotiation<br/>75% probability"]
    NEGOTIATE --> DECISION{Customer<br/>Decision}

    DECISION -->|Yes| WON["Close Won<br/>probability = 100%"]
    DECISION -->|No| LOST["Close Lost<br/>probability = 0%<br/>Record reason"]

    WON --> CONVERT["Convert Contact<br/>to Customer"]
    CONVERT --> HANDOFF["Handoff to<br/>Support Team"]
    HANDOFF --> ONGOING["Ongoing Support<br/>& Upsell"]
    ONGOING --> UPSELL{Upsell<br/>Opportunity?}
    UPSELL -->|Yes| CREATE_DEAL
    UPSELL -->|No| RETAIN["Retention<br/>Activities"]

    LOST --> ANALYSIS["Win/Loss<br/>Analysis"]
    ANALYSIS --> IMPROVE["Improve<br/>Sales Process"]

    style WON fill:#22c55e,color:#fff
    style LOST fill:#ef4444,color:#fff
    style NURTURE fill:#f59e0b,color:#fff
```

## 2. Ticket Lifecycle Workflow

The complete support ticket lifecycle from creation to closure.

```mermaid
flowchart TB
    START2([Customer Issue]) --> CHANNEL{Arrival<br/>Channel}
    CHANNEL -->|Email| PARSE_EMAIL["Parse Email<br/>Extract Subject/Body"]
    CHANNEL -->|Web Form| PARSE_FORM["Parse Form<br/>Submission"]
    CHANNEL -->|Live Chat| CHATBOT_CHECK{Chatbot<br/>Resolves?}
    CHANNEL -->|Phone| MANUAL_CREATE["Agent Creates<br/>Ticket Manually"]
    CHANNEL -->|API| API_CREATE["API Ticket<br/>Creation"]

    CHATBOT_CHECK -->|Yes| RESOLVED_CHAT([Resolved<br/>via Chat])
    CHATBOT_CHECK -->|No| ESCALATE_CHAT["Escalate to<br/>Human Agent"]

    PARSE_EMAIL --> CREATE_TICKET["Create Ticket<br/>Status: New"]
    PARSE_FORM --> CREATE_TICKET
    ESCALATE_CHAT --> CREATE_TICKET
    MANUAL_CREATE --> CREATE_TICKET
    API_CREATE --> CREATE_TICKET

    CREATE_TICKET --> ASSIGN_SLA["Assign SLA Policy<br/>Start Breach Timer"]
    ASSIGN_SLA --> AUTO_ASSIGN{Auto-Assignment<br/>Rule Exists?}

    AUTO_ASSIGN -->|Yes| ASSIGN_AUTO["Auto-Assign<br/>to Agent"]
    AUTO_ASSIGN -->|No| QUEUE["Add to<br/>Unassigned Queue"]
    QUEUE --> AGENT_PICK["Agent Picks<br/>from Queue"]

    ASSIGN_AUTO --> OPEN["Status: Open"]
    AGENT_PICK --> OPEN

    OPEN --> INVESTIGATE["Agent<br/>Investigates"]
    INVESTIGATE --> KB_SEARCH{KB Article<br/>Available?}

    KB_SEARCH -->|Yes| SHARE_KB["Share KB Article<br/>with Customer"]
    KB_SEARCH -->|No| DEEP_INVESTIGATE["Deep<br/>Investigation"]

    SHARE_KB --> CUSTOMER_REPLY{Customer<br/>Satisfied?}
    DEEP_INVESTIGATE --> NEED_INFO{Need More<br/>Info?}

    NEED_INFO -->|Yes| PENDING["Status: Pending<br/>Awaiting Customer"]
    NEED_INFO -->|No| RESOLVE["Add Resolution<br/>Comment"]

    PENDING --> CUSTOMER_RESPONDS{Customer<br/>Responds?}
    CUSTOMER_RESPONDS -->|Yes| OPEN
    CUSTOMER_RESPONDS -->|No, SLA Risk| SLA_CHECK{SLA<br/>Breach Risk?}

    SLA_CHECK -->|Yes| ESCALATE2["Escalate<br/>Priority: Urgent"]
    SLA_CHECK -->|No| PENDING

    ESCALATE2 --> SENIOR["Senior Agent<br/>Takes Over"]
    SENIOR --> RESOLVE

    CUSTOMER_REPLY -->|Yes| RESOLVE
    CUSTOMER_REPLY -->|No| INVESTIGATE

    RESOLVE --> SOLVED["Status: Solved"]
    SOLVED --> CONFIRM{Customer<br/>Confirms?}
    CONFIRM -->|Yes| CLOSED["Status: Closed"]
    CONFIRM -->|Reopens| OPEN
    CONFIRM -->|No Response 48h| CLOSED

    CLOSED --> CSAT["Send CSAT<br/>Survey"]

    style CLOSED fill:#22c55e,color:#fff
    style ESCALATE2 fill:#ef4444,color:#fff
    style PENDING fill:#f59e0b,color:#fff
```

## 3. Form Submission Workflow

From form creation to lead capture and CRM integration.

```mermaid
flowchart TB
    ADMIN([Administrator]) --> BUILD["Build Form<br/>name, slug, fields, settings"]
    BUILD --> PUBLISH["Publish Form<br/>status: active"]
    PUBLISH --> EMBED["Embed on Website<br/>iframe or JS widget"]

    VISITOR([Website Visitor]) --> VIEW["View Page<br/>with Embedded Form"]
    VIEW --> FILL["Fill Form Fields"]
    FILL --> SUBMIT["Click Submit"]
    SUBMIT --> VALIDATE{Server-Side<br/>Validation}

    VALIDATE -->|Pass| STORE["Store Submission<br/>data + metadata"]
    VALIDATE -->|Fail| ERROR["Show Validation<br/>Errors"]
    ERROR --> FILL

    STORE --> EVENT["Publish<br/>form-builder.created Event"]
    EVENT --> AUTOMATION{Web-to-Lead<br/>Enabled?}

    AUTOMATION -->|Yes| CHECK_EXIST{Contact<br/>Exists?}
    AUTOMATION -->|No| NOTIFY["Send Notification<br/>Email to Admin"]

    CHECK_EXIST -->|No| CREATE_CONTACT["Create Contact<br/>from Form Data"]
    CHECK_EXIST -->|Yes| UPDATE_CONTACT["Update Contact<br/>Custom Fields"]

    CREATE_CONTACT --> SCORE2["Calculate Initial<br/>Lead Score"]
    UPDATE_CONTACT --> RESCORE2["Recalculate<br/>Lead Score"]

    SCORE2 --> ASSIGN3["Apply Assignment<br/>Rules"]
    RESCORE2 --> ASSIGN3
    ASSIGN3 --> CRM_RECORD["Contact in CRM<br/>with Form Source"]

    NOTIFY --> MANUAL_REVIEW["Admin Reviews<br/>Submission"]

    style CRM_RECORD fill:#22c55e,color:#fff
```

## 4. Chat Escalation Workflow

Live chat conversation flow with chatbot and human agent escalation.

```mermaid
flowchart TB
    VISITOR2([Website Visitor]) --> WIDGET["Open Chat Widget"]
    WIDGET --> CHATBOT2["Chatbot Greeting<br/>'How can I help?'"]
    CHATBOT2 --> INPUT["Visitor Describes<br/>Issue"]
    INPUT --> AI_ANALYZE{AI Analyzes<br/>Intent}

    AI_ANALYZE -->|Simple Query| KB_MATCH{KB Article<br/>Match?}
    AI_ANALYZE -->|Complex Issue| HUMAN["Route to<br/>Human Agent"]
    AI_ANALYZE -->|Sales Inquiry| SALES["Route to<br/>Sales Rep"]

    KB_MATCH -->|Yes| SUGGEST_ARTICLE["Chatbot Suggests<br/>KB Article"]
    KB_MATCH -->|No| HUMAN

    SUGGEST_ARTICLE --> VISITOR_SATISFIED{Visitor<br/>Satisfied?}
    VISITOR_SATISFIED -->|Yes| CLOSE_CHAT["Close Chat<br/>Session"]
    VISITOR_SATISFIED -->|No| HUMAN

    HUMAN --> QUEUE2["Agent Chat<br/>Queue"]
    QUEUE2 --> AGENT_ACCEPT["Agent Accepts<br/>Chat"]
    AGENT_ACCEPT --> CONVERSATION["Real-Time<br/>Conversation"]
    CONVERSATION --> RESOLVE3{Issue<br/>Resolved?}

    RESOLVE3 -->|Yes| CLOSE_CHAT
    RESOLVE3 -->|No, Needs Ticket| CREATE_TICKET2["Create Support<br/>Ticket from Chat"]

    SALES --> SALES_QUEUE["Sales Rep<br/>Chat Queue"]
    SALES_QUEUE --> SALES_CONV["Sales<br/>Conversation"]
    SALES_CONV --> LEAD_CAPTURE["Capture Lead<br/>Create Contact"]

    CLOSE_CHAT --> TRANSCRIPT["Save Chat<br/>Transcript"]
    TRANSCRIPT --> ANALYTICS["Chat<br/>Analytics"]

    CREATE_TICKET2 --> TICKET_FLOW["Enter Ticket<br/>Lifecycle"]

    style CLOSE_CHAT fill:#22c55e,color:#fff
    style CREATE_TICKET2 fill:#3b82f6,color:#fff
```

## 5. Contact Lifecycle Workflow

The complete lifecycle of a contact through all stages.

```mermaid
flowchart LR
    SUB["Subscriber<br/>Score: 0-20"] -->|Email/Event Engagement| LEAD["Lead<br/>Score: 20-40"]
    LEAD -->|Marketing Engagement| MQL["Marketing<br/>Qualified Lead<br/>Score: 40-60"]
    MQL -->|Sales Accepts| SQL["Sales Qualified<br/>Lead<br/>Score: 60-80"]
    SQL -->|Deal Created| OPP["Opportunity<br/>Score: 80+"]
    OPP -->|Deal Won| CUST["Customer"]
    CUST -->|High Advocacy| EVAN["Evangelist"]

    LEAD -->|Disqualified| DQ["Disqualified"]
    MQL -->|Disqualified| DQ
    SQL -->|Disqualified| DQ

    style CUST fill:#22c55e,color:#fff
    style EVAN fill:#8b5cf6,color:#fff
    style DQ fill:#ef4444,color:#fff
```

## 6. Deal Stage Transition Workflow

Internal system flow when a deal moves between stages.

```mermaid
sequenceDiagram
    participant Rep as Sales Rep
    participant API as REST API
    participant Deal as Deal Aggregate
    participant DB as PostgreSQL
    participant Events as NATS/Pulsar

    Rep->>API: PUT /deals/:id/stage
    API->>DB: Load deal by ID
    DB-->>API: Deal record
    API->>Deal: move_to_stage(new_stage_id, probability)

    alt Deal is Open
        Deal->>Deal: Validate not same stage
        Deal->>Deal: Record StageChange
        Deal->>Deal: Update stage_id, probability
        Deal->>Deal: Raise DealEvent::StageChanged
        Deal-->>API: Ok(deal)
        API->>DB: Persist updated deal
        API->>Events: Publish deal.stage_changed
        API-->>Rep: 200 OK with updated deal
    else Deal is Closed
        Deal-->>API: Err(DealNotOpen)
        API-->>Rep: 400 Bad Request
    end
```

## 7. Automation Engine Workflow

How the automation service processes events and executes rules.

```mermaid
flowchart TB
    EVENT([Domain Event<br/>Published]) --> RECEIVE["Automation Service<br/>Receives Event"]
    RECEIVE --> MATCH["Match Event Against<br/>Configured Rules"]
    MATCH --> RULES{Matching<br/>Rules Found?}

    RULES -->|No| LOG["Log: No rules matched"]
    RULES -->|Yes| EVALUATE["Evaluate Rule<br/>Conditions"]

    EVALUATE --> CONDITIONS{All Conditions<br/>Met?}
    CONDITIONS -->|No| SKIP["Skip Rule<br/>Log Reason"]
    CONDITIONS -->|Yes| EXECUTE["Execute Actions"]

    EXECUTE --> ACTION1["Action: Send<br/>Notification"]
    EXECUTE --> ACTION2["Action: Update<br/>Entity Field"]
    EXECUTE --> ACTION3["Action: Create<br/>Follow-up Activity"]
    EXECUTE --> ACTION4["Action: Publish<br/>Webhook"]
    EXECUTE --> ACTION5["Action: Assign<br/>to User/Group"]

    ACTION1 & ACTION2 & ACTION3 & ACTION4 & ACTION5 --> AUDIT_LOG["Write Audit Log<br/>Rule ID, Actions, Outcome"]
    AUDIT_LOG --> DONE([Complete])
```

## 8. Data Import Workflow

Bulk data import process for CRM migration.

```mermaid
flowchart TB
    UPLOAD([Admin Uploads<br/>CSV File]) --> PARSE["Parse CSV<br/>Headers and Rows"]
    PARSE --> MAP["Column Mapping<br/>CSV Column -> CRM Field"]
    MAP --> VALIDATE3["Validate Each Row<br/>Email format, required fields"]
    VALIDATE3 --> REPORT{Validation<br/>Report}

    REPORT -->|Errors Found| FIX["Admin Fixes<br/>Errors"]
    FIX --> UPLOAD
    REPORT -->|All Valid| DEDUP["Deduplication Check<br/>Match by email"]

    DEDUP --> STRATEGY{Duplicate<br/>Strategy}
    STRATEGY -->|Skip| SKIP2["Skip Existing<br/>Records"]
    STRATEGY -->|Update| UPDATE2["Update Existing<br/>Records"]
    STRATEGY -->|Create New| CREATE2["Create All<br/>as New Records"]

    SKIP2 & UPDATE2 & CREATE2 --> BATCH["Batch Insert/Update<br/>PostgreSQL"]
    BATCH --> EVENTS2["Publish Events<br/>Batch Created/Updated"]
    EVENTS2 --> SUMMARY["Import Summary<br/>Created/Updated/Skipped/Failed"]
```
