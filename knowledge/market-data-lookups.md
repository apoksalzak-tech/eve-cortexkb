---
title: Public Market Data Lookups (No Auth Required)
status: verified working, 2026-07-01 — all example calls below were actually run
---

# Public Market Data Lookups (No Auth Required)

Purpose: get the **current price of one or a few specific items** — for an agent
answering a quick question, or for pulling a number into a Google Sheet — without
standing up a database and without EVE SSO/OAuth2. This is deliberately scoped
small: a handful of items, not a market-wide sync (EVEFORGE, or the raw ESI
`markets/{region_id}/orders/` endpoint in `esi-api-index.md`, are the right tools
for that).

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
C-J6MT Keepstar, `structure_id 1049588174021` (see `esi-api-index.md` → Player-
owned market hubs) — just pass it as `station_id`:

```
curl "https://goonmetrics.apps.goonswarm.org/api/price_data/?station_id=1049588174021&type_id=34"
```
returned live, sensible data (`buy max 3.22`, `sell min 4.23`) at time of testing.
Confirming that also **upgrades the earlier "unverified" status on structure_id
1049588174021** in `esi-api-index.md` — a made-up or wrong ID would not return
coherent live pricing here.

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
budget (`esi-api-index.md`). Treat that as "no published limit" rather than "no
limit" — keep call volume proportional to "a few specific items," per the stated
goal, not a scripted loop over hundreds of types. If a heavier need comes up
later, that's a "build a real sync against ESI/EVEFORGE" problem, not a "call
these three harder" problem.
