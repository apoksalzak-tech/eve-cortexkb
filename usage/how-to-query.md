# How to query this knowledge base

## Any web chatbot (fastest path — this repo is public)

Paste the raw URL of `CONTEXT.md` directly into the chat, e.g.:

```
https://raw.githubusercontent.com/<your-username>/eve-cortexkb/main/CONTEXT.md
```

Most chatbots with URL-fetch/browsing can pull it directly; otherwise open that URL
yourself and paste the contents in as your first message. That one file is
self-contained — site map, ESI index, SDE reference, and methodologies all in one.

## Claude CLI / Gemini CLI

```
git clone https://github.com/<your-username>/eve-cortexkb
cd eve-cortexkb
claude    # or: gemini
```

It auto-reads `CLAUDE.md`/`GEMINI.md` → `AGENTS.md` → the `knowledge/` and
`tools/` files. Ask directly, e.g. "Where in EVEFORGE would I set up a stockpile
target for a T2 module?" or "What ESI scope do I need to read corp blueprints?"
Once other tools (Eve Guru, POD, ...) get their own `tools/<name>/site-map.md`,
the same pattern of questions works for them too.

## Keeping it current

`CONTEXT.md` and `knowledge/*.md` intentionally carry the same content in two
shapes (one file vs. topic files). When you edit one, update the other to match —
there's no build step, so it's a manual sync. The ESI index and SDE reference are
point-in-time snapshots; re-fetch from `esi.evetech.net/latest/swagger.json` and
`fuzzwork.co.uk/dump/latest/` if you suspect drift after a game patch.
