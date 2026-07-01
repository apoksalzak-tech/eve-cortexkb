---
title: EVEFORGE Site Map & Terminology
status: fully documented — the tool currently in use by the operator (not treated as permanent/primary)
source: https://eveforge.org
observed: 2026-06-30
---

# EVEFORGE Site Map & Terminology

## What EVEFORGE is

A free, browser-based EVE Online **industry & market ERP**, synced to your
characters via ESI. Single-page app at `https://eveforge.org`. Version observed:
**v0.8.8 beta**. Author credit in-app: "ELFX" (linked to a zKillboard character
page). Copyright footer: "© 2026 Fenris Creations." "Help & Docs" in the footer
links to `github.com/sa1rex/eve-forge`, which returned 404 (private or renamed) at
time of writing — not a usable source.

## How this map was built (and its limits)

EVEFORGE is a client-rendered SPA — the server sends an empty shell and JavaScript
builds the page, so a plain HTTP fetch returns nothing useful. This map was built by
**rendering every route in a real headless browser** on 2026-06-30. No EVE character
was linked during observation, so every screen shown here is in its **empty /
onboarding state** — table columns and tab names are accurate, but example data
(numbers, item names) were not observed being live-populated except on pages that
show public/shared data with no character required (Scanner, Reactions, PI, LP
Calculator — those numbers ARE live but will have moved on by the time you read
this; treat them as illustrative of *what the page shows*, not current values).

One transient state was observed on the Characters page: `"Server unreachable —
showing cached data."` Noted for completeness — likely a one-off backend hiccup
during this scrape, not a known standing issue.

## Top navigation (5 groups)

The left nav is organized into five collapsible sections. This is the primary
mental model for "where do I go to do X":

```
COMMAND CENTER   Dashboard · Net Worth · Smart Queue
PLAN             Production · Inventory · Jobs
DISCOVER         Market · Profit Optimizer · Reactions · Invention ·
                 T3 Production · Planetary · Scanner
TOOLS            Item Explorer · BOM Calculator · Build by Fit ·
                 Resource Calc · LP Calculator
ACCOUNT          Characters · Assets · Corporation · Workspaces · Settings
```

Every page also has a persistent header: sync status ("Synced: —" until you
refresh), a global **Refresh All** button (pulls fresh ESI data), and a
**Best Practices & Tips** control present on most tool pages (in-app guidance —
worth checking per-page inside the actual UI; content wasn't reliably extractable
by automation here).

## Page-by-page detail

### Command Center

**Dashboard** (`/dashboard`) — Home screen. Shows Total Wallet, Active Jobs, Sell
Orders, Weekly P&L, Stock Health at a glance. New-user onboarding is literally three
steps, shown as cards:
1. **Configure your structure & skills** — set engineering complex, rig bonuses,
   skill levels (this lives in Settings → Facilities).
2. **Add items to manufacture** — build your **Item Master**, "the list of products
   you want to keep in stock on the market" (this lives in Production).
3. **Check profitability** — analyze margins and ISK/hr (this lives in Market /
   Profit Optimizer).
Also shows Active Industry Jobs and Market Orders (sell/buy split with total
investment) once characters are linked.

**Net Worth** (`/networth`) — net worth tracking over time (empty-state only
observed; no distinguishing copy rendered without linked characters).

**Smart Queue** (`/queue`) — "Optimal slot allocation by ISK/hr." This is
EVEFORGE's **stock-target-driven production scheduler**: you set inventory targets
per item (stock types), and it computes what to build against the deficit, not
against a manually-planned run list. Shows MFG ISK/hr, RXN ISK/hr, Combined ISK/hr,
Active Jobs, Manufacturing Slots (x/max) and Reaction Slots (x/max) as headline
stats. Has a "Fill Timeframe" selector (None / 6h / 12h / 1d / 2d / 3d / 7d / 14d /
Custom) that controls how far ahead it plans to cover the deficit. Three
sub-queues: **Manufacturing Queue**, **Reaction Queue** (reaction intermediates
needed to feed the manufacturing queue), **Transport Queue** (moves materials to
wherever the job is slotted, from other stock locations). Explicitly the "stockpile
to target, build to cover the deficit" workflow — see `methodologies.md`.

