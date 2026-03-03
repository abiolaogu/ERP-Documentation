# Portfolio Figma Category-King Playbook

## Purpose

This playbook sets the creative and execution bar for all project-labeled Figma Make prompts in this repository. It ensures every Product, System, Tenant, Org, User, Admin, API, Service, and Environment touchpoint is represented with category-defining clarity.

## Design Intent Hierarchy

1. Mission-critical workflow completion speed.
2. Operational trust and policy transparency.
3. Information architecture that scales with data density.
4. Distinct visual identity per project without fragmenting platform consistency.
5. Accessible-by-default interaction models.

## Universal Prompt Scaffolding

Use this shape for all project prompts:

```text
Context: product + role + environment.
Goal: business and user outcome.
Inputs: data objects, policy constraints, API states.
Output format: flows/screens/components/prototype.
Quality gates: accessibility, consistency, latency assumptions, auditability.
```

## Multi-Tenant UX Guardrails

- Tenant context is always visible in global shell.
- Org and role scope are visible near critical actions.
- Admin actions require clear blast-radius messaging.
- API-induced delays have deterministic skeleton and retry states.
- Service degradation has explicit fallback UI and timeline status.

## Information Architecture Rules

- Use a three-zone rule on primary pages:
  - Command zone (intent + next action)
  - Insight zone (metrics + alerts + context)
  - Execution zone (table/form/timeline)
- Keep cognitive load low with progressive disclosure.
- Every dense table needs saved views, quick filters, and keyboard shortcuts.

## Motion and Interaction Rules

- Motion communicates causality, not decoration.
- Use transitions to explain hierarchy changes only.
- For optimistic updates, show immediate state + rollback affordance.
- Toasts include action and context, not generic success text.

## Content Strategy Rules

- Replace ambiguous labels with action verbs.
- Error copy includes cause, impact, and next step.
- Compliance copy is plain-language and user-legible.
- Empty states provide one guided path to value.

## Accessibility Baseline

- WCAG AA minimum contrast on all text and controls.
- Focus order follows business workflow sequence.
- Components have keyboard path and screen-reader labels.
- Color is never the only state indicator.

## Category-King Review Rubric

Score 1-10 for each criterion. No screen ships below 8.

| Criterion | What Good Looks Like |
|---|---|
| Workflow clarity | User can predict next action without ambiguity. |
| Decision confidence | Data context is sufficient for high-stakes actions. |
| Policy transparency | Role restrictions and audit effects are explicit. |
| Error recoverability | User can recover without external support. |
| Visual hierarchy | Attention is directed to value-driving actions first. |
| Consistency | Cross-project patterns feel coherent and deliberate. |
| Responsiveness | Mobile/tablet preserve critical workflow capability. |

## Handoff Contract

Every project handoff includes:

- Screen map with scenario labels.
- Component and token references.
- State matrix (empty/loading/error/degraded/live).
- API dependency notes per screen.
- QA checklist with accessibility and policy checks.

