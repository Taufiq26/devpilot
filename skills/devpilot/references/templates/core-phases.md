# Execution Phases

<!-- Status values: pending / in progress / done.
     Check a task's box IMMEDIATELY when it completes, never batched.
     Each completed phase gets a git commit checkpoint.

     Split layout: when config.md's Phase file layout is `split`, this
     content lives in docs/core/phases/00-overview.md (Milestones + Overview
     tables + status only — no per-phase task detail) and each `## Phase N`
     section below becomes its own docs/core/phases/phase-{N}.md instead.
     Same structure either way, just distributed across files. -->

## Milestones

<!-- Optional — omit this whole section entirely for small/simple projects.
     Include it when phases group into clearly separable modules/subsystems,
     or when the phase count is large enough that a flat list stops being
     scannable (see SKILL.md Global rules). -->
| Milestone | Phases | Status |
|---|---|---|
| {M1 — name} | {1–4} | {pending} |

## Overview

| Phase | Name | Status | Checkpoint commit |
|---|---|---|---|
| 1 | {name} | pending | |

## Phase 1 — {name} `[pending]`

**Goal:** {what this phase delivers}
**Depends on:** {— / Phase N}

- [ ] 1.1 {task}
- [ ] 1.2 {task}

## Phase 2 — {name} `[pending]`

**Goal:** {…}
**Depends on:** Phase 1

- [ ] 2.1 {task}
