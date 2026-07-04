---
title: EVE Swagger Interface (ESI) — API Index
source: https://esi.evetech.net/latest/swagger.json
spec_version: "1.36"
fetched: 2026-06-30
---

# ESI API Index

ESI (EVE Swagger Interface) is CCP's official REST API for EVE Online. It's the
data backbone under EVEFORGE (and every other third-party industry/market tool):
character/corp data, assets, industry jobs, market orders, and static universe data
all come from here. This file is a structural index of **what data exists and how
to get it** — not live data itself.

- Base URL: `https://esi.evetech.net/latest`
- Full machine-readable spec: `https://esi.evetech.net/latest/swagger.json`
  (this index was generated from spec version 1.36 on 2026-06-30 — endpoints are
  fairly stable release to release, but **re-fetch the swagger.json if precision
  matters**, don't trust this list blindly on a long-lived assistant session)
- Interactive docs / try-it-now UI: `https://esi.evetech.net/ui/`
- Auth: EVE SSO (OAuth2). Public endpoints need no token; scoped endpoints need a
  token carrying the specific scope named below. Scopes look like
  `esi-assets.read_assets.v1` and are requested per-character at SSO login —
  EVEFORGE's own requested scope set is documented in
  `archive/eveforge/site-map.md` under Characters.
- 180 endpoints total across 29 tags, as of spec 1.36.

## What this file does NOT define: how authentication actually happens

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

## How to read an entry

`` `GET /characters/{character_id}/assets/` — Get character assets [scope: esi-assets.read_assets.v1] ``

means: HTTP GET, path takes a character_id, and the calling token must carry the
named scope. `[public]` means no token/scope required.

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
   see `sde-reference.md`) for a `typeName` match — exact match is safest, EVE
   item names are unique. Use this when you only have the SDE loaded and no live
   ESI access.

Do not guess a typeID from memory or pattern-match a name to a "likely" ID —
type IDs aren't predictable from the name, and a wrong ID silently returns data
for the wrong item with no error. Lock in the correct typeID before doing
anything downstream with it (blueprint materials via
`industryActivityMaterials`, module attributes via `dgmTypeAttributes`, market
data via `/markets/{region_id}/orders/`, etc.).

## Endpoint index by category

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
  (The Forge) — see the trade-hub → region ID table in `sde-reference.md`.
- Corp-level assets/jobs/wallet/blueprints require a **director** role token, not
  just any corp member — matches what EVEFORGE's Corporation Hub says.

## Player-owned market hubs (structure IDs)

Unlike the five NPC trade hubs (see the region-ID table in `sde-reference.md`),
which are public and keyed by region, player-owned Upwell structures (Keepstars,
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
**since corroborated**: querying Goonmetrics' public price API
(`market-data-lookups.md`) with this exact structure_id as `station_id` returns
live, coherent buy/sell data, which a wrong or made-up ID would not do. Treat the
ID itself as confirmed. Raw unauthenticated ESI calls to
`/markets/structures/1049588174021/` and `/universe/structures/1049588174021/`
still return `401 Unauthorized` though — the structure is **not** publicly
queryable via ESI directly; Goonmetrics can serve price data for it because it
holds its own authenticated character token and re-publishes the result. **Full
access (assets, jobs, anything beyond price_data) is still gated by docking
rights and scopes on the querying character — the operator has flagged that as
an open problem to solve separately.** See `market-data-lookups.md` for the
price-only workaround that's already solved.