### Plan

**Production** (`/productionhub`, also reachable at `/stocktypes` — same page)
— tabs: **Item Master · Pipeline · Kanban Board · Materials · Shopping · Raw
Material · Readiness · Blueprints · BPO Checklist · BPO Collection**. Item Master is
the core list of tracked products, with per-item Home/Jita/Backup price columns,
ME/TE, Runs, Total, Profit %, Profit, Vol/D, and an inline chart. Supports "Quick
Plan" presets and CSV export/import.

**Inventory** (`/inventoryhub`) — tabs: **Stock Levels · Edit Stock · Price
Overrides · Build/Buy Rules · Restock Alerts · Idle Capital · Locations · Move
Planner · Transport**. Centerpiece feature: **Stockpiles** — "named inventory
target containers for tracking stock health." You create named Stockpiles, each
with target quantities; the dashboard shows count of stockpiles, average stock %,
total items, and deficit value. This is the direct in-app implementation of a
target-based ("stockpile production") inventory model, as opposed to tracking each
build individually.

**Jobs** (`/jobshub`) — tabs: **Manufacturing · Reactions · Timeline · Industry ·
Research**. Two main tables: **Manufacturing Jobs** (Item, Group, Runs, Output,
Current Inventory, Elsewhere, Build Cost, Unit Price, Job Cost, Margin, ISK/hr, Time
(TE-adjusted), Action) and **Invention Jobs** (T2 Item, T1 BPC, Datacores,
Decryptor, Total Attempts, Probability, Expected BPCs, Runs/BPC, ME/TE,
Cost/Attempt, Total Cost). Also shows an Investment Summary (Material Costs, Job
Installation, Invention, Grand Total) and a large category filter list (Ships,
Components, Modules & Equipment, Ammo & Charges, Drones & Fighters, Structures,
Fuel Blocks, T3 Ships/Subsystems, Rigs by tier, etc.) — this filter list is a good
proxy for how EVEFORGE buckets the whole EVE item catalog for industry purposes.

### Discover

**Market** — three related routes surface the same underlying "Market" interface
with many tabs, differing by which tab loads by default:
- `/market` defaults to a **Listings** view: stock targets vs actual — columns
  Item, Inventory, Home Target, Jita Target, To List, Stock Level, Jita Price, Home
  Price, Import Price, Build Price. This is the sell-side counterpart to
  Stockpiles: are you carrying enough finished stock to list at each hub.
- `/margins` and `/markethub` both default to a **Profitability** view — headline
  stats: Total Positions, Avg Margin, Best ISK/hr, Daily Revenue, Portfolio ROI,
  Capital Required, Win Rate, Top Score. Table columns: Item, Category, Total Cost,
  Jita Price, Profit, Build Margin, ROI, ISK/hr, Score, Listed, Med Vol/D, Turnover,
  Liq, Trend.
- Full tab set available under Market: **Dashboard · Profitability · Order Control ·
  Technical Analysis · Analyzer · Competitors · Portfolio & P&L · Market Depth ·
  BPC Contract Margin · BPC Contract Prices · BPC Watch · Listings · Alerts ·
  Financial · Arbitrage · Stock Checker.**

**Profit Optimizer** — nav label for the Profitability-flavored entry into Market
above (see Market).

