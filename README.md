# eveforge-kb

A knowledge base that lets an AI chatbot actually understand
**[eveforge.org](https://eveforge.org)** (EVEFORGE) — a free EVE Online industry &
market ERP — plus the two data sources underneath it: the **EVE ESI API** and the
**EVE Online SDE** (Static Data Export).

The problem this solves: EVEFORGE's layout and terminology aren't always obvious,
especially running a large multi-character operation. This repo documents what the
site actually contains (real navigation, real page purposes, real terminology — not
guessed), what data ESI exposes, and what static game data the SDE contains, so any
AI assistant can reason about your setup correctly instead of guessing.

## The one rule

Everything here is **structural/reference knowledge** — what pages exist, what an
API endpoint returns, what columns a data table has. It's deliberately **not** live
numbers (prices, current stock, current jobs). Structural knowledge is stable
enough to embed directly; live numbers must always come from EVEFORGE, ESI, or
in-game, never from memory or from this repo.

## Layout

```
CONTEXT.md                    single self-contained file — everything below, one paste/fetch
knowledge/
  eveforge-site.md            EVEFORGE site map & terminology
  esi-api-index.md            ESI API endpoint index (what data is available)
  sde-reference.md            EVE SDE table reference (Fuzzwork export)
  methodologies.md            operator workflow/goal patterns (placeholder, WIP)
AGENTS.md                     universal agent primer (read first)
CLAUDE.md / GEMINI.md         thin pointers so each CLI auto-loads context
usage/how-to-query.md         exact steps for pointing each tool at this repo
```

`CONTEXT.md` and the `knowledge/` files carry the same content — `CONTEXT.md` is
the single-file, easy-to-paste-a-URL version; `knowledge/*.md` is the same material
split by topic for smaller-context tools or targeted editing. Keep them in sync
when either changes.

## Quick use

- **Any web chatbot:** paste the raw GitHub URL for `CONTEXT.md`, or paste its
  contents directly.
- **Claude CLI / Gemini CLI:** `cd` into a local clone and start the CLI — it
  auto-reads `CLAUDE.md`/`GEMINI.md` → `AGENTS.md` → the knowledge files.

See `usage/how-to-query.md` for exact steps per tool.

## Caveats

- The EVEFORGE site map (`knowledge/eveforge-site.md`) was built by rendering the
  live site with no character linked — every screen is documented in its empty/
  onboarding state. Table columns and navigation are accurate; populated-data
  behavior (once you link characters) hasn't been verified yet.
- The ESI index and SDE reference are snapshots (spec v1.36 / SDE build 3409592,
  2026-06-30). Both are fairly stable but can drift after CCP patches — re-fetch
  from source if precision matters.
- `methodologies.md` is a placeholder — only "stockpile production" is sketched in
  as a seed example, cross-referenced to EVEFORGE's actual Stockpiles/Smart Queue
  features. More methodologies to be added by the operator.
