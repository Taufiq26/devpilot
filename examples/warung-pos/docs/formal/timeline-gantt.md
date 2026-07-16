<!-- GENERATED from docs/core/ on 2026-07-02 by devpilot — do not hand-edit.
     Regenerate with /devpilot docs gantt. -->
# Project Timeline — warung-pos

> Generated 2026-07-02 from docs/core/. Durations derive from mandays in
> features.md and phase order in phases.md. Estimates, not commitments.

## Assumptions

| Assumption | Value |
|---|---|
| Working days | Mon–Fri |
| Team size per division | 1 person each |
| Start date | 2026-07-06 |

## Gantt chart

```mermaid
gantt
    title warung-pos — Delivery Plan
    dateFormat YYYY-MM-DD
    axisFormat %d %b

    section Backend
    Scaffold & database   :be1, 2026-07-06, 3d
    Order endpoints       :be2, after be1, 3d
    Payment & QRIS        :be3, after be2, 4d

    section Frontend
    Order entry screen    :fe1, after be1, 4d
    Payment screen        :fe2, after be3, 3d

    section UI/UX
    POS flow design       :ux1, 2026-07-06, 3d

    section QA
    Order flow tests      :qa1, after fe1, 2d
    Payment E2E tests     :qa2, after fe2, 2d
```

## Milestones

| Milestone | Target | Depends on |
|---|---|---|
| Orders live (internal) | 2026-07-15 | Phase 2 |
| MVP ready | 2026-07-22 | Phase 3 |