**Reactions** (`/reactions`) — "Profitability analysis for all EVE reaction types —
Composite, Hybrid, Boosters, Neurolink, and Mineral." Category counts observed:
Composite (41), Hybrid (9), Boosters (32), Neurolink (12), Mineral (8). Sub-tabs:
**Scheduler · Alchemy · Ledger**. Reactions are split into **Simple Reactions**
(single-stage, raw moon goo → intermediate) and **Complex Reactions** (chained,
intermediate → higher-tier material, e.g. feeding T2/T3 production), each with its
own leaderboard table (Input Cost, Tax, Revenue, Profit, Margin, ROI, ISK/hr, Vol,
Liq, Trend). Complex reactions additionally show **Chain ISK/hr** (profitability of
the whole chain, not just this step).

**Invention** (`/inventionhub`) — "Analyze T2 invention opportunities and compare
decryptors." Sub-tabs: **Analyzer · Overview · Copy Planner**.

**T3 Production** (`/t3hub`) — "Strategic Cruisers, Tactical Destroyers, Subsystems
& Reverse Engineering." Sub-tabs: **Overview · Cruiser Builder · Destroyer Builder ·
Subsystem Browser · Reverse Engineering**.

**Planetary** (`/pi`) — "PI Planner: plan PI production, find optimal planets,
design colony setups, explore all production chains." Sub-tabs: **Profitability ·
Catalog · Production Plan · Planet Finder · Find Systems · Colonies**. Profitability
table filters by tier (All/P1/P2/P3/P4), columns: Item, Product Tier, Build Cost,
Sell Price, Profit, Margin, ROI, ISK/hr, Jita Vol, Liq, Trend.

**Scanner** (`/scanner`) — "Market Scanner: find cross-hub arbitrage opportunities
across EVE trade hubs." Filters: Buy From / Sell To (per-hub or All Hubs, Jita /
Amarr / Dodixie / Rens / Hek), Sell/Buy Orders toggle, Search, Min Profit ISK, Min
Profit %, Min Volume, Max Buy Price, Category. Table: Item, Buy Price, Sell Price,
Profit/Unit, Profit %, Avail Vol, Est. Profit. Also links out to **Evepraisal**.

### Tools

**Item Explorer** (`/explorer`) — "Browse manufacturable items by category, or
search for any item with a blueprint." Left-panel category counts observed (useful
as a rough census of EVE's manufacturable catalog as EVEFORGE buckets it): Ship
405, Module 2051, Structure Module 337, Charge 521, Fighter 94, Drone 128, Implant
188, Deployable 39, Material 123, Commodity 497, Structure 28, Starbase 192,
Subsystem 48, Infrastructure Upgrades 77, Asteroid 60, Celestial 26, Sovereignty
Structures 3, Orbitals 2, Accessories 2, Special Edition Assets 1.

**BOM Calculator** (`/bom`) — recursive bill-of-materials tree for one or more
output items (Item, Quantity, ME, TE inputs), with a "View Tree" toggle.

**Build by Fit** (`/fit-importer`) — "Paste an EFT fit to generate a Bill of
Materials and add items to your production queue." Supports EVE in-game fit export,
EFT, Pyfa, and DHA formats; handles multiplied items (x3), drones, cargo, fighters.

**Resource Calc** (`/raw`) — "Convert materials to raw forms and calculate
reprocessing yields." Sub-tabs: **Materials to Raw · Reprocessing · Raw to
Compressed**. Has an "Operating Zone" selector (Hi-Sec / Low-Sec / Null-WH) —
reprocessing yield depends on security-space-driven station/structure bonuses.

**LP Calculator** (`/lpcalculator`) — "Find the best ISK per Loyalty Point across
all NPC corporation stores." 31,847 offers from 181 corporations observed. Sub-tabs:
**Top Offers · Corp Browser · Item Search · My LP**. Filters by faction (Gallente
Federation, Amarr Empire, Caldari State, Minmatar Republic, Serpentis, Angels, ...),
sell method (Sell Order / Instant Sell), Min ISK/LP, Min Volume, LP Acq. Cost.
Columns: Item, Corporation, Qty, LP Cost, ISK Cost, Required Items, Sell Price,
Profit, ISK/LP, Volume.

### Account

