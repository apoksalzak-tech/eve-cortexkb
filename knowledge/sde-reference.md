---
title: EVE Online SDE Reference (Fuzzwork export)
source: https://www.fuzzwork.co.uk/dump/latest/
observed: 2026-06-30 (dump dated 2026-06-26)
---

# EVE Online SDE Reference

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

Columns below were pulled from each CSV's actual header row.

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
`GET /industry/systems/`, public) × structure/facility tax settings. See
EVEFORGE's own Settings → Facilities screen (`eveforge-site.md`) for the SCC
Industry Tax / Facility Tax split it exposes there.

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
(structure IDs)" in `esi-api-index.md` — different lookup mechanism entirely
(structure_id + authenticated token, not region_id + public call).

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
  (see `esi-api-index.md` → Dogma) returns the item's real, rolled attributes and
  **completely overrides** anything `dgmTypeAttributes` would say for that
  specific item instance. If a user mentions "abyssal," "mutated," or a
  mutaplasmid name, assume static SDE attribute data does not apply — the
  dynamic-items endpoint (or an ESI-synced tool like EVEFORGE) is required.

## Practical notes

- **typeID is the universal join key.** Almost every question ("what does this
  blueprint need," "what does this module do," "what system is this station in")
  resolves to joining through `invTypes.typeID`. If you're starting from an item
  *name* rather than an ID, resolve it to a typeID first — see "Name → typeID
  resolution" in `esi-api-index.md` (or `invTypes.typeName` if working offline
  from the SDE alone). Never guess an ID from the name.
- **The SDE has no prices.** `invTypes.basePrice` exists but is a vestigial
  NPC-seed value, not a market price — never treat it as current. Live prices only
  come from ESI market endpoints (see `esi-api-index.md`) or a market tool like
  EVEFORGE.
- **Security status** for reprocessing/industry bonuses comes from
  `mapSolarSystems.security` (system) and `staStations.reprocessingEfficiency` /
  `staStationTypes.reprocessingEfficiency` (station type) — matches EVEFORGE's
  Resource Calculator "Operating Zone: Hi-Sec / Low-Sec / Null-WH" selector.
