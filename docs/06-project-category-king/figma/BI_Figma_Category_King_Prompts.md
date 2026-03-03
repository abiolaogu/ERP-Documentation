# BI Figma Category-King Prompts

## Objective

Create a category-defining BI Product experience that outperforms incumbents on clarity, speed, trust, and operational depth for every User, Admin, Org, Tenant, API, and Service touchpoint.

## Global Creative Direction Prompt

Use this as the first prompt in Figma Make:

```text
You are designing the BI Product for enterprise-grade multi-tenant operations. Build a premium, high-clarity interface with decisive information hierarchy, strong contrast, and calm visual rhythm. The System should feel fast, trustworthy, and intelligent. Prioritize critical workflows, contextual guidance, and transparent state changes. Generate desktop-first and mobile-adaptive variants. Use modern enterprise visual language with distinctive personality, not generic SaaS tropes.
```

## Stage 1: BRD -> PRD Translation Prompt

```text
Transform BRD priorities into a PRD-aligned UX blueprint for BI. Include: personas (User/Admin), top 10 jobs-to-be-done, risk-heavy workflows, failure states, Environment differences (dev/staging/prod), and measurable success criteria. Output IA map + screen inventory + priority matrix.
```

## Stage 2: UX Flow Architecture Prompt

```text
Design end-to-end UX flows for BI covering: onboarding, core hero workflow (Explore dashboards, drill into KPIs, schedule briefings), exception handling, approvals, escalation, and audit review. Include branch logic for role permissions, Tenant context switching, and Org-level policy constraints. Produce flow nodes with entry/exit conditions and friction risk notes.
```

## Stage 3: Low-Fidelity Wireframe Prompt

```text
Generate lo-fi wireframes for BI with explicit layout zones: global nav, contextual side panel, primary workspace, command bar, activity rail, and evidence drawer. Include data-dense states and empty/loading/error states. Maintain keyboard-first operability and visible hierarchy.
```

## Stage 4: High-Fidelity Visual Language Prompt

```text
Elevate the wireframes into high-fidelity UI for BI. Define typography scale, color tokens, spacing system, iconography voice, and elevation model. Build visual cadence for dashboards, forms, tables, timelines, and detail views. Ensure design supports expert users at high information throughput without cognitive overload.
```

## Stage 5: Design System Prompt

```text
Create a BI design system package with components, variants, states, and usage rules. Include: buttons, inputs, selects, data table, filters, chips, status badges, timeline cards, notification center, command palette, modal/sheet patterns, and audit evidence panel. Define accessibility specs (contrast, focus rings, keyboard paths, aria notes).
```

## Stage 6: Interaction and Motion Prompt

```text
Define motion for BI: page transitions, panel reveals, sortable table feedback, optimistic update confirmation, toast stack behavior, and loading skeleton choreography. Motion should communicate causality and confidence, avoid noise, and remain performance-safe.
```

## Stage 7: Realtime + Status Intelligence Prompt

```text
Design realtime UI behaviors for BI: live counters, freshness indicators, conflict banners, stale-data warning, and safe-refresh controls. Add visual semantics for system health, degraded mode, and recovery. Show how User and Admin views differ while sharing one design language.
```

## Stage 8: Admin Control Plane Prompt

```text
Generate Admin portal screens for BI: Tenant management, Org settings, role editor, API credentials, policy simulator, audit explorer, and compliance evidence export. Include guardrails for destructive actions, approval chains, and dual confirmation for high-risk operations.
```

## Stage 9: End-User Experience Prompt

```text
Generate end-user portal screens for BI: task dashboard, guided actions, contextual help, notifications center, and personalized work queues. Emphasize low-friction completion with transparent dependencies and clear next-best-action guidance.
```

## Stage 10: Billing + Notifications + Audit Prompt

```text
Create dedicated flows for billing summary, charge lifecycle, notification preferences, and audit event detail in BI. Include reconciliation views, anomaly flags, and event trace drill-down. Ensure each action exposes policy context and actor metadata.
```

## Stage 11: Responsive and Mobile Prompt

```text
Adapt BI for tablet/mobile with role-aware prioritization. Convert dense desktop tables to stacked cards with progressive reveal. Preserve mission-critical actions via sticky command bar, quick filters, and bottom-sheet detail expansion.
```

## Stage 12: Microcopy and Content Strategy Prompt

```text
Write product microcopy for BI: headers, helper text, empty states, errors, warnings, success confirmations, and compliance banners. Tone: authoritative, concise, and humane. Replace ambiguous language with action-oriented directives.
```

## Stage 13: Prototype and Handoff Prompt

```text
Produce an interactive prototype for BI covering primary and failure paths. Include component annotations, token references, responsive behaviors, and engineer handoff notes mapped to API/Service dependencies. Export a screen-level checklist for QA and accessibility verification.
```

## Stage 14: Category-King Quality Rubric Prompt

```text
Audit the full BI design against category-king criteria: clarity under load, workflow completion speed, error prevention, policy transparency, accessibility, consistency, and emotional trust. Score each screen 1-10 with actionable redesign instructions for anything below 8.
```

## Stage 15: Experimentation Prompt

```text
Design 6 A/B experiments for BI across onboarding, conversion, task completion, and retention. For each test include hypothesis, variant spec, primary metric, guardrail metric, and expected impact range.
```

## Deliverable Checklist for Figma Team

- Project-specific UI sitemap
- Core and edge-case flow boards
- Tokenized design system assets
- Desktop + mobile variants
- Interaction prototype with annotations
- Admin and User role-specific views
- Accessibility and QA checklists
- Handoff package tied to API and Service contracts
