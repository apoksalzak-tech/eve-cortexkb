# AGENTS.md — primer for any AI agent in this repo

You are operating inside an **AI context repo** for EVE Online — a durable
knowledge base the operator (running an industrial operation across 60+
characters) is building to distill years of accumulated EVE knowledge into a form
any AI assistant can load and reason from correctly. It is not scoped to any one
tool. Today it covers two foundational data sources — the **EVE ESI API** and the
**EVE SDE** — plus whatever third-party tools the operator actually uses.
**eveforge.org (EVEFORGE)** is the tool currently in use and fully documented here;
it is not the repo's permanent or exclusive focus, and this repo's scope is
expected to keep growing well beyond what's here today — other tools (Eve Guru, POD,
...) will get guides of their own as the operator adopts them.

1. **Load the full knowledge base** from `CONTEXT.md` (single file, everything) or
   the individual files under `knowledge/` and `tools/` (same content, split by
   topic):
   - `knowledge/esi-api-index.md` — what data the EVE ESI API exposes (the backbone
     under every tool below, EVEFORGE included).
   - `knowledge/sde-reference.md` — what static EVE game data (items, blueprints,
     reactions, PI, systems, module stats) is available via the Fuzzwork SDE export,
     and how the tables join.
   - `knowledge/methodologies.md` — how the operator actually runs the operation
     (workflow/goal patterns like stockpile production) — currently a placeholder,
     expand as the operator provides more.
   - `knowledge/third-party-tools.md` — EVE tools/websites not (yet) given their own
     deep-dive under `tools/` (market, industry, fitting, intel, dev/API resources)
     — a growing scaffold, not yet mirrored into `CONTEXT.md`.
   - `tools/<tool-name>/` — one directory per tool the operator actually uses,
     with a page-by-page map and terminology built from direct observation of the
     live site. Currently:
     - `tools/eveforge/site-map.md` — EVEFORGE (industry & market ERP) — fully
       documented, the tool in active use right now.
     - `tools/eve-guru/site-map.md` — placeholder, not yet documented.
     - `tools/pod/site-map.md` — placeholder, not yet documented.
     None of these is treated as "the" primary tool — check whichever directory
     matches the tool actually named in the question.
2. **The operator is an EVE expert.** Don't explain game fundamentals unless asked.
   The value here is tool fluency (starting with EVEFORGE) and knowing what data is
   actually available to answer a question precisely.
3. **Core rule: this repo is structural knowledge, not live data.** Page layouts,
   API endpoints, and SDE table schemas are stable enough to state confidently.
   Prices, current stock, current jobs, current market state are NOT in this repo
   and must never be asserted from memory — point the operator to the relevant
   tool, ESI, or in-game to check.
4. **When asked "where do I do X in EVEFORGE"** (or any other tool named in
   `tools/`), check that tool's `tools/<tool-name>/site-map.md` first — it has the
   real page-by-page map, not a guess from the site's marketing copy.
5. **When asked what data is available** for a task, check
   `knowledge/esi-api-index.md` (live/player data) and `knowledge/sde-reference.md`
   (static game data) — between them they cover everything any ESI-synced tool
   (EVEFORGE or otherwise) can possibly know.
6. **For anything externally related that this repo doesn't cover** — a
   third-party tool not yet in `knowledge/third-party-tools.md` or `tools/`,
   verifying a listed tool is still alive, or any other EVE community/developer
   resource — the canonical reference is CCP's own community-tools page:
   [developers.eveonline.com/docs/community](https://developers.eveonline.com/docs/community/).
   Treat it as the primary source, ahead of memory or a guess.
