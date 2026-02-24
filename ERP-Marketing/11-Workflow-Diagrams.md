# ERP-Marketing -- Workflow Diagrams

## 1. Campaign Lifecycle Workflow

```mermaid
flowchart TB
    START([Campaign Brief]) --> CREATE[Create Campaign<br/>Name, channel, objective, budget]
    CREATE --> AUDIENCE[Select Audience<br/>Segment or audience list]
    AUDIENCE --> TEMPLATE[Assign Template<br/>Email, SMS, or social content]
    TEMPLATE --> REVIEW{Content Review<br/>Complete?}
    REVIEW -->|No| EDIT[Edit Content]
    EDIT --> TEMPLATE
    REVIEW -->|Yes| SCHEDULE{Schedule or<br/>Launch Now?}
    SCHEDULE -->|Schedule| SCHED_TIME[Set Schedule Time]
    SCHED_TIME --> WAIT_SCHED[Wait for Scheduled Time]
    WAIT_SCHED --> GUARDRAIL
    SCHEDULE -->|Launch Now| GUARDRAIL{AIDD Guardrail<br/>Evaluation}
    GUARDRAIL -->|Auto-Approved| SENDING[Campaign Sending]
    GUARDRAIL -->|Needs Review| HUMAN[Human Approver Review]
    GUARDRAIL -->|Blocked| REVISE[Revise Campaign Parameters]
    REVISE --> CREATE
    HUMAN -->|Approved| SENDING
    HUMAN -->|Rejected| REVISE
    SENDING --> DELIVER[Deliver to Recipients]
    DELIVER --> TRACK[Track Metrics<br/>Opens, clicks, bounces]
    TRACK --> COMPLETE{All Sent?}
    COMPLETE -->|No| DELIVER
    COMPLETE -->|Yes| SENT[Campaign Sent]
    SENT --> REPORT[Generate Performance Report]
    REPORT --> ATTRIBUTE[Attribution Calculation]
    ATTRIBUTE --> END([Campaign Complete])
```

## 2. Journey Execution Workflow

```mermaid
flowchart TB
    ENTRY([Contact Enters Segment]) --> ENROLL[Enroll in Journey]
    ENROLL --> STEP{Next Step Type?}
    STEP -->|send_message| SEND[Send Message<br/>Email/SMS/In-App]
    STEP -->|wait| WAIT[Wait Period<br/>Minutes/Hours/Days]
    STEP -->|branch| EVAL{Evaluate Condition}
    STEP -->|escalation| TASK[Create Task<br/>for Manual Follow-up]
    SEND --> NEXT{More Steps?}
    WAIT --> NEXT
    EVAL -->|Condition True| TRUE_PATH[True Branch Steps]
    EVAL -->|Condition False| FALSE_PATH[False Branch Steps]
    TRUE_PATH --> NEXT
    FALSE_PATH --> NEXT
    TASK --> NEXT
    NEXT -->|Yes| STEP
    NEXT -->|No| GOAL{Goal Achieved?}
    GOAL -->|Yes| SUCCESS[Mark Completed<br/>Goal Met]
    GOAL -->|No| EXIT[Mark Completed<br/>Journey End]
    SUCCESS --> EVENT[Emit Journey Completion Event]
    EXIT --> EVENT
    EVENT --> END([Journey Complete for Contact])
```

## 3. Lead Scoring Workflow

```mermaid
flowchart TB
    TRIGGER([Behavioral Event<br/>Page view, form submit, email open]) --> CAPTURE[Capture Event<br/>Touchpoint record]
    CAPTURE --> LOOKUP[Lookup Scoring Model<br/>Active model rules]
    LOOKUP --> CALC[Calculate Score Delta<br/>Apply rule weights]
    CALC --> UPDATE[Update Contact Score<br/>New total score]
    UPDATE --> THRESHOLD{Score >= SQL<br/>Threshold?}
    THRESHOLD -->|Yes, New SQL| GUARDRAIL{AIDD Guardrail<br/>Evaluate Stage Change}
    THRESHOLD -->|No| CONTINUE[Continue Monitoring]
    GUARDRAIL -->|Approved| PROMOTE[Promote to SQL<br/>Update lifecycle_stage]
    GUARDRAIL -->|Needs Review| HUMAN_REVIEW[Human Review<br/>Score Change]
    HUMAN_REVIEW -->|Approved| PROMOTE
    HUMAN_REVIEW -->|Rejected| CONTINUE
    PROMOTE --> NOTIFY[Create Task for Sales<br/>New SQL handoff]
    NOTIFY --> JOURNEY[Trigger High-Intent<br/>Journey Branch]
    JOURNEY --> EVENT[Emit Score Changed Event]
    CONTINUE --> EVENT
    EVENT --> END([Scoring Complete])
```

