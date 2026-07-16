<!-- GENERATED from docs/core/ on 2026-07-16 by devpilot — do not hand-edit.
     Regenerate with /devpilot docs prd. -->
# Product Requirements Document — warung-pos

| | |
|---|---|
| Version | 1 (generated 2026-07-16) |
| Source | docs/core/ |
| Status | Generated document |

## 1. Overview

warung-pos is a web-based point-of-sale application for small Indonesian
eateries (warung). It lets a cashier take table orders on a counter tablet,
settle payments by cash or QRIS with a printable receipt, and gives the owner
basic ingredient-stock visibility.

## 2. Problem statement

Small warung owners track orders on paper and stock by memory. Orders get
lost between table and kitchen, end-of-day cash rarely reconciles, and
ingredients run out mid-service without warning. Existing POS products are
priced and designed for multi-branch restaurants, not a two-person warung.

## 3. Goals & success metrics

| Goal | Metric |
|---|---|
| Replace paper order-taking | 100% of orders entered digitally within 2 weeks of go-live |
| Reconcile daily revenue | End-of-day cash/QRIS mismatch < 1% |
| Prevent stock-outs | Low-stock alert fires ≥ 1 day before an ingredient runs out |

## 4. Users & personas

- **Cashier (Sari)** — takes orders and settles payments during rush hour;
  works on a shared counter tablet; needs speed and large touch targets over
  configurability.
- **Owner (Pak Budi)** — checks revenue and stock after closing, on a laptop;
  needs clear numbers, not dashboards to configure.

## 5. Features

### Must have
- **Order entry** — cashier creates and amends orders per table (Point of Sale module)
- **Payment** — cash & QRIS settlement with receipt print (Point of Sale module)

### Should have
- **Stock tracking** — ingredient stock levels with low-stock alerts (Inventory module)

## 6. Design direction

Minimalist and clean, light mode only: cashiers work fast under pressure, so
the UI stays low-noise and compact — order grid and menu picker fit one
tablet screen without scrolling. Tablet-first with desktop-capable layouts
for the owner's reports. Full detail in docs/core/design.md.

| Aspect | Choice |
|---|---|
| Visual style | Minimalist & clean |
| Primary palette | #EA580C / #1F2937 / #16A34A |
| Color mode | Light only |
| Target devices | Tablet-first, desktop-capable |

## 7. Timeline summary

Three phases — scaffold & database, order entry, payment — planned at 31
total mandays across four divisions, targeting an MVP on 2026-07-22. Details
in timeline-gantt.md. Estimates, not commitments.

## 8. Risks & assumptions

- QRIS integration depends on the payment aggregator's sandbox availability.
- Single counter tablet assumed; concurrent multi-cashier use is out of scope
  for MVP and would affect order locking.
- Receipt printing assumes an ESC/POS-compatible printer on the local network.

## 9. Out of scope

- Multi-branch or multi-outlet support
- Kitchen display system (orders are still shouted/printed to kitchen)
- Payroll, accounting, and tax reporting
