---
title: Industrialist Methodologies
status: placeholder — to be filled in by the operator
---

# Industrialist Methodologies

This file is intentionally mostly empty. It's where different ways EVE
industrialists manage their operations get documented — not mechanics (that's
`tools/<tool-name>/site-map.md` / `esi-api-index.md` / `sde-reference.md`), but
**workflow and goal patterns**: how someone actually runs their empire day to day,
and why they chose that pattern over another.

## Why this matters

Two people with the same blueprints, the same skills, and the same EVEFORGE
account can run completely different operations because they're optimizing for
different things — some chase margin, some chase throughput, some chase zero
attention-cost. An assistant that only knows game mechanics can calculate a build;
an assistant that also knows the operator's methodology can recommend the *right*
build for how they actually work.

## Methodologies

### Stockpile production (seed example — expand this)

Instead of mapping out and building each item individually per order/need, the
industrialist maintains a **pool of ready resources/intermediates** and builds
toward **target quantities** ("stockpiles") of finished goods. Production is
triggered by *deficit against target*, not by a manually planned run list. Demand
is met from standing stock; jobs are queued to refill whatever fell below target.

This is not a hypothetical pattern for EVEFORGE users — it's a **built-in feature**:
see `archive/eveforge/site-map.md` → Inventory → **Stockpiles** ("named inventory
target containers for tracking stock health") and → Smart Queue ("optimal slot
allocation by ISK/hr... runs based on stock deficit"). An operator running this
methodology should be pointed straight at those two screens rather than a manual
BOM/queue workflow.

*(Placeholder — the operator will expand this with their actual target-setting
logic, review cadence, and how it interacts with the 60-character roster.)*

### Other methodologies (to be added)

- Per-order / made-to-order production (the opposite of stockpile production —
  build exactly what's needed when it's needed, nothing held in reserve)
- Margin-maximizing / opportunistic production (chase whatever EVEFORGE's
  Profitability or Scanner tables currently rank highest, rotate product lines)
- Vertical chain ownership (control every step from raw/reaction through
  component through final product, vs. buying intermediates on market)
- Specialized-alt / role-based roster management (see the operator's actual
  character-role breakdown once documented)

## How to add to this file

Each methodology should cover: what it optimizes for, what it deliberately
sacrifices, what EVEFORGE screens/features support it, and how it changes what a
recommendation should look like when this is the operator's active mode.
