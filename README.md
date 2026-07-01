# eveforge-kb

A knowledge base that lets an AI chatbot actually understand the two data sources
underneath any EVE Online industry/market tool — the **EVE ESI API** and the
**EVE Online SDE** (Static Data Export) — plus the specific tools an operator uses
day to day. **[eveforge.org](https://eveforge.org)** (EVEFORGE), a free EVE Online
industry & market ERP, is the tool currently in active use and the first to get a
full deep-dive; it isn't treated as the repo's permanent or exclusive focus — more
tools get their own guides under `tools/` as the operator adopts them.

The problem this solves: a tool's layout and terminology aren't always obvious,
especially running a large multi-character operation. This repo documents what a
given site actually contains (real navigation, real page purposes, real
terminology — not guessed), what data ESI exposes, and what static game data the
SDE contains, so any AI assistant can reason about your setup correctly instead of
guessing.

## The one rule

Everything here is **structural/reference knowledge** — what pages exist, what an
API endpoint returns, what columns a data table has. It's deliberately **not** live
numbers (prices, current stock, current jobs). Structural knowledge is stable
enough to embed directly; live numbers must always come from the relevant tool,
ESI, or in-game, never from memory or from this repo.

## Primary external reference

For anything outside this repo's own structural knowledge — third-party tools,
community resources, official developer documentation — the canonical starting
point is CCP's own community-tools page:

**[developers.eveonline.com/docs/community](https://developers.eveonline.com/docs/community/)**

It supersedes the now-archived `devfleet/awesome-eve` list that
`knowledge/third-party-tools.md` was originally compiled from. Treat it as the
first place to check when indexing new tools into this repo later, verifying
whether a tool in `third-party-tools.md` is still alive, or resolving anything
externally related that this repo doesn't cover yet.

## Layout

```
CONTEXT.md                    single self-contained file — everything below, one paste/fetch
knowledge/
  esi-api-index.md            ESI API endpoint index (what data is available)
  sde-reference.md            EVE SDE table reference (Fuzzwork export)
  methodologies.md            operator workflow/goal patterns (placeholder, WIP)
  third-party-tools.md        EVE tools/websites not (yet) given their own tools/ guide (starter scaffold)
tools/
  eveforge/site-map.md        EVEFORGE site map & terminology — fully documented, tool in active use
  eve-guru/site-map.md        placeholder — not yet documented
  pod/site-map.md             placeholder — not yet documented
AGENTS.md                     universal agent primer (read first)
CLAUDE.md / GEMINI.md         thin pointers so each CLI auto-loads context
usage/how-to-query.md         exact steps for pointing each tool at this repo
```

`CONTEXT.md` and the `knowledge/`/`tools/` files carry the same content —
`CONTEXT.md` is the single-file, easy-to-paste-a-URL version; the split files are
the same material by topic for smaller-context tools or targeted editing. Keep
them in sync when either changes. **Exception:** `knowledge/third-party-tools.md`
is not yet mirrored into `CONTEXT.md` — it's a growing scaffold, sync it in once
the list stabilizes.

## Quick use

- **Any web chatbot:** paste the raw GitHub URL for `CONTEXT.md`, or paste its
  contents directly.
- **Claude CLI / Gemini CLI:** `cd` into a local clone and start the CLI — it
  auto-reads `CLAUDE.md`/`GEMINI.md` → `AGENTS.md` → the knowledge/tools files.

See `usage/how-to-query.md` for exact steps per tool.

## Caveats

- The EVEFORGE site map (`tools/eveforge/site-map.md`) was built by rendering the
  live site with no character linked — every screen is documented in its empty/
  onboarding state. Table columns and navigation are accurate; populated-data
  behavior (once you link characters) hasn't been verified yet.
- The ESI index and SDE reference are snapshots (spec v1.36 / SDE build 3409592,
  2026-06-30). Both are fairly stable but can drift after CCP patches — re-fetch
  from source if precision matters.
- `methodologies.md` is a placeholder — only "stockpile production" is sketched in
  as a seed example, cross-referenced to EVEFORGE's actual Stockpiles/Smart Queue
  features. More methodologies to be added by the operator.
- `third-party-tools.md` is a starter scaffold compiled from the (now archived)
  `devfleet/awesome-eve` list, everookies.com, and a few tools named directly by
  the operator — not exhaustive, some linked tools may be dead, and one entry
  (evebook.com) is flagged unverified. See "Primary external reference" above.
- `tools/eve-guru/` and `tools/pod/` are placeholders — no observation of those
  sites has been done yet. Fill them in following the pattern in
  `tools/eveforge/site-map.md` once the operator starts using them.
