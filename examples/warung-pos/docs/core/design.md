# Design — UI/UX

## Design direction

| Aspect | Choice | Note |
|---|---|---|
| Visual style | Minimalist & clean | Cashiers work fast under pressure; low visual noise, content-focused |
| Color mode | Light only | Bright counter environment; dark mode deferred (backlog candidate) |
| Layout density | Compact | Order grid and menu picker must fit one screen without scrolling |
| Target devices | Tablet-first, desktop-capable | POS runs on a counter tablet; owner reports viewed on desktop |

## Color palette

| Role | Color name | Hex |
|---|---|---|
| Primary | Warm orange | #EA580C |
| Secondary | Deep charcoal | #1F2937 |
| Accent | Fresh green | #16A34A |
| Background | Warm white | #FAFAF9 |
| Text | Near black | #111827 |
| Success / Warning / Error | Green / Amber / Red | #16A34A / #D97706 / #DC2626 |

## Typography

| Role | Font | Notes (size/weight) |
|---|---|---|
| Headings | Inter | 600 weight; 20–28px |
| Body | Inter | 400 weight; 16px minimum for cashier readability |

## References & inspiration

| Reference | What to take from it |
|---|---|
| Moka POS | One-screen order entry, large tappable menu tiles |
| Square POS | Clear settlement flow, uncluttered payment screen |

## Components & patterns

| Element | Convention |
|---|---|
| UI library / framework | Tailwind CSS + Headless UI (Vue) |
| Icon set | Heroicons outline |
| Corner radius | Slightly rounded (8px) |
| Shadows & depth | Subtle — single elevation level for modals only |
| Spacing scale | Tailwind default (4px base) |

## Tone & UI copy

Casual and direct, Indonesian UI copy ("Bayar", "Tambah pesanan") — document
language remains English.

## Accessibility baseline

- Contrast: WCAG AA minimum for all text
- Touch targets: ≥ 48px — cashiers tap quickly on a tablet
- Keyboard navigation: payment and order screens fully operable via keyboard

## Per-screen notes

| Screen | Design note |
|---|---|
| Order entry | Table grid left, menu picker right; running total always visible |
| Payment | Full-screen takeover; QRIS code rendered at ≥ 300px for scanning |
