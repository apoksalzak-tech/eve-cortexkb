---
title: Third-Party EVE Online Tools & Websites
status: scaffold — starter list, operator will expand
sources:
  - https://github.com/devfleet/awesome-eve (archived, June 2016–2025)
  - https://everookies.com/3rd-party-tools/3rd-party-websites-eve-online/
  - https://developers.eveonline.com/docs/community/ (current official CCP list)
compiled: 2026-06-30
---

# Third-Party EVE Online Tools & Websites

A reference list of tools outside EVEFORGE that come up in industry/market
workflows — market data, blueprint calculators, fitting, intel, and the
developer-facing ESI/SDE tooling. This is a **starting scaffold**, not exhaustive;
the operator will extend it over time.

## Source notes (read before trusting anything below blindly)

- `awesome-eve` (devfleet/awesome-eve on GitHub) was a community-maintained list
  from 2016 onward, **archived** once CCP launched an official listing. Some
  entries may be dead/abandoned — it says so itself ("Eve apps and websites come
  and go"). Verify a tool is still live before relying on it.
- The **current canonical, CCP-maintained list** is
  [developers.eveonline.com/docs/community](https://developers.eveonline.com/docs/community/)
  — check there for anything not in this file, or if something below turns out
  to be dead.
- Everything here is a **third-party, non-EVEFORGE tool**. None of this is
  ESI-authenticated by default in this repo's context — treat each one as its
  own separate login/data-sharing decision.

## Market & Industry

- **[EVE Tycoon](https://evetycoon.com/)** — Profit tracking and order
  management application, with a market browser.
- **[Fuzzworks](https://www.fuzzwork.co.uk)** — Already a primary source in this
  repo (`sde-reference.md`) for SDE exports; also runs industry calculators,
  stats, a typename→typeID API, and route planners.
  [GitHub](https://github.com/fuzzysteve/)
- **[Adam4EVE](https://www.adam4eve.eu)** — Large collection of market/industry/PI
  tools and statistics: margin finder, market trends, contract prices/profit,
  manufacturing profitability, reprocessing profitability, material influence,
  cost index history, PI profitability and chains, and a station/structure ID
  lookup (`info_stations.php`) — useful for verifying structure IDs like the
  C-J6MT Keepstar entry in `esi-api-index.md`.
- **[Ravworks](https://ravworks.com/)** — Free market/industry planner in the
  spirit of Fuzzwork's blueprint calculator + Evepraisal's ease of use. Handles
  recursive material planning (put in end products, get full shopping list) and
  an invention tab comparing decryptors/ancient relics (successes, cost/success,
  expected profit).
- **[eve-industry.org](http://eve-industry.org/calc/)** ("EVE Industry -
  Blueprint Calculator") — Manufacturing/invention/industrial profitability
  calculator; factors skill levels, facilities, and market prices (sourced from
  EVEMarketer) into material cost, time, and profit.
- **[EVE Cookbook](https://www.evecookbook.com)** — Industry build calculator
  with reactions support and detailed build steps.
- **[EVE Market Browser](http://evemarketbrowser.com/)** — In-game-style market
  clone: real-time orders and price history from anywhere in New Eden.
- **[EVE Trade](https://evetrade.space)** — Station-to-station and
  region-to-region trading/hauling deal finder (public citadels included).
- **[Abyssal Market](https://mutaplasmid.space/)** / **[MutaMarket](https://mutamarket.com/)**
  — Appraising/trading abyssal (mutated) modules off the contract market — pairs
  with the Abyssal/Mutated data-conflict note in `sde-reference.md` (their stats
  aren't in the static SDE).
- **evebook.com** — Appeared in a search alongside other EVE tool listings;
  **purpose not confirmed** — the site is a client-rendered app that redirects
  to a `/lander` page with no content retrievable without full JS execution.
  Flag as unverified until checked directly in a browser; don't assume what it
  does.

## Reference / lookup

- **[EVE Ref](https://everef.net/)** — Full item database, stats, and a combined
  SDE+ESI+Hoboleaks dataset
  ([EVE Reference Data](https://docs.everef.net/datasets/reference-data.html)),
  available as an OpenAPI and download — a broader alternative/companion to the
  raw SDE + ESI split this repo documents separately.
- **[Adam4EVE station/structure IDs](https://www.adam4eve.eu/info_stations.php)**
  — searchable station/structure name↔ID lookup; a practical way to verify a
  structure_id like the one recorded for the operator's C-J6MT Keepstar.

## Fitting

- **[Pyfa](https://github.com/pyfa-org/Pyfa/releases)** — Create, experiment
  with, and save ship fittings offline.
- **[EVEShip.fit](https://eveship.fit)** — View/create/share fits online.
- **[EVE Workbench](https://www.eveworkbench.com/)** — Browse market, build haul
  routes, browse/share fittings (also referenced by everookies.com via the EVE
  University wiki).

## Intel & Maps

- **[zKillboard](https://zkillboard.com)** — Killmail presentation and
  statistical aggregation; the de facto killboard.
- **[EVE-KILL](https://eve-kill.com)** — Alternative killmail tracking site.
- **[Dotlan](http://evemaps.dotlan.net/)** — Maps, jump/route planning, system
  activity and kill data.
- **[Local Threat](https://localthreat.xyz)** — Aggregates zKillboard data for
  every pilot currently in local chat, on one page.
- **[EveWho](http://evewho.com)** — Lists members of a corp/alliance
  out-of-game.
- **[EVE Gate Camp Check](http://eve-gatecheck.space/eve/)** — Flags likely gate
  camps along a route.

## Community & Events

- **[EVE University Wiki](https://wiki.eveuniversity.org/Main_Page)** — New
  player resource, class schedule, and a large third-party-tools page of its
  own: [wiki.eveuniversity.org/Third-party_tools](https://wiki.eveuniversity.org/Third-party_tools).
- **Eve Online Events** — community events/fleet calendar (site referenced by
  everookies.com; verify current URL before use, not independently confirmed
  here).

## Auth / corp-management (for context, not currently used by this operator)

- **[Alliance Auth](https://gitlab.com/allianceauth/allianceauth)** — Auth
  system tying Discord/Mumble/forums/etc. access to in-game corp/alliance
  membership. The dominant self-hosted auth framework in EVE null-sec/large
  corp circles.
- **[SeAT](https://github.com/eveseat/seat)** — PHP-based EVE corp management
  tool (assets, industry, HR).

## Developer / API resources

- **[ESI Swagger UI](https://esi.evetech.net/ui/)** — Interactive ESI docs
  (already referenced in `esi-api-index.md`).
- **[EVE Reference Data (EVE Ref)](https://docs.everef.net/datasets/reference-data.html)**
  — combined SDE+ESI+Hoboleaks dataset, OpenAPI + download.
  **Hoboleaks** specifically fills gaps the SDE doesn't cover (e.g. which
  industry rig affects which module) — worth knowing about if `sde-reference.md`
  ever comes up short: [sde.hoboleaks.space](https://sde.hoboleaks.space/).
- **[developers.eveonline.com/docs/community](https://developers.eveonline.com/docs/community/)**
  — CCP's own current, maintained community-tools listing — the first place to
  check for anything not covered above.

## TODO (operator to fill in later)

This file is a scaffold. Not yet mirrored into `CONTEXT.md` (kept as a separate
file for now, given the size this list could grow to) — sync it in once the list
stabilizes. Categories worth adding next: PI planners beyond EVEFORGE, invention
decryptor comparison tools beyond Ravworks, reaction-chain planners, and
corp/alliance-specific tools the operator's own corp already uses.
