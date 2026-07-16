<!-- GENERATED from docs/core/ on 2026-07-16 by devpilot — do not hand-edit.
     Regenerate with /devpilot docs srs. -->
# Software Requirements Specification — warung-pos

| | |
|---|---|
| Version | 1 (generated 2026-07-16) |
| Source | docs/core/ |
| Status | Generated document |

## 1. Introduction

### 1.1 Purpose
Specify the functional and non-functional requirements of warung-pos, a
point-of-sale web application for small eateries, as a reference for the
development team and for acceptance testing.

### 1.2 Scope
Covers order entry, payment settlement (cash & QRIS) with receipt printing,
and ingredient stock tracking. Excludes multi-branch support, kitchen
display, and accounting.

### 1.3 Definitions & abbreviations
| Term | Meaning |
|---|---|
| Warung | Small Indonesian family-owned eatery |
| QRIS | Quick Response Code Indonesian Standard — national QR payment standard |
| POS | Point of Sale |

## 2. Overall description

### 2.1 Product perspective
Single-tenant web application: Vue 3 SPA served against a Laravel 11 REST
API with a PostgreSQL 16 database, deployed as Docker containers on a single
host. Receipt printing targets a network ESC/POS printer.

### 2.2 User classes
- **Cashier** — creates/amends orders, settles payments; primary daily user.
- **Owner** — reviews revenue and stock levels; administers menu items.

### 2.3 Operating environment
Counter tablet or desktop running a current Chromium-based browser; server
side Docker on Linux; local network access to the receipt printer.

### 2.4 Constraints
- QRIS payments must go through a licensed payment aggregator.
- The POS must remain usable on a 10-inch tablet in landscape.
- Single cashier session at a time (MVP).

## 3. Functional requirements

| ID | Requirement | Related feature |
|---|---|---|
| FR-1 | Cashier can create an order attached to a table and add/remove menu items | Order entry |
| FR-2 | Cashier can amend or void an unpaid order with a recorded reason | Order entry |
| FR-3 | Cashier can settle an order by cash (with change calculation) or QRIS | Payment |
| FR-4 | System prints a receipt on successful settlement | Payment |
| FR-5 | System decrements ingredient stock per recipe when an order is settled | Stock tracking |
| FR-6 | System alerts the owner when an ingredient falls below its threshold | Stock tracking |

## 4. Non-functional requirements

| ID | Category | Requirement |
|---|---|---|
| NFR-1 | Performance | Order entry interactions respond within 200 ms on the counter tablet |
| NFR-2 | Reliability | A failed QRIS callback never marks an order as paid |
| NFR-3 | Security | All endpoints require an authenticated session; payments are authorized per role |
| NFR-4 | Usability | Core cashier flows operable with touch targets ≥ 48px |

## 5. External interfaces

### 5.1 User interface

Minimalist & clean visual style, light mode only, compact layout —
tablet-first with desktop-capable report views. Primary palette
#EA580C / #1F2937 / #16A34A on warm-white #FAFAF9; Inter typography,
body ≥ 16px. Accessibility baseline: WCAG AA contrast, ≥ 48px touch
targets, keyboard-operable order and payment screens. Full detail in
docs/core/design.md.

### 5.2 APIs

REST API under `/api/v1`: order CRUD (`/orders`), settlement
(`/orders/{id}/payments`), menu (`/menu-items`), and stock
(`/ingredients`, `/stock-alerts`). Outbound integration: payment
aggregator's QRIS charge + webhook callback. Full contract in
docs/core/api-contract.md.

## 6. Data requirements

Core entities: `tables`, `menu_items`, `orders`, `order_items`,
`payments`, `ingredients`, `recipe_items` (menu item ↔ ingredient usage).
Orders are immutable after settlement; amendments before settlement are
versioned via order events. Schema detail and ERD in docs/core/database.md.