## 4. Content Publishing Workflow

```mermaid
flowchart TB
    IDEA([Content Idea]) --> BRIEF[Create Content Brief<br/>Type, keywords, audience]
    BRIEF --> DRAFT[Write Draft<br/>Title, body, SEO keywords]
    DRAFT --> SEO[SEO Optimization<br/>Slug, meta, keywords]
    SEO --> CTA[Configure CTA<br/>Label, destination URL]
    CTA --> PREVIEW[Preview Content<br/>Desktop + Mobile]
    PREVIEW --> REVIEW{Editorial Review}
    REVIEW -->|Approved| PUBLISH[Publish to CMS]
    REVIEW -->|Revisions Needed| DRAFT
    PUBLISH --> INDEX[Index for Search<br/>SEO crawlability]
    INDEX --> PROMOTE{Promote?}
    PROMOTE -->|Campaign| CAMPAIGN[Create Campaign<br/>Linking to Content]
    PROMOTE -->|Social| SOCIAL[Create Social Posts<br/>Linking to Content]
    PROMOTE -->|Both| CAMPAIGN
    CAMPAIGN --> SOCIAL
    PROMOTE -->|Skip| MONITOR
    SOCIAL --> MONITOR[Monitor Performance<br/>Views, engagement, conversions]
    MONITOR --> END([Content Live])
```

## 5. Social Media Publishing Workflow

```mermaid
flowchart TB
    CREATE([Create Social Post]) --> CHANNEL[Select Platform<br/>LinkedIn, X, Facebook, Instagram, TikTok]
    CHANNEL --> COMPOSE[Compose Content<br/>Title, body, media]
    COMPOSE --> LINK{Link to Campaign?}
    LINK -->|Yes| CAMPAIGN[Associate Campaign<br/>for Attribution]
    LINK -->|No| SCHEDULE_Q
    CAMPAIGN --> SCHEDULE_Q{Schedule or<br/>Publish Now?}
    SCHEDULE_Q -->|Schedule| SET_TIME[Set Publication Time]
    SET_TIME --> QUEUE[Queue for Publishing]
    QUEUE --> WAIT_TIME[Wait for Scheduled Time]
    WAIT_TIME --> GUARDRAIL
    SCHEDULE_Q -->|Now| GUARDRAIL{AIDD Guardrail<br/>Evaluate Publishing}
    GUARDRAIL -->|Approved| PUBLISH[Publish to Platform API]
    GUARDRAIL -->|Needs Review| HUMAN[Human Approval]
    HUMAN -->|Approved| PUBLISH
    HUMAN -->|Rejected| COMPOSE
    PUBLISH --> TRACK[Track Engagement<br/>Likes, comments, shares]
    TRACK --> SENTIMENT[Analyze Sentiment<br/>on Responses]
    SENTIMENT --> END([Post Published])
```

## 6. A/B Testing Workflow

```mermaid
flowchart TB
    HYPO([Formulate Hypothesis]) --> CREATE[Create Experiment<br/>Name, hypothesis, campaign]
    CREATE --> VARIANTS[Define Variants<br/>Control + N variations]
    VARIANTS --> SPLIT[Configure Traffic Split<br/>Equal or custom %]
    SPLIT --> LAUNCH[Launch Experiment]
    LAUNCH --> DISTRIBUTE[Distribute to Variants<br/>per Traffic Rules]
    DISTRIBUTE --> COLLECT[Collect Performance Data<br/>Opens, clicks, conversions]
    COLLECT --> SIG{Statistical<br/>Significance?}
    SIG -->|Not Yet| COLLECT
    SIG -->|Yes| WINNER[Declare Winner Variant]
    WINNER --> APPLY[Apply Winner<br/>to Remaining Audience]
    APPLY --> REPORT[Generate Experiment Report]
    REPORT --> LEARN[Document Learnings<br/>for Future Campaigns]
    LEARN --> END([Experiment Complete])
```

