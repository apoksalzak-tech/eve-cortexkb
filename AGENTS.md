# AGENTS.md — primer for any AI agent in this repo

You are operating inside a knowledge base for an EVE Online industrial operation
run across 60+ characters, using **eveforge.org (EVEFORGE)** as the industry &
market ERP.

1. **Load the full knowledge base** from `CONTEXT.md` (single file, everything) or
   the individual files under `knowledge/` (same content, split by topic):
   - `knowledge/eveforge-site.md` — what EVEFORGE's pages/tabs/terminology actually
     are, from direct observation of the live site.
   - `knowledge/esi-api-index.md` — what data the EVE ESI API exposes (the backbone
     under EVEFORGE and any other third-party tool).
   - `knowledge/sde-reference.md` — what static EVE game data (items, blueprints,
     reactions, PI, systems, module stats) is available via the Fuzzwork SDE export,
     and how the tables join.
   - `knowledge/methodologies.md` — how the operator actually runs the operation
     (workflow/goal patterns like stockpile production) — currently a placeholder,
     expand as the operator provides more.
   - `knowledge/third-party-tools.md` — non-EVEFORGE EVE tools/websites (market,
     industry, fitting, intel, dev/API resources) — a growing scaffold, not yet
     mirrored into `CONTEXT.md`.
2. **The operator is an EVE expert.** Don't explain game fundamentals unless asked.
   The value here is EVEFORGE tool fluency and knowing what data is actually
   available to answer a question precisely.
3. **Core rule: this repo is structural knowledge, not live data.** Page layouts,
   API endpoints, and SDE table schemas are stable enough to state confidently.
   Prices, current stock, current jobs, current market state are NOT in this repo
   and must never be asserted from memory — point the operator to EVEFORGE, ESI, or
   in-game to check.
4. **When asked "where do I do X in EVEFORGE"** or "does EVEFORGE have Y", check
   `knowledge/eveforge-site.md` first — it has the real page-by-page map, not a
   guess from the site's marketing copy.
5. **When asked what data is available** for a task, check
   `knowledge/esi-api-index.md` (live/player data) and `knowledge/sde-reference.md`
   (static game data) — between them they cover everything EVEFORGE itself can
   possibly know.
6. **For anything externally related that this repo doesn't cover** — a
   third-party tool not yet in `knowledge/third-party-tools.md`, verifying a
   listed tool is still alive, or any other EVE community/developer resource —
   the canonical reference is CCP's own community-tools page:
   [developers.eveonline.com/docs/community](https://developers.eveonline.com/docs/community/).
   Treat it as the primary source, ahead of memory or a guess.
7. **If you have HTTP fetch access and need a current price for one or a few
   specific items** (not a full market sync), `knowledge/market-data-lookups.md`
   has three verified, no-auth-required endpoints (Fuzzwork, EVE Tycoon,
   Goonmetrics) that return an aggregated current price in one call — including
   one that covers the operator's own C-J6MT home market. Use one of those
   rather than stating a price from memory, per rule 3.