**Characters** (`/charactershub`) — "Link your EVE Online characters to
automatically sync assets, orders, industry jobs, skills, and blueprints." Sub-tabs:
**Overview · Skills · Wallet · Clones · Employment · Standings · API Status**. Uses
EVE SSO (OAuth2) — password never shared; scoped, time-limited, revocable tokens.
**ESI scopes requested for a normal character**, verbatim: *Read character assets,
Read market orders, Read industry jobs, Read wallet balance, Read skill queue, Read
blueprints, Read mining ledger.* A separate "+ Add Corp Director" flow requests
**additional corp-level scopes** for corp assets, jobs, wallet, and blueprints (see
Corporation Hub below).

**Assets** (`/assets`) — "Browse, track, and analyze assets across all linked
characters." Sub-tabs: **Browse · Blueprints · Orders · Mining · Contracts · LP ·
Risk · Diff**. Browse columns: Name, Character, Location, Category, Qty, Price,
Total Value, with hangar/location filters (Hide Fitted, Hide Ship Cargo) and preset
saving.

**Corporation** (`/corphub`) — "Corporation Hub: corporation-level data from linked
directors." Requires linking a character with the in-game Director role to unlock —
directors get extra ESI scopes for corp-level assets, jobs, wallet, and blueprints.

**Workspaces** (`/workspaces`) — multi-character/corp **collaboration** layer, mostly
gated behind login. Sub-tabs observed: **Dashboard · My Tasks · Calendar · Timeline
· Transfers · BPC Pool · Profit · Snapshots · Digests · Invites · Members &
Settings.** This looks like the natural place for roster-level, multi-alt
coordination (task assignment, BPC sharing pool, shared profit tracking) — worth a
deeper look once characters are linked, and directly relevant to a 60-character
operation.

**Settings** (`/settingshub`) — "Configure application settings, bonuses, and
logistics." Sub-tabs: **Facilities · Market · Blueprints · Skills · Production ·
Notifications**. Facilities/Production Setup is where you define **structures**
(each with independent location, rigs, and cost index — mark one Primary per
purpose), plus **SCC Industry Tax** (flat surcharge on item value, not reduced by
structure) and **Facility Tax** (structure owner's tax, checked in-game). Supports
per-item **category overrides** (which station bonuses apply to a given item) and
"Import from GEZ." Data Management section: Export/Import Config (JSON), Reset to
Defaults, and a **Delete All My Data** danger-zone action (characters, ESI cache,
app state, session — irreversible).

### Other

**Commodity** (`/commodity`) — "Unified item detail — prices, history, margin, BOM,
and trend in one view." A single-item deep-dive page; reached by searching any item
name. Not in the left nav directly (likely linked from search results / item rows
elsewhere).

## Terminology quick-reference

| EVEFORGE term | What it means |
|---|---|
| Item Master | Your curated list of products you track/build/stock |
| Stockpile | A named inventory target container — quantity you want on hand, tracked against actual |
| Smart Queue | Auto-generated build queue driven by stock deficits, not manual planning |
| Stock Health / Stock Level | How close current inventory is to its target |
| Fill Timeframe | Planning horizon Smart Queue uses to cover a deficit |
| ISK/hr | Profit per hour of job-slot time — EVEFORGE's primary ranking metric across Market, Reactions, PI, T3 |
| Score | A composite ranking (exact formula not published in the UI) used in Profitability tables |
| Chain ISK/hr | ISK/hr for a whole multi-step reaction chain, not just one step |
| Home / Jita / Backup | Per-item price and stock columns for up to three reference locations |
| Elsewhere | Stock of an item held somewhere other than the active production hub |
| BPO Checklist / BPO Collection | Tracking which blueprint originals you own vs. still need |
| GEZ | An import format for facility settings (unexpanded acronym; verify meaning in-app) |
| Diff (Assets) | Likely an asset-change/reconciliation view — verify in-app, not observed populated |