## 7. Data Sync Workflow

```mermaid
flowchart TB
    TRIGGER([Cron Schedule Fires]) --> CONNECT[Connect to Source System<br/>Salesforce, Zendesk, etc.]
    CONNECT --> FETCH[Fetch Records<br/>Since Last Sync]
    FETCH --> TRANSFORM[Transform Records<br/>Map fields to schema]
    TRANSFORM --> VALIDATE{Validation<br/>Pass?}
    VALIDATE -->|Yes| UPSERT[Upsert to PostgreSQL<br/>Insert or Update]
    VALIDATE -->|No| ERROR[Log Error<br/>Increment error_rate]
    ERROR --> CONTINUE{More Records?}
    UPSERT --> CONTINUE
    CONTINUE -->|Yes| TRANSFORM
    CONTINUE -->|No| COMPLETE[Update Sync Job<br/>last_run_at, records_processed]
    COMPLETE --> END([Sync Complete])
```

## 8. Guardrail Decision Workflow

```mermaid
flowchart TB
    ACTION([High-Impact Action Requested]) --> EVALUATE[Evaluate Action Parameters]
    EVALUATE --> CONF{Confidence >= Min?}
    CONF -->|No| BLOCKED[Decision: BLOCKED<br/>Action Denied]
    CONF -->|Yes| BLAST{Blast Radius <= Max?}
    BLAST -->|No| REVIEW[Decision: NEEDS REVIEW<br/>Named Approver Required]
    BLAST -->|Yes| VALUE{Monetary Value >= High?}
    VALUE -->|Yes| REVIEW
    VALUE -->|No| MED_CONF{Confidence >= Medium?}
    MED_CONF -->|No| REVIEW
    MED_CONF -->|Yes| APPROVED[Decision: APPROVED<br/>Action Permitted]
    BLOCKED --> LOG[Log Guardrail Event]
    REVIEW --> LOG
    APPROVED --> LOG
    LOG --> PERSIST[Persist to marketing_aidd_guardrail_events]
    PERSIST --> END([Decision Recorded])
```

## 9. Opportunity Pipeline Workflow

```mermaid
flowchart LR
    QUAL[Qualification] --> DISC[Discovery]
    DISC --> PROP[Proposal]
    PROP --> NEG[Negotiation]
    NEG --> CLOSE{Close?}
    CLOSE -->|Won| WON[Closed-Won<br/>Revenue Attributed]
    CLOSE -->|Lost| LOST[Closed-Lost<br/>Loss Reason Logged]
    WON --> EXPAND[Expansion Opportunity]
    LOST --> RECOVER[Stalled Deal Recovery<br/>Playbook]
```

## 10. End-to-End Marketing Funnel

```mermaid
flowchart TB
    subgraph "Top of Funnel"
        ADS[Paid Ads<br/>Google, Meta, LinkedIn, TikTok]
        SOCIAL[Social Posts<br/>LinkedIn, X, Facebook]
        BLOG[Blog Content<br/>SEO-Driven]
        LP[Landing Pages<br/>with Forms]
    end
    subgraph "Middle of Funnel"
        CAPTURE[Lead Capture<br/>Form Submission]
        SCORE[Lead Scoring<br/>Behavioral + Firmographic]
        SEGMENT[Dynamic Segmentation]
        NURTURE[Journey Nurture<br/>Multi-Channel]
    end
    subgraph "Bottom of Funnel"
        MQL[MQL Handoff<br/>to Sales]
        SEQ[Sales Sequences<br/>Outbound Follow-up]
        MTG[Meetings<br/>Discovery + Proposals]
        OPP[Opportunity Management]
    end
    subgraph "Post-Sale"
        ONBOARD[Customer Onboarding Journey]
        EXPAND2[Expansion Campaigns]
        ADVOCATE[Advocacy Programs]
    end
    ADS & SOCIAL & BLOG --> LP --> CAPTURE
    CAPTURE --> SCORE --> SEGMENT --> NURTURE
    NURTURE --> MQL --> SEQ --> MTG --> OPP
    OPP --> ONBOARD --> EXPAND2 --> ADVOCATE
```
