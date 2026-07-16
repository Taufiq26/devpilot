# Execution Phases

## Overview

| Phase | Name | Status | Checkpoint commit |
|---|---|---|---|
| 1 | Project scaffold & database | done | `feat: phase 1 — scaffold & database` |
| 2 | Order entry | in progress | |
| 3 | Payment | pending | |

## Phase 1 — Project scaffold & database `[done]`

**Goal:** Runnable Laravel + Vue skeleton with migrated schema
**Depends on:** —

- [x] 1.1 Scaffold Laravel 11 + Vue 3 project
- [x] 1.2 Create migrations: tables, menu_items, orders, order_items
- [x] 1.3 Seeders for menu items

## Phase 2 — Order entry `[in progress]`

**Goal:** Cashier can create and amend table orders
**Depends on:** Phase 1

- [x] 2.1 Order CRUD endpoints
- [x] 2.2 Order entry screen (table grid, menu picker)
- [ ] 2.3 Order amendment & void flow
- [ ] 2.4 Feature tests for order flows

## Phase 3 — Payment `[pending]`

**Goal:** Cash & QRIS settlement with printable receipt
**Depends on:** Phase 2

- [ ] 3.1 Payment endpoints + QRIS integration
- [ ] 3.2 Payment screen & receipt template
- [ ] 3.3 End-to-end payment tests
