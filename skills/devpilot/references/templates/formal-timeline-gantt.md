<!-- GENERATED from docs/core/ on {YYYY-MM-DD} by devpilot — do not hand-edit.
     Regenerate with /devpilot docs gantt. -->
# Project Timeline — {Project Name}

> Generated {YYYY-MM-DD} from docs/core/. Durations derive from mandays in
> features.md and phase order in phases.md. Estimates, not commitments.

## Assumptions

| Assumption | Value |
|---|---|
| Working days | {Mon–Fri} |
| Team size per division | {1 person each unless noted} |
| Start date | {YYYY-MM-DD} |

## Gantt chart

<!-- One section per division from config.md. Divisions run in parallel where
     phase dependencies allow. Task ids let dependencies use `after`. -->

```mermaid
gantt
    title {Project Name} — Delivery Plan
    dateFormat YYYY-MM-DD
    axisFormat %d %b

    section {Backend}
    {Phase 1 task} :be1, {YYYY-MM-DD}, {n}d
    {Phase 2 task} :be2, after be1, {n}d

    section {Frontend}
    {Phase 2 task} :fe1, after be1, {n}d

    section {QA}
    {Testing} :qa1, after fe1, {n}d
```

## Milestones

| Milestone | Target | Depends on |
|---|---|---|
| {MVP ready} | {YYYY-MM-DD} | {Phase 3} |
