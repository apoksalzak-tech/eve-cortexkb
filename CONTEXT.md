---
title: eve-cortexkb — Combined Context
purpose: Single-file, self-contained AI context repo distilling the operator's EVE Online knowledge over time — the EVE ESI API, the EVE SDE, operator methodologies, and per-tool deep-dives (EVEFORGE currently, more planned) — not scoped to any one tool
---

# eve-cortexkb — Combined Context

Paste this whole file, or point a chatbot at its raw GitHub URL, and it has
everything below in one shot. If you're an AI reading this: this is the full
knowledge base in one file. The same content also exists split into topic files
under `knowledge/` and `tools/` in this repo, kept in sync with this file — use
whichever is more convenient for your context window.

## Who this is for

The operator plays EVE Online with 60+ characters and is building this repo as a
durable **AI context repo** — a place to distill years of accumulated EVE Online
knowledge so any AI assistant can load it and reason correctly instead of
guessing. It is not scoped to any single tool. Today that covers two foundational
data sources — the **EVE ESI API** and the **EVE SDE** (Static Data Export) — the
operator's own methodologies (Part 3), per-tool deep-dives (Part 4), and
standalone practical references like public no-auth market data lookups
(Part 5). **[eveforge.org](https://eveforge.org) (EVEFORGE)** is the tool
currently in use and fully documented; it is not treated as the operator's only
or permanent tool — other tools (Eve Guru, POD, ...) will get their own sections
in Part 4 as they're adopted, and this file's scope is expected to keep growing
well beyond what's here today. The problem this repo solves: a given tool's layout and
terminology aren't self-explanatory, game mechanics have sharp edges that are
easy to misremember, and understanding any of it well requires knowing what data
is available (ESI), what static game data underlies everything (the SDE), and how
the operator actually works. This file gives an assistant all of that at once.

## The one rule

Everything in this file is **structural/reference knowledge** (what pages exist,
what an API endpoint returns, what columns a data table has) — deliberately **not**
live numbers (prices, current stock, current market state). Structural knowledge
is stable enough to embed directly. Live numbers age out within hours and must
always be pulled fresh from the relevant tool, ESI, or in-game — never asserted
from memory or from this file.

---

# Part 1 — ESI API Index

*(source: https://esi.evetech.net/latest/swagger.json, spec version 1.36, fetched 2026-06-30)*

ESI (EVE Swagger Interface) is CCP's official REST API for EVE Online. It's the
data backbone under EVEFORGE (and every other third-party industry/market tool):
character/corp data, assets, industry jobs, market orders, and static universe data
all come from here. This is a structural index of **what data exists and how to get
it** — not live data itself.

- Base URL: `https://esi.evetech.net/latest`
- Full machine-readable spec: `https://esi.evetech.net/latest/swagger.json`
  (re-fetch if precision matters — endpoints are fairly stable but this snapshot
  will age)
- Interactive docs: `https://esi.evetech.net/ui/`
- Auth: EVE SSO (OAuth2). Public endpoints need no token; scoped endpoints need a
  token carrying the specific scope named below.
- 180 endpoints total across 29 tags, as of spec 1.36.

## What this KB does NOT define: how authentication actually happens

This index tells you which scope a call needs — it does not, and cannot, tell you
how your specific environment obtains or attaches the OAuth2 bearer token. That's
environment-specific; don't assume a mechanism:

- **Never attempt to run the OAuth2 flow yourself** — don't construct SSO login
  URLs, don't ask the operator to paste a client secret or access token into
  chat.
- **If your environment provides a tool/function/MCP server for ESI calls**,
  assume it handles token lookup and injection for you based on the
  `character_id` you specify — you supply the character_id and the endpoint; the
  tool supplies the Bearer token behind the scenes.
- **If you have no such tool** — e.g. you're a plain chatbot with this file
  pasted in and no live API access — you cannot make ESI calls at all, public or
  scoped. Say so explicitly rather than fabricating a plausible-looking response.
  In that case you're reasoning from this file's structural knowledge only, and
  any actual data question should be redirected to EVEFORGE or in-game.

`` `GET /characters/{character_id}/assets/` — Get character assets [scope: esi-assets.read_assets.v1] ``
means: HTTP GET, path takes a character_id, caller's token must carry that scope.
`[public]` = no token required.

## Name → typeID resolution (do this before anything else)

Every EVE item, ship, or module has a canonical name (e.g. "Paladin") and a
canonical integer ID (`typeID`) that all structural data — every ESI response,
every SDE table — is actually keyed on. **When a user asks about an item by name,
resolve the name to a typeID FIRST, before attempting any lookup, join, or
calculation.** Two ways to do it, in order of preference:

1. **ESI**: `POST /universe/ids/` [public] — send a list of names, get back
   typeIDs (and IDs for other categories: characters, corporations, systems,
   etc., all in one call). Fastest option if you have live API access.
2. **SDE**: search `invTypes.csv` (or the `invTypes` table in a full DB import,
   see Part 2) for a `typeName` match — exact match is safest, EVE item names are
   unique. Use this when you only have the SDE loaded and no live ESI access.

Do not guess a typeID from memory or pattern-match a name to a "likely" ID —
type IDs aren't predictable from the name, and a wrong ID silently returns data
for the wrong item with no error. Lock in the correct typeID before doing
anything downstream with it (blueprint materials via
`industryActivityMaterials`, module attributes via `dgmTypeAttributes`, market
data via `/markets/{region_id}/orders/`, etc.).

### Alliance
- `GET /alliances/` — List all alliances [public]
- `GET /alliances/{alliance_id}/` — Get alliance information [public]
- `GET /alliances/{alliance_id}/corporations/` — List alliance's corporations [public]
- `GET /alliances/{alliance_id}/icons/` — Get alliance icon [public]

### Assets
- `GET /characters/{character_id}/assets/` — Get character assets [scope: esi-assets.read_assets.v1]
- `POST /characters/{character_id}/assets/locations/` — Get character asset locations [scope: esi-assets.read_assets.v1]
- `POST /characters/{character_id}/assets/names/` — Get character asset names [scope: esi-assets.read_assets.v1]
- `GET /corporations/{corporation_id}/assets/` — Get corporation assets [scope: esi-assets.read_corporation_assets.v1]
- `POST /corporations/{corporation_id}/assets/locations/` — Get corporation asset locations [scope: esi-assets.read_corporation_assets.v1]
- `POST /corporations/{corporation_id}/assets/names/` — Get corporation asset names [scope: esi-assets.read_corporation_assets.v1]

### Calendar
- `GET /characters/{character_id}/calendar/` — List calendar event summaries [scope: esi-calendar.read_calendar_events.v1]
- `GET /characters/{character_id}/calendar/{event_id}/` — Get an event [scope: esi-calendar.read_calendar_events.v1]
- `PUT /characters/{character_id}/calendar/{event_id}/` — Respond to an event [scope: esi-calendar.respond_calendar_events.v1]
- `GET /characters/{character_id}/calendar/{event_id}/attendees/` — Get attendees [scope: esi-calendar.read_calendar_events.v1]

### Character
- `POST /characters/affiliation/` — Character affiliation [public]
- `GET /characters/{character_id}/` — Get character's public information [public]
- `GET /characters/{character_id}/agents_research/` — Get agents research [scope: esi-characters.read_agents_research.v1]
- `GET /characters/{character_id}/blueprints/` — Get blueprints [scope: esi-characters.read_blueprints.v1]
- `GET /characters/{character_id}/corporationhistory/` — Get corporation history [public]
- `POST /characters/{character_id}/cspa/` — Calculate a CSPA charge cost [scope: esi-characters.read_contacts.v1]
- `GET /characters/{character_id}/fatigue/` — Get jump fatigue [scope: esi-characters.read_fatigue.v1]
- `GET /characters/{character_id}/medals/` — Get medals [scope: esi-characters.read_medals.v1]
- `GET /characters/{character_id}/notifications/` — Get character notifications [scope: esi-characters.read_notifications.v1]
- `GET /characters/{character_id}/notifications/contacts/` — Get new contact notifications [scope: esi-characters.read_notifications.v1]
- `GET /characters/{character_id}/portrait/` — Get character portraits [public]
- `GET /characters/{character_id}/roles/` — Get character corporation roles [scope: esi-characters.read_corporation_roles.v1]
- `GET /characters/{character_id}/standings/` — Get standings [scope: esi-characters.read_standings.v1]
- `GET /characters/{character_id}/titles/` — Get character corporation titles [scope: esi-characters.read_titles.v1]

### Clones
- `GET /characters/{character_id}/clones/` — Get clones [scope: esi-clones.read_clones.v1]
- `GET /characters/{character_id}/implants/` — Get active implants [scope: esi-clones.read_implants.v1]

### Contacts
- `GET /alliances/{alliance_id}/contacts/` — Get alliance contacts [scope: esi-alliances.read_contacts.v1]
- `GET /alliances/{alliance_id}/contacts/labels/` — Get alliance contact labels [scope: esi-alliances.read_contacts.v1]
- `DELETE /characters/{character_id}/contacts/` — Delete contacts [scope: esi-characters.write_contacts.v1]
- `GET /characters/{character_id}/contacts/` — Get contacts [scope: esi-characters.read_contacts.v1]
- `POST /characters/{character_id}/contacts/` — Add contacts [scope: esi-characters.write_contacts.v1]
- `PUT /characters/{character_id}/contacts/` — Edit contacts [scope: esi-characters.write_contacts.v1]
- `GET /characters/{character_id}/contacts/labels/` — Get contact labels [scope: esi-characters.read_contacts.v1]
- `GET /corporations/{corporation_id}/contacts/` — Get corporation contacts [scope: esi-corporations.read_contacts.v1]
- `GET /corporations/{corporation_id}/contacts/labels/` — Get corporation contact labels [scope: esi-corporations.read_contacts.v1]

### Contracts
- `GET /characters/{character_id}/contracts/` — Get contracts [scope: esi-contracts.read_character_contracts.v1]
- `GET /characters/{character_id}/contracts/{contract_id}/bids/` — Get contract bids [scope: esi-contracts.read_character_contracts.v1]
- `GET /characters/{character_id}/contracts/{contract_id}/items/` — Get contract items [scope: esi-contracts.read_character_contracts.v1]
- `GET /contracts/public/bids/{contract_id}/` — Get public contract bids [public]
- `GET /contracts/public/items/{contract_id}/` — Get public contract items [public]
- `GET /contracts/public/{region_id}/` — Get public contracts [public]
- `GET /corporations/{corporation_id}/contracts/` — Get corporation contracts [scope: esi-contracts.read_corporation_contracts.v1]
- `GET /corporations/{corporation_id}/contracts/{contract_id}/bids/` — Get corporation contract bids [scope: esi-contracts.read_corporation_contracts.v1]
- `GET /corporations/{corporation_id}/contracts/{contract_id}/items/` — Get corporation contract items [scope: esi-contracts.read_corporation_contracts.v1]

### Corporation
- `GET /corporations/npccorps/` — Get npc corporations [public]
- `GET /corporations/{corporation_id}/` — Get corporation information [public]
- `GET /corporations/{corporation_id}/alliancehistory/` — Get alliance history [public]
- `GET /corporations/{corporation_id}/blueprints/` — Get corporation blueprints [scope: esi-corporations.read_blueprints.v1]
- `GET /corporations/{corporation_id}/containers/logs/` — Get all corporation ALSC logs [scope: esi-corporations.read_container_logs.v1]
- `GET /corporations/{corporation_id}/divisions/` — Get corporation divisions [scope: esi-corporations.read_divisions.v1]
- `GET /corporations/{corporation_id}/facilities/` — Get corporation facilities [scope: esi-corporations.read_facilities.v1]
- `GET /corporations/{corporation_id}/icons/` — Get corporation icon [public]
- `GET /corporations/{corporation_id}/medals/` — Get corporation medals [scope: esi-corporations.read_medals.v1]
- `GET /corporations/{corporation_id}/medals/issued/` — Get corporation issued medals [scope: esi-corporations.read_medals.v1]
- `GET /corporations/{corporation_id}/members/` — Get corporation members [scope: esi-corporations.read_corporation_membership.v1]
- `GET /corporations/{corporation_id}/members/limit/` — Get corporation member limit [scope: esi-corporations.track_members.v1]
- `GET /corporations/{corporation_id}/members/titles/` — Get corporation's members' titles [scope: esi-corporations.read_titles.v1]
- `GET /corporations/{corporation_id}/membertracking/` — Track corporation members [scope: esi-corporations.track_members.v1]
- `GET /corporations/{corporation_id}/roles/` — Get corporation member roles [scope: esi-corporations.read_corporation_membership.v1]
- `GET /corporations/{corporation_id}/roles/history/` — Get corporation member roles history [scope: esi-corporations.read_corporation_membership.v1]
- `GET /corporations/{corporation_id}/shareholders/` — Get corporation shareholders [scope: esi-wallet.read_corporation_wallets.v1]
- `GET /corporations/{corporation_id}/standings/` — Get corporation standings [scope: esi-corporations.read_standings.v1]
- `GET /corporations/{corporation_id}/starbases/` — Get corporation starbases (POSes) [scope: esi-corporations.read_starbases.v1]
- `GET /corporations/{corporation_id}/starbases/{starbase_id}/` — Get starbase (POS) detail [scope: esi-corporations.read_starbases.v1]
- `GET /corporations/{corporation_id}/structures/` — Get corporation structures [scope: esi-corporations.read_structures.v1]
- `GET /corporations/{corporation_id}/titles/` — Get corporation titles [scope: esi-corporations.read_titles.v1]

### Dogma
- `GET /dogma/attributes/` — Get attributes [public]
- `GET /dogma/attributes/{attribute_id}/` — Get attribute information [public]
- `GET /dogma/dynamic/items/{type_id}/{item_id}/` — Get dynamic item information [public]
- `GET /dogma/effects/` — Get effects [public]
- `GET /dogma/effects/{effect_id}/` — Get effect information [public]

### Faction Warfare
- `GET /characters/{character_id}/fw/stats/` — Overview of a character involved in faction warfare [scope: esi-characters.read_fw_stats.v1]
- `GET /corporations/{corporation_id}/fw/stats/` — Overview of a corporation involved in faction warfare [scope: esi-corporations.read_fw_stats.v1]
- `GET /fw/leaderboards/` — List of the top factions in faction warfare [public]
- `GET /fw/leaderboards/characters/` — List of the top pilots in faction warfare [public]
- `GET /fw/leaderboards/corporations/` — List of the top corporations in faction warfare [public]
- `GET /fw/stats/` — An overview of statistics about factions involved in faction warfare [public]
- `GET /fw/systems/` — Ownership of faction warfare systems [public]
- `GET /fw/wars/` — Data about which NPC factions are at war [public]

### Fittings
- `GET /characters/{character_id}/fittings/` — Get fittings [scope: esi-fittings.read_fittings.v1]
- `POST /characters/{character_id}/fittings/` — Create fitting [scope: esi-fittings.write_fittings.v1]
- `DELETE /characters/{character_id}/fittings/{fitting_id}/` — Delete fitting [scope: esi-fittings.write_fittings.v1]

### Fleets
- `GET /characters/{character_id}/fleet/` — Get character fleet info [scope: esi-fleets.read_fleet.v1]
- `GET /fleets/{fleet_id}/` — Get fleet information [scope: esi-fleets.read_fleet.v1]
- `PUT /fleets/{fleet_id}/` — Update fleet [scope: esi-fleets.write_fleet.v1]
- `GET /fleets/{fleet_id}/members/` — Get fleet members [scope: esi-fleets.read_fleet.v1]
- `POST /fleets/{fleet_id}/members/` — Create fleet invitation [scope: esi-fleets.write_fleet.v1]
- `DELETE /fleets/{fleet_id}/members/{member_id}/` — Kick fleet member [scope: esi-fleets.write_fleet.v1]
- `PUT /fleets/{fleet_id}/members/{member_id}/` — Move fleet member [scope: esi-fleets.write_fleet.v1]
- `DELETE /fleets/{fleet_id}/squads/{squad_id}/` — Delete fleet squad [scope: esi-fleets.write_fleet.v1]
- `PUT /fleets/{fleet_id}/squads/{squad_id}/` — Rename fleet squad [scope: esi-fleets.write_fleet.v1]
- `GET /fleets/{fleet_id}/wings/` — Get fleet wings [scope: esi-fleets.read_fleet.v1]
- `POST /fleets/{fleet_id}/wings/` — Create fleet wing [scope: esi-fleets.write_fleet.v1]
- `DELETE /fleets/{fleet_id}/wings/{wing_id}/` — Delete fleet wing [scope: esi-fleets.write_fleet.v1]
- `PUT /fleets/{fleet_id}/wings/{wing_id}/` — Rename fleet wing [scope: esi-fleets.write_fleet.v1]
- `POST /fleets/{fleet_id}/wings/{wing_id}/squads/` — Create fleet squad [scope: esi-fleets.write_fleet.v1]

### Incursions
- `GET /incursions/` — List incursions [public]

### Industry
- `GET /characters/{character_id}/industry/jobs/` — List character industry jobs [scope: esi-industry.read_character_jobs.v1]
- `GET /characters/{character_id}/mining/` — Character mining ledger [scope: esi-industry.read_character_mining.v1]
- `GET /corporation/{corporation_id}/mining/extractions/` — Moon extraction timers [scope: esi-industry.read_corporation_mining.v1]
- `GET /corporation/{corporation_id}/mining/observers/` — Corporation mining observers [scope: esi-industry.read_corporation_mining.v1]
- `GET /corporation/{corporation_id}/mining/observers/{observer_id}/` — Observed corporation mining [scope: esi-industry.read_corporation_mining.v1]
- `GET /corporations/{corporation_id}/industry/jobs/` — List corporation industry jobs [scope: esi-industry.read_corporation_jobs.v1]
- `GET /industry/facilities/` — List industry facilities [public]
- `GET /industry/systems/` — List solar system cost indices [public]

### Insurance
- `GET /insurance/prices/` — List insurance levels [public]

### Killmails
- `GET /characters/{character_id}/killmails/recent/` — Get a character's recent kills and losses [scope: esi-killmails.read_killmails.v1]
- `GET /corporations/{corporation_id}/killmails/recent/` — Get a corporation's recent kills and losses [scope: esi-killmails.read_corporation_killmails.v1]
- `GET /killmails/{killmail_id}/{killmail_hash}/` — Get a single killmail [public]

### Location
- `GET /characters/{character_id}/location/` — Get character location [scope: esi-location.read_location.v1]
- `GET /characters/{character_id}/online/` — Get character online [scope: esi-location.read_online.v1]
- `GET /characters/{character_id}/ship/` — Get current ship [scope: esi-location.read_ship_type.v1]

### Loyalty
- `GET /characters/{character_id}/loyalty/points/` — Get loyalty points [scope: esi-characters.read_loyalty.v1]
- `GET /loyalty/stores/{corporation_id}/offers/` — List loyalty store offers [public]

### Mail
- `GET /characters/{character_id}/mail/` — Return mail headers [scope: esi-mail.read_mail.v1]
- `POST /characters/{character_id}/mail/` — Send a new mail [scope: esi-mail.send_mail.v1]
- `GET /characters/{character_id}/mail/labels/` — Get mail labels and unread counts [scope: esi-mail.read_mail.v1]
- `POST /characters/{character_id}/mail/labels/` — Create a mail label [scope: esi-mail.organize_mail.v1]
- `DELETE /characters/{character_id}/mail/labels/{label_id}/` — Delete a mail label [scope: esi-mail.organize_mail.v1]
- `GET /characters/{character_id}/mail/lists/` — Return mailing list subscriptions [scope: esi-mail.read_mail.v1]
- `DELETE /characters/{character_id}/mail/{mail_id}/` — Delete a mail [scope: esi-mail.organize_mail.v1]
- `GET /characters/{character_id}/mail/{mail_id}/` — Return a mail [scope: esi-mail.read_mail.v1]
- `PUT /characters/{character_id}/mail/{mail_id}/` — Update metadata about a mail [scope: esi-mail.organize_mail.v1]

### Market
- `GET /characters/{character_id}/orders/` — List open orders from a character [scope: esi-markets.read_character_orders.v1]
- `GET /characters/{character_id}/orders/history/` — List historical orders by a character [scope: esi-markets.read_character_orders.v1]
- `GET /corporations/{corporation_id}/orders/` — List open orders from a corporation [scope: esi-markets.read_corporation_orders.v1]
- `GET /corporations/{corporation_id}/orders/history/` — List historical orders from a corporation [scope: esi-markets.read_corporation_orders.v1]
- `GET /markets/groups/` — Get item groups [public]
- `GET /markets/groups/{market_group_id}/` — Get item group information [public]
- `GET /markets/prices/` — List market prices [public]
- `GET /markets/structures/{structure_id}/` — List orders in a structure [scope: esi-markets.structure_markets.v1]
- `GET /markets/{region_id}/history/` — List historical market statistics in a region [public]
- `GET /markets/{region_id}/orders/` — List orders in a region [public]
- `GET /markets/{region_id}/types/` — List type IDs relevant to a market [public]

### Planetary Interaction
- `GET /characters/{character_id}/planets/` — Get colonies [scope: esi-planets.manage_planets.v1]
- `GET /characters/{character_id}/planets/{planet_id}/` — Get colony layout [scope: esi-planets.manage_planets.v1]
- `GET /corporations/{corporation_id}/customs_offices/` — List corporation customs offices [scope: esi-planets.read_customs_offices.v1]
- `GET /universe/schematics/{schematic_id}/` — Get schematic information [public]

### Routes
- `GET /route/{origin}/{destination}/` — Get route [public]

### Search
- `GET /characters/{character_id}/search/` — Search on a string [scope: esi-search.search_structures.v1]

### Skills
- `GET /characters/{character_id}/attributes/` — Get character attributes [scope: esi-skills.read_skills.v1]
- `GET /characters/{character_id}/skillqueue/` — Get character's skill queue [scope: esi-skills.read_skillqueue.v1]
- `GET /characters/{character_id}/skills/` — Get character skills [scope: esi-skills.read_skills.v1]

### Sovereignty
- `GET /sovereignty/campaigns/` — List sovereignty campaigns [public]
- `GET /sovereignty/map/` — List sovereignty of systems [public]
- `GET /sovereignty/structures/` — List sovereignty structures [public]

### Status
- `GET /status/` — Retrieve the uptime and player counts [public]

### Universe
- `GET /universe/ancestries/` — Get ancestries [public]
- `GET /universe/asteroid_belts/{asteroid_belt_id}/` — Get asteroid belt information [public]
- `GET /universe/bloodlines/` — Get bloodlines [public]
- `GET /universe/categories/` — Get item categories [public]
- `GET /universe/categories/{category_id}/` — Get item category information [public]
- `GET /universe/constellations/` — Get constellations [public]
- `GET /universe/constellations/{constellation_id}/` — Get constellation information [public]
- `GET /universe/factions/` — Get factions [public]
- `GET /universe/graphics/` — Get graphics [public]
- `GET /universe/graphics/{graphic_id}/` — Get graphic information [public]
- `GET /universe/groups/` — Get item groups [public]
- `GET /universe/groups/{group_id}/` — Get item group information [public]
- `POST /universe/ids/` — Bulk names to IDs [public]
- `GET /universe/moons/{moon_id}/` — Get moon information [public]
- `POST /universe/names/` — Get names and categories for a set of IDs [public]
- `GET /universe/planets/{planet_id}/` — Get planet information [public]
- `GET /universe/races/` — Get character races [public]
- `GET /universe/regions/` — Get regions [public]
- `GET /universe/regions/{region_id}/` — Get region information [public]
- `GET /universe/stargates/{stargate_id}/` — Get stargate information [public]
- `GET /universe/stars/{star_id}/` — Get star information [public]
- `GET /universe/stations/{station_id}/` — Get station information [public]
- `GET /universe/structures/` — List all public structures [public]
- `GET /universe/structures/{structure_id}/` — Get structure information [scope: esi-universe.read_structures.v1]
- `GET /universe/system_jumps/` — Get system jumps [public]
- `GET /universe/system_kills/` — Get system kills [public]
- `GET /universe/systems/` — Get solar systems [public]
- `GET /universe/systems/{system_id}/` — Get solar system information [public]
- `GET /universe/types/` — Get types [public]
- `GET /universe/types/{type_id}/` — Get type information [public]

### User Interface
- `POST /ui/autopilot/waypoint/` — Set Autopilot Waypoint [scope: esi-ui.write_waypoint.v1]
- `POST /ui/openwindow/contract/` — Open Contract Window [scope: esi-ui.open_window.v1]
- `POST /ui/openwindow/information/` — Open Information Window [scope: esi-ui.open_window.v1]
- `POST /ui/openwindow/marketdetails/` — Open Market Details [scope: esi-ui.open_window.v1]
- `POST /ui/openwindow/newmail/` — Open New Mail Window [scope: esi-ui.open_window.v1]

### Wallet
- `GET /characters/{character_id}/wallet/` — Get a character's wallet balance [scope: esi-wallet.read_character_wallet.v1]
- `GET /characters/{character_id}/wallet/journal/` — Get character wallet journal [scope: esi-wallet.read_character_wallet.v1]
- `GET /characters/{character_id}/wallet/transactions/` — Get wallet transactions [scope: esi-wallet.read_character_wallet.v1]
- `GET /corporations/{corporation_id}/wallets/` — Returns a corporation's wallet balance [scope: esi-wallet.read_corporation_wallets.v1]
- `GET /corporations/{corporation_id}/wallets/{division}/journal/` — Get corporation wallet journal [scope: esi-wallet.read_corporation_wallets.v1]
- `GET /corporations/{corporation_id}/wallets/{division}/transactions/` — Get corporation wallet transactions [scope: esi-wallet.read_corporation_wallets.v1]

### Wars
- `GET /wars/` — List wars [public]
- `GET /wars/{war_id}/` — Get war information [public]
- `GET /wars/{war_id}/killmails/` — List kills for a war [public]

## Notes for a 60-character operation

### The error budget — read this before looping over 60 characters

ESI enforces a rolling **error budget**, separate from normal request handling.
Every response carries `X-Esi-Error-Limit-Remain` (errors left in the current
window) and `X-Esi-Error-Limit-Reset` (seconds until it resets). If the remaining
count drops low — or you get repeated 4xx/5xx responses (an expired character
token returning 401 is the classic trigger when looping a large roster) — **stop
and back off** rather than retrying aggressively. Exhausting the budget gets your
application rate-limited (HTTP 420) for the rest of the window, and sustained
abuse risks a longer-term block. Concretely, when iterating many characters:
check `X-Esi-Error-Limit-Remain` on every call, treat a 401 from one character's
token as "skip this character and move on," not "retry immediately," and never
busy-loop through failures. Also respect the `Expires` header on cacheable
responses — polling faster than the cache validity window burns your error/rate
budget for no new data, which matters most exactly in a 60-character loop.

- Every scoped endpoint above is **per-character** (or per-corporation with a
  director token) — there's no bulk "give me all 60 characters" call. A tool
  covering a large roster makes one authenticated call per character per endpoint
  it needs (this is exactly what EVEFORGE's "Refresh All" button is doing behind
  the scenes, and why big rosters take longer to sync).
- `industry/systems` (system cost indices) and `industry/facilities` are **public**
  — no character auth needed to know the current cost index for any system.
- `markets/{region_id}/orders/`, `/history/`, and `/types/` are **public** per
  region — this is the live-price backbone EVEFORGE (and any market tool) polls;
  it's per-region, not per-hub, so a hub like Jita is fetched via its region
  (The Forge) — see the trade-hub → region ID table in Part 2.
- Corp-level assets/jobs/wallet/blueprints require a **director** role token, not
  just any corp member — matches what EVEFORGE's Corporation Hub says.

## Player-owned market hubs (structure IDs)

Unlike the five NPC trade hubs (see the region-ID table in Part 2), which are
public and keyed by region, player-owned Upwell structures (Keepstars,
Fortizars, etc.) are **private by default** and keyed by an individual
`structure_id`, not a region. Querying one requires an authenticated token from a
character with docking/market access to that structure, carrying
`esi-universe.read_structures.v1` (for `GET /universe/structures/{structure_id}/`)
and `esi-markets.structure_markets.v1` (for
`GET /markets/structures/{structure_id}/`). A bare unauthenticated call returns
`401 Unauthorized` regardless of whether the ID is even valid — that response
doesn't confirm or deny anything about the structure itself.

**Operator's home market**: Goonswarm Federation's staging Keepstar, **"1st Taj
Mahgoon"**, in system **C-J6MT** (Insmother region), structure_id
**`1049588174021`**. Originally sourced from Goonswarm's own internal tool
(goonmetrics.apps.goonswarm.org) via a market-import URL referencing that ID —
**since corroborated**: querying Goonmetrics' public price API (Part 5) with
this exact structure_id as `station_id` returns live, coherent buy/sell data,
which a wrong or made-up ID would not do. Treat the ID itself as confirmed. Raw
unauthenticated ESI calls to `/markets/structures/1049588174021/` and
`/universe/structures/1049588174021/` still return `401 Unauthorized` though —
the structure is **not** publicly queryable via ESI directly; Goonmetrics can
serve price data for it because it holds its own authenticated character token
and re-publishes the result. **Full access (assets, jobs, anything beyond
price_data) is still gated by docking rights and scopes on the querying
character — the operator has flagged that as an open problem to solve
separately.** See Part 5 for the price-only workaround that's already solved.


---

# Part 2 — EVE Online SDE Reference (Fuzzwork export)

*(source: https://www.fuzzwork.co.uk/dump/latest/, observed 2026-06-30, dump dated 2026-06-26)*

The **Static Data Export (SDE)** is CCP's dump of all the *static* game data —
every item type, blueprint, reaction, PI schematic, region/system, NPC corp,
station, module attribute, and more. It does not contain live prices, player
assets, or anything ESI-authenticated — it's the "rulebook," not the "current
state." [Fuzzwork](https://www.fuzzwork.co.uk/dump/) republishes CCP's SDE in
several developer-friendly formats and keeps it current with each game patch.

## Where to get it

Base: `https://www.fuzzwork.co.uk/dump/latest/`

Whole-database dumps (pick one format):
- `sdeyaml_mysql_<build>_<date>.sql.gz` — MySQL dump, ~81M compressed
- `sdeyaml_postgres_<build>_<date>.pgdump` — PostgreSQL dump, ~81M
- `sdeyaml_postgresschema_<build>_<date>.pgdump` — Postgres, schema only, ~81M
- `sdeyaml_mssql_<build>_<date>.bak` — SQL Server backup, ~193M
- `eve_<build>_<date>.db.gz` — SQLite, ~138M compressed

Per-table exports: `https://www.fuzzwork.co.uk/dump/latest/csv/<tableName>.csv` —
useful when you only need one or two tables rather than the whole DB. This is the
easier path for a chatbot/agent to reason about (fetch just the table you need).

As of this write-up the published dump is build **3409592**, dated **2026-06-26**.
Fuzzwork refreshes this after every game patch — **check the directory listing for
the current date/build before assuming freshness.**

## Table index (industry/market-relevant, with real columns)

### Item catalog
| Table | Columns |
|---|---|
| `invTypes` | typeID, groupID, typeName, description, mass, volume, capacity, portionSize, raceID, basePrice, published, marketGroupID, iconID, soundID, graphicID, factionID, metaLevel, techLevel, shipTreeGroupID |
| `invGroups` | groupID, categoryID, groupName, iconID, useBasePrice, anchored, anchorable, fittableNonSingleton, published |
| `invCategories` | categoryID, categoryName, iconID, published |
| `invMarketGroups` | marketGroupID, parentGroupID, marketGroupName, description, iconID, hasTypes |
| `invMetaTypes` | typeID, parentTypeID, metaGroupID |
| `invMetaGroups` | metaGroupID, metaGroupName, description, iconID |
| `invTraits` | traitID, typeID, skillID, bonus, bonusText, unitID |
| `invVolumes` | typeID, volume (packaged-volume overrides, e.g. ships) |
| `invContrabandTypes` | factionID, typeID, standingLoss, confiscateMinSec, fineByValue, attackMinSec |
| `invUniqueNames` | itemID, itemName, groupID (named celestial/universe objects) |
| `invNames` | itemID, itemName |
| `compressibleTypes` | typeID, compressedTypeID (ore/ice compression pairs) |

`invTypes` is the root table — almost everything else joins to it by `typeID`.
`marketGroupID` is what powers "browse by market category" UIs; `groupID`/
`categoryID` is CCP's internal item taxonomy (ships, modules, etc.) and is what
EVEFORGE's Item Explorer category counts are built from.

### Manufacturing & industry
| Table | Columns |
|---|---|
| `industryBlueprints` | typeID, maxProductionLimit |
| `industryActivity` | typeID, activityID, time |
| `industryActivityMaterials` | typeID, activityID, materialTypeID, quantity |
| `industryActivityProducts` | typeID, activityID, productTypeID, quantity |
| `industryActivityProbabilities` | typeID, activityID, productTypeID, probability (invention odds) |
| `industryActivitySkills` | typeID, activityID, skillID, level |
| `industryActivityRaces` | typeID, activityID, productTypeID, raceID |
| `invTypeMaterials` | typeID, materialTypeID, quantity (reprocessing yield) |
| `invTypeReactions` | reactionTypeID, input, typeID, quantity |
| `ramActivities` | activityID, activityName, iconNo, description, published |
| `ramAssemblyLineTypes` | assemblyLineTypeID, assemblyLineTypeName, description, baseTimeMultiplier, baseMaterialMultiplier, baseCostMultiplier, volume, activityID, minCostPerHour |
| `ramAssemblyLineTypeDetailPerCategory` | assemblyLineTypeID, categoryID, timeMultiplier, materialMultiplier, costMultiplier |
| `ramAssemblyLineTypeDetailPerGroup` | assemblyLineTypeID, groupID, timeMultiplier, materialMultiplier, costMultiplier |
| `ramInstallationTypeContents` | installationTypeID, assemblyLineTypeID, quantity |

`activityID` conventions (stable across CCP's industry system): 1 = Manufacturing,
3 = Researching Time Efficiency (TE), 4 = Researching Material Efficiency (ME),
5 = Copying, 8 = Invention, 9 = Reverse Engineering, 11 = Reactions. **Verify
against `ramActivities` directly rather than trusting this list blindly** — it's
included here as a memory aid, not guaranteed current.

#### Industrial math: from SDE base values to actual build requirements

`industryActivityMaterials.quantity` is the **base, unmodified** material
requirement per run at zero bonuses — it is never the actual number of units
you'll consume. Treating it as a final answer is the single most common mistake
an agent will make with this data. The real requirement runs through a reduction
pipeline, in this order:

1. **Base quantity** — `industryActivityMaterials.quantity` (SDE), × number of runs.
2. **Blueprint Material Efficiency (ME)** — the ME level of the *specific
   blueprint copy being used*. This is character-owned state, not static data —
   get it from ESI `GET /characters/{character_id}/blueprints/` (field
   `material_efficiency`), never from the SDE.
3. **Structure type bonus** — the industry structure (e.g. Raitaru/Azbel/Sotiyo)
   has its own base ME bonus for certain item categories.
4. **Rig bonus** — T1/T2 industry rigs fitted to that structure add a further ME
   bonus, and the size of that bonus depends on the structure's security-space
   (highsec vs. lowsec/null/WH rigs give different bonus tiers).
5. Rounding is applied **once, at the end** of the whole chain (per material
   line, with a floor of 1 unit) — not re-rounded after each individual step.

**Character industry skills do not reduce material quantity** in current EVE
mechanics — a common misconception worth correcting explicitly, since an agent
asked to "account for skills" might otherwise fold a skill bonus into the
material math. Skills like Industry and Advanced Industry reduce **job
installation cost (ISK)** and **job time**, not material consumption. Job *time*
has its own, separate reduction chain: base `industryActivity.time` → blueprint
Time Efficiency (TE, also from the ESI blueprint record) → structure/rig TE
bonus → character time-reducing skills.

**Job cost (ISK) is a separate calculation from material cost entirely** —
roughly EIV (Estimated Item Value) × system cost index (ESI
`GET /industry/systems/`, public) × structure/facility tax settings. See Part 4's
Settings → Facilities entry for the SCC Industry Tax / Facility Tax split
EVEFORGE exposes there.

**Bottom line for an agent**: never present a "build cost" derived only from
`industryActivityMaterials` × current market price. At minimum, pull the actual
blueprint ME/TE from ESI before doing the arithmetic, and flag structure/rig
bonuses and system cost index as verify-in-EVEFORGE inputs if you don't have
them to hand. The *exact* percentages and rounding formula are patch-sensitive —
verify against current CCP documentation or EVEFORGE's own calculators (BOM
Calculator, Production Hub) rather than trusting a hardcoded formula.

### Planetary Industry
| Table | Columns |
|---|---|
| `planetSchematics` | schematicID, schematicName, cycleTime |
| `planetSchematicsTypeMap` | schematicID, typeID, quantity, isInput |
| `planetSchematicsPinMap` | schematicID, pinTypeID |
| `planetResources` | celestialID, power, workforce, reagentTypeID, reagentAmountPerCycle, reagentCyclePeriod, reagentSecuredCap, reagentUnsecuredCap |

### Universe / map
| Table | Columns |
|---|---|
| `mapRegions` | regionID, regionName, x, y, z, xMin/Max, yMin/Max, zMin/Max, factionID, nebula, radius |
| `mapConstellations` | regionID, constellationID, constellationName, coordinates..., factionID, radius |
| `mapSolarSystems` | regionID, constellationID, solarSystemID, solarSystemName, coordinates..., luminosity, border, fringe, corridor, hub, international, regional, constellation, **security**, factionID, radius, sunTypeID, securityClass |
| `mapSolarSystemJumps` | fromRegionID, fromConstellationID, fromSolarSystemID, toSolarSystemID, toConstellationID, toRegionID |
| `mapRegionJumps`, `mapConstellationJumps` | region/constellation-level jump graph |
| `mapDenormalize` | large flattened table of every celestial object (~80MB) |
| `staStations` | stationID, security, dockingCostPerVolume, maxShipVolumeDockable, officeRentalCost, operationID, stationTypeID, corporationID, solarSystemID, constellationID, regionID, stationName, coordinates, **reprocessingEfficiency**, reprocessingStationsTake, reprocessingHangarFlag |
| `staStationTypes` | stationTypeID, dock geometry, operationID, officeSlots, reprocessingEfficiency, conquerable |
| `staServices` | serviceID, serviceName, description |

#### Major trade hub → region ID (verified against live ESI, 2026-06-30)

| Hub | Region | Region ID |
|---|---|---|
| Jita | The Forge | 10000002 |
| Amarr | Domain | 10000043 |
| Dodixie | Sinq Laison | 10000032 |
| Rens | Heimatar | 10000030 |
| Hek | Metropolis | 10000042 |

Region IDs are effectively permanent (CCP doesn't reassign them). "Check Jita
prices" means "call `/markets/10000002/orders/` or `/history/`" — resolve the
hub name to its region ID from this table before hitting any market endpoint;
don't guess or re-derive it at query time.

These five are all **NPC stations**, public and permanent. For a **player-owned
null-sec market hub** (e.g. a home Keepstar), see "Player-owned market hubs
(structure IDs)" in Part 1 — different lookup mechanism entirely (structure_id +
authenticated token, not region_id + public call).

### Module/ship attributes (Dogma)
| Table | Columns |
|---|---|
| `dgmAttributeTypes` | attributeID, attributeName, description, iconID, defaultValue, published, displayName, unitID, stackable, highIsGood, categoryID |
| `dgmAttributeCategories` | categoryID, categoryName, categoryDescription |
| `dgmTypeAttributes` | typeID, attributeID, valueInt, valueFloat — **this is the big one**: every module/ship's every stat, keyed by attributeID (~16MB) |
| `dgmEffects` | effectID, effectName, effectCategory, expressions, description, and flags (isOffensive, isAssistance, isWarpSafe, etc.) |
| `dgmTypeEffects` | typeID, effectID, isDefault |

This is how ME/TE bonuses, rig bonuses, and reprocessing bonuses are actually
represented in the SDE: as `dgmTypeAttributes` rows against a structure/rig/skill's
typeID, decoded via `dgmAttributeTypes`. Skill-bonus percentages live here too —
this is the correct place to verify one, not memory.

### Skills / certificates
| Table | Columns |
|---|---|
| `certCerts` | certID, description, groupID, name |
| `certSkills` | certID, skillID, certLevelInt, skillLevel, certLevelText |
| `certMasteries` | typeID, masteryLevel, certID |

### NPC corporations / LP stores
| Table | Columns |
|---|---|
| `crpNPCCorporations` | corporationID, size, extent, solarSystemID, investor fields, publicShares, initialPrice, minSecurity, factionID, description, corporationName, ... |
| `crpNPCCorporationTrades` | corporationID, typeID (items an NPC corp deals in) |
| `crpNPCCorporationDivisions`, `crpNPCCorporationResearchFields`, `crpNPCDivisions` | corp structure/research metadata |

LP store *offer* contents (item for LP+ISK) live in the SDE's mission/LP-adjacent
tables at a structural level; EVEFORGE's LP Calculator likely combines this with
live ESI/market prices to compute ISK/LP — the SDE alone gives you the recipe, not
the current payout.

### Starbases / control towers
| Table | Columns |
|---|---|
| `invControlTowerResources` | controlTowerTypeID, resourceTypeID, purpose, quantity, minSecurityLevel, factionID (POS/refinery fuel requirements) |
| `invControlTowerResourcePurposes` | purpose, purposeText |

### Character creation / lore (lower priority for industry use)
`chrFactions`, `chrRaces`, `chrAncestries`, `chrBloodlines`, `chrAttributes`,
`chrTitles`, `chrCloneGrades`, `chrCloneGradeSkills` — character-creation and
faction metadata. Relevant mainly for standings/faction-based mechanics (LP
corp faction, contraband by faction) rather than day-to-day production.

### Not itemized above but present in the export
Missions/agents (`agtAgents*`, `mstMissions*`, `epicArcs*`), skins
(`skin*`), dungeons (`dungeon*`), sovereignty (`sovereigntyUpgrades`), ship
tech-tree/insurance/graphics/translation tables. Full current list: fetch
`https://www.fuzzwork.co.uk/dump/latest/csv/` directly — it's a plain Apache
directory listing, cheap to re-check.

## Data conflicts: when the SDE isn't the final answer

Two known cases where static SDE data is either the wrong table to use, or gets
overridden entirely by live/dynamic data:

- **Ship volume for logistics/hauling.** `invTypes.volume` is a ship's
  **unpackaged/assembled** volume (huge — the in-space, fitted size). For cargo,
  freight, or hauling math, you almost always want the **packaged** volume
  instead, which lives in `invVolumes.typeID/volume` as an override table — only
  present for types that can be repackaged (mainly ships). If a typeID has a row
  in `invVolumes`, use that value for logistics; otherwise fall back to
  `invTypes.volume`.
- **Abyssal / Mutated modules.** These have no fixed stats in the SDE at all —
  their attributes are rolled per individual item instance by a mutaplasmid.
  `dgmTypeAttributes` only has the *unmutated* base module's stats. For an
  actual mutated item, ESI's `GET /dogma/dynamic/items/{type_id}/{item_id}/`
  (Part 1 → Dogma) returns the item's real, rolled attributes and **completely
  overrides** anything `dgmTypeAttributes` would say for that specific item
  instance. If a user mentions "abyssal," "mutated," or a mutaplasmid name,
  assume static SDE attribute data does not apply — the dynamic-items endpoint
  (or an ESI-synced tool like EVEFORGE) is required.

## Practical notes

- **typeID is the universal join key.** Almost every question ("what does this
  blueprint need," "what does this module do," "what system is this station in")
  resolves to joining through `invTypes.typeID`.
- **The SDE has no prices.** `invTypes.basePrice` exists but is a vestigial
  NPC-seed value, not a market price — never treat it as current. Live prices only
  come from ESI market endpoints (Part 1) or a market tool like EVEFORGE.
- **Security status** for reprocessing/industry bonuses comes from
  `mapSolarSystems.security` (system) and `staStations.reprocessingEfficiency` /
  `staStationTypes.reprocessingEfficiency` (station type) — matches EVEFORGE's
  Resource Calculator "Operating Zone: Hi-Sec / Low-Sec / Null-WH" selector.


---

# Part 3 — Industrialist Methodologies

*(status: placeholder — to be filled in by the operator)*

This section is intentionally mostly empty. It's where different ways EVE
industrialists manage their operations get documented — not mechanics (Parts 1–3),
but **workflow and goal patterns**: how someone actually runs their empire day to
day, and why they chose that pattern over another.

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
see Part 4 → Inventory → **Stockpiles** ("named inventory target containers for
tracking stock health") and → Smart Queue ("optimal slot allocation by ISK/hr...
runs based on stock deficit"). An operator running this methodology should be
pointed straight at those two screens rather than a manual BOM/queue workflow.

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


---

# Part 4 — Tool Deep Dives

Per-tool page maps and terminology, built from direct observation of each tool's
live UI. None of these is treated as the operator's single primary tool — check
whichever section matches the tool actually in question. Currently one tool is
fully documented; more will be added here as the operator adopts and observes
them.

## EVEFORGE

*(source: https://eveforge.org, observed 2026-06-30)*

### What EVEFORGE is

A free, browser-based EVE Online **industry & market ERP**, synced to your
characters via ESI. Single-page app at `https://eveforge.org`. Version observed:
**v0.8.8 beta**. Author credit in-app: "ELFX" (linked to a zKillboard character
page). Copyright footer: "© 2026 Fenris Creations." "Help & Docs" in the footer
links to `github.com/sa1rex/eve-forge`, which returned 404 (private or renamed) at
time of writing — not a usable source.

### How this map was built (and its limits)

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

### Top navigation (5 groups)

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

### Page-by-page detail

#### Command Center

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
to target, build to cover the deficit" workflow — see Part 3 (Methodologies).

#### Plan

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

#### Discover

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

#### Tools

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

#### Account

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

#### Other

**Commodity** (`/commodity`) — "Unified item detail — prices, history, margin, BOM,
and trend in one view." A single-item deep-dive page; reached by searching any item
name. Not in the left nav directly (likely linked from search results / item rows
elsewhere).

### Terminology quick-reference

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


## Eve Guru

*(placeholder — not yet documented; add a page-by-page map here once observed)*

## POD

*(placeholder — not yet documented; add a page-by-page map here once observed)*
---

# Part 5 — Public Market Data Lookups (No Auth Required)

*(status: verified working, 2026-07-01 — all example calls below were actually run)*

Purpose: get the **current price of one or a few specific items** — for an agent
answering a quick question, or for pulling a number into a Google Sheet — without
standing up a database and without EVE SSO/OAuth2. This is deliberately scoped
small: a handful of items, not a market-wide sync (EVEFORGE, or the raw ESI
`markets/{region_id}/orders/` endpoint in Part 1, are the right tools for that).

Raw ESI (`GET /markets/{region_id}/orders/`) is public and would work, but it
returns the **entire order book** (potentially thousands of rows, paginated) — you'd
have to fetch it all and aggregate client-side just to answer "what's the price."
The three services below do that aggregation for you and hand back a single
current-price-shaped answer in one call.

## Quick comparison

| Source | Auth | Response format | Scope | Best for |
|---|---|---|---|---|
| **Fuzzwork** | none | JSON | region OR station | quick buy/sell aggregate, multiple types per call |
| **EVE Tycoon** | none | JSON (served as `text/plain`) | region only | richer stats (outliers, 5%-trimmed averages) |
| **Goonmetrics** | none | XML | station (incl. player structures Goonswarm has access to) | Jita **and** the operator's own C-J6MT home market; native Google Sheets `IMPORTXML` support |

All three were queried live on 2026-07-01 with Tritanium (`typeID 34`) against
The Forge (`region_id 10000002`) / Jita 4-4 (`station_id 60003760`) to confirm
they work as documented below — not taken on faith from old docs.

## Fuzzwork market aggregates

```
GET https://market.fuzzwork.co.uk/aggregates/?region={region_id}&types={type_id}[,{type_id}...]
GET https://market.fuzzwork.co.uk/aggregates/?station={station_id}&types={type_id}[,{type_id}...]
```

Region-level or station-level (use one or the other, not both); comma-separate
multiple `types`. Example (Jita 4-4, Tritanium):

```
curl "https://market.fuzzwork.co.uk/aggregates/?station=60003760&types=34"
```
```json
{"34":{"buy":{"weightedAverage":"2.22","max":"3.41","min":"0.01","stddev":"0.98","median":"3.005","volume":"12632295075.0","orderCount":"34","percentile":"3.324"},"sell":{"weightedAverage":"3.97","max":"55000.0","min":"3.59","stddev":"6436.15","median":"4.12","volume":"16700843562.0","orderCount":"73","percentile":"3.5995"}}}
```

Response is keyed by typeID, each with `buy`/`sell` objects. **`percentile` is the
closest thing to "current price"** — a 5th-percentile-trimmed buy/sell to filter
out troll orders; `min`/`max` are the raw extremes (don't use `max` on `sell` as
"the price," it's often a joke listing in the millions). Fuzzworks also has a
name→typeID helper used elsewhere in this repo:
`GET https://www.fuzzwork.co.uk/api/typeid.php?typename={name}`.

## EVE Tycoon market stats

```
GET https://evetycoon.com/api/v1/market/stats/{region_id}/{type_id}
```

One type per call, region only (a structure-ID path was tried and returned
`400` — this endpoint doesn't take player-structure IDs). Example (The Forge,
Tritanium):

```
curl "https://evetycoon.com/api/v1/market/stats/10000002/34"
```
```json
{"buyVolume":17874287962,"sellVolume":17629423351,"buyOrders":76,"sellOrders":191,"buyOutliers":1,"sellOutliers":11,"buyThreshold":0.387,"sellThreshold":36.0,"buyAvgFivePercent":3.38963431678987,"sellAvgFivePercent":3.5999999999999996,"maxBuy":3.87,"minSell":3.6}
```

`minSell` / `maxBuy` are the cleanest single-number answer to "what's the current
price" — already outlier-filtered (see `buyOutliers`/`sellOutliers` counts).
`buyAvgFivePercent`/`sellAvgFivePercent` mirror Fuzzwork's `percentile`. No
published OpenAPI/docs page was found (the `/docs` page is a client-rendered SPA
with no visible endpoint list) — this endpoint shape was confirmed by direct
testing, not documentation; treat it as slightly less stable than Fuzzwork's
(which has been a long-standing community-standard API).

## Goonmetrics price data (also covers the operator's C-J6MT home market)

```
GET https://goonmetrics.apps.goonswarm.org/api/price_data/?station_id={station_or_structure_id}&type_id={type_id}[,{type_id}...]
GET https://goonmetrics.apps.goonswarm.org/api/price_history/?region_id={region_id}&type_id={type_id}[,{type_id}...]
```

Up to 50 `type_id`s per call (comma-separated). `price_history` returns the last
30 days. Self-documented at
[goonmetrics.apps.goonswarm.org/api/](https://goonmetrics.apps.goonswarm.org/api/).
Example (Jita 4-4, Tritanium):

```
curl "https://goonmetrics.apps.goonswarm.org/api/price_data/?station_id=60003760&type_id=34"
```
```xml
<goonmetrics method="price_data" version="1.0">
  <price_data>
    <type id="34">
      <updated>2026-07-01T04:37:57Z</updated>
      <all><weekly_movement>39570789234.4</weekly_movement></all>
      <buy><max>3.41</max><listed>13632295075</listed></buy>
      <sell><min>3.60</min><listed>16478386271</listed></sell>
    </type>
  </price_data>
</goonmetrics>
```

**This same endpoint also works for the operator's home market** — Goonswarm's
C-J6MT Keepstar, `structure_id 1049588174021` (Part 1 → Player-owned market
hubs) — just pass it as `station_id`:

```
curl "https://goonmetrics.apps.goonswarm.org/api/price_data/?station_id=1049588174021&type_id=34"
```
returned live, sensible data (`buy max 3.22`, `sell min 4.23`) at time of testing.
Confirming that also **upgrades the earlier "unverified" status on structure_id
1049588174021** in Part 1 — a made-up or wrong ID would not return coherent live
pricing here.

**Important nuance on the private-structure problem:** raw unauthenticated ESI
calls to this same structure_id (`/markets/structures/1049588174021/` and
`/universe/structures/1049588174021/`) still return `401 Unauthorized` when
tested directly — the structure itself is **not** publicly queryable via ESI.
Goonmetrics can serve it because it's a Goonswarm-operated tool holding its own
authenticated character token with access, and it re-publishes the result
through its own public API. So this doesn't make the structure's raw ESI data
public — it means **Goonmetrics functions as a read-only public proxy for
Goonswarm's own home market**, which is enough to solve price-checking for this
one hub without the operator needing their own docked-character token. It does
**not** extend to assets, jobs, or anything beyond price_data/price_history for
that structure.

## Cross-verified caveat: PLEX doesn't behave like a normal item

Querying PLEX (`typeID 44992`) against The Forge returned **all zeros** on both
Fuzzwork and EVE Tycoon independently:
```
curl "https://market.fuzzwork.co.uk/aggregates/?region=10000002&types=44992"
curl "https://evetycoon.com/api/v1/market/stats/10000002/44992"
```
Two independent aggregators agreeing on zero is a real signal, not a fluke: PLEX
trades through its own separate universal market mechanism, not through normal
per-region player buy/sell orders like everything else in this file. Don't be
surprised by a zero/empty result for PLEX from any of these three sources — it's
expected, not a bug in the query.

## Using these in Google Sheets

- **Goonmetrics (XML)** works with Sheets' native `IMPORTXML` — no Apps Script
  needed:
  ```
  =IMPORTXML("https://goonmetrics.apps.goonswarm.org/api/price_data/?station_id=60003760&type_id=34", "//sell/min")
  ```
- **Fuzzwork and EVE Tycoon (JSON)** — Sheets has no native JSON import function.
  Use Apps Script: a small custom function wrapping
  `UrlFetchApp.fetch(url).getContentText()` and `JSON.parse(...)`, called from
  the sheet like a normal formula. `IMPORTDATA` will fetch the raw text but won't
  parse JSON into cells for you.
- Whichever source, **cache/throttle in the sheet** (e.g. a manual refresh
  button or a time-based trigger, not a formula recalculating on every open) —
  these are shared community services, not the operator's own infrastructure;
  don't hammer them.

## Etiquette / stability notes

None of these three publish a documented rate limit, unlike ESI's explicit error
budget (Part 1). Treat that as "no published limit" rather than "no limit" — keep
call volume proportional to "a few specific items," per the stated goal, not a
scripted loop over hundreds of types. If a heavier need comes up later, that's a
"build a real sync against ESI/EVEFORGE" problem, not a "call these three
harder" problem.
