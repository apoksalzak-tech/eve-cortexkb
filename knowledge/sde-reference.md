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
