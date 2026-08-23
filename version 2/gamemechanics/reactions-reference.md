# EVE Online Reactions & T2 Industry Reference
### Personal + AI context document — David's industrial operation
*Last verified: August 2026. Volatile numbers are flagged with ⚠️ VERIFY — CCP changes tax/fee rates in balance patches.*

---

## 1. The Full Production Chain (where reactions sit)

```
Moon ore (mined at Athanor/Tatara moon drill)
  → Reprocessing → Raw moon materials ("moon goo", 20 types)
    → SIMPLE REACTIONS → Intermediate materials (21 types)
      → ADVANCED (COMPOSITE) REACTIONS → Composite materials (15 types)
        → MANUFACTURING (blueprints, not reactions) → Construction components ("T2 components", 34 types)
          → MANUFACTURING → T2 ships/modules
```

Key distinctions:
- **Reactions use Formulas**, not blueprints. Formulas have infinite runs, cannot be researched or copied. There is **no ME/TE research** — the only material/time savings come from structure, rigs, and skills.
- **Reactions require security ≤ 0.4** (lowsec, nullsec, wormholes). No highsec reactions, period.
- **Construction components are manufacturing**, not reactions — they run in an engineering complex (Raitaru/Azbel/Sotiyo) with normal blueprint ME/TE.

### Reaction families & their service modules
| Family | Service module | Products |
|---|---|---|
| Simple + Advanced (Composite) | Standup Composite Reactor I | Intermediates → composite materials for T2 |
| Biochemical | Standup Biochemical Reactor I | Booster intermediates → boosters |
| Hybrid | Standup Hybrid Reactor I | Hybrid polymers for T3 production |
| Molecular-Forged | (Composite reactor) ⚠️ VERIFY module | Isogen-10/Zero-Point Condensate reactions for capital components |

Rigs are **reactor-type specific** — a Composite ME rig does nothing for Hybrid reactions.

---

## 2. Structure Choice: Athanor vs Tatara

| | Athanor (M) | Tatara (L) |
|---|---|---|
| Reaction time bonus | none | **–25% reaction job time** |
| Reactor service fuel | base | **–25% service module fuel** |
| Refining role bonus | 2% | 4% |
| Rig slots | M-Set rigs | L-Set rigs (L rigs affect BOTH material and time) |
| Docking | subcap + freighter | subcap + freighter + Rorqual |

**Why Tatara for a serious reaction farm:** the 25% time bonus is effectively +33% throughput per slot, and the fuel discount compounds daily. At 400-slot scale the Tatara time bonus is the single biggest structural lever you have.

### Rig bonuses (Reaction rigs)
| Rig | Material reduction | Time reduction |
|---|---|---|
| T1 | 2.0% | 20% |
| T2 | 2.4% | 24% |

**Security multiplier applied to rig bonus:**
- Lowsec: ×1.0
- Nullsec / WH: ×1.1

So a **T2 material rig in nullsec = 2.64% material reduction**, and a **T2 time rig in null = 26.4% time reduction**. (Medium rigs do one stat; Large rigs on a Tatara do both.)

---

## 3. Skills

| Skill | Effect | Notes |
|---|---|---|
| Reactions | –4% reaction job time per level | –20% at V |
| Mass Reactions | +1 reaction slot per level | |
| Advanced Mass Reactions | +1 reaction slot per level | Requires Mass Reactions V |
| (Total slots) | 1 + 5 + 5 = **11 per character** | This is what defines your alt count for 400 slots (~37 chars at max skills) |

---

## 4. The Core Math

### 4a. Material consumption per job
```
materials_used = ceil( runs × base_qty × (1 − rig_bonus × sec_multiplier) )
```
- Rounding is applied **per job**, not per run — bigger jobs waste fractionally less. (This is why in-game numbers come out slightly lower than naive per-run math.)
- Nullsec Tatara/Athanor with T2 mat rig: multiplier = 1 − (0.024 × 1.1) = **0.9736**

### 4b. Reaction job time
```
job_time = base_time × runs
         × (1 − 0.25 if Tatara)
         × (1 − 0.04 × Reactions_level)
         × (1 − rig_TE × sec_multiplier)
```
Example — nullsec Tatara, Reactions V, T2 time rig:
`3h × 0.75 × 0.80 × 0.736 = ~1h 19m per run` — call it **~18 runs/slot/day** vs 8/day unbonused. ⚠️ VERIFY base_time per formula in-game (most composite reactions are 3h base, but check each).

### 4c. Job installation cost (the "tax" people forget)
```
job_cost = EIV × (system_cost_index + facility_tax + SCC_surcharge)
```
- **EIV** (Estimated Item Value) = CCP's adjusted price of the *input materials* — not market price, pulled from CCP's monthly-ish adjusted price data.
- **System Cost Index (SCI)** for reactions is driven by *all reaction activity in that system* — your own 400 slots inflate your own index. This is a real cost at your scale; consider whether splitting across systems beats convenience.
- **Facility tax**: set by structure owner (yours = whatever you set; alliance structures vary).
- **SCC surcharge**: flat NPC surcharge on all industry jobs — **4%** ⚠️ VERIFY (CCP has moved this: 1.5% → 4%; check current patch notes).

---

## 5. Market Fees: Broker Fee vs Sales Tax (the one you keep forgetting)

**Two totally separate charges. One is for *listing*, one is for *selling*.**

### Broker fee — paid when you CREATE a non-immediate order (buy or sell)
- Charged **up front**, on the full order value, **not refunded** if the order expires or is cancelled.
- **NPC stations:** base 3.0%, reduced 0.3%/level by Broker Relations → **1.5% at BR V**; faction/corp standings can push it to a floor of **1.0%** (unmodified standings only — Connections doesn't help).
- **Player structures (Upwell):** fee is set by the structure owner. **Broker Relations does NOT apply in player structures.** 0.5% of the fee goes to the SCC as an ISK sink.
- **Relist fee:** every price edit costs ISK; Advanced Broker Relations reduces relist fees (–5 percentage points/level off the relist penalty). If you update orders often, this skill pays for itself.

### Sales tax — paid when a sale actually EXECUTES (seller pays)
- Base **7.5%**, reduced 11%/level by Accounting → **3.375% at Accounting V**.
- Applies everywhere, including player structures.
- Deducted automatically from the transaction.

### Quick mental model
| | When | Base | Max skills | Where it applies |
|---|---|---|---|---|
| Broker fee | Order creation | 3% (NPC) | 1.5% (1.0% w/ standings) | NPC; structures = owner-set, skills don't help |
| Sales tax | Sale executes | 7.5% | 3.375% | Everywhere |

**Total friction selling in Jita at max skills: ~4.875%** (1.5% + 3.375%). Selling to buy orders (immediate) skips the broker fee → 3.375% only. That ~1.5% spread is often the entire difference between sell-order margin and instant-sell margin on composites.

If you *buy inputs via buy orders*, add the broker fee on the buy side to input cost. If you buy from sell orders, no fee — the price is the price.

---

## 6. Profitability Framework

```
Profit = Revenue − Input cost − Job fees − Market fees − Logistics − (Fuel overhead share)

Revenue      = units × sell_price × (1 − sales_tax − broker_fee_if_sell_order)
Input cost   = Σ inputs × price × (1 + broker_fee_if_bought_with_buy_orders)
Job fees     = EIV × (SCI + facility_tax + SCC)
Logistics    = JF fuel + collateral risk allocation (both directions!)
Fuel overhead = daily fuel block burn of reactor services ÷ daily output
```

Decision rules worth writing down:
1. **Always price inputs at replacement cost (Jita), not what you paid.** If moon goo is worth X in Jita, reacting it only "profits" if the margin beats selling the goo raw and doing nothing.
2. **Compare against the null alternative at every chain step:** sell goo raw vs sell intermediates vs sell composites vs sell T2 components vs sell hulls. Each step must justify itself; sometimes the composite→component step is where all the margin lives and hulls are charity.
3. **Reactor service fuel is a fixed daily cost** (~fuel blocks/day per service, –25% on Tatara). Divide it across daily throughput — it matters at low utilization, vanishes at full 400-slot utilization. Keep slots full or shut services down.
4. **Your own SCI is endogenous** — profitability calculated at today's index understates cost if you scale up in one system.

---

## 7. Jump Freighter Logistics Math

### Fuel per jump
```
isotopes = base_consumption_per_LY × distance_LY × (1 − 0.10 × JF_skill) × (1 − 0.10 × JFC_skill)
```
- **Jump Freighters skill** and **Jump Fuel Conservation** each: –10%/level, multiplicative with each other.
- All-V pilot burns **0.25× base**; JF V + JFC IV burns 0.30×; both at IV burns 0.36×.
- **Jump Drive Calibration:** +20% max range per level. Base 5 LY → **10 LY at JDC V**.
- Each hull uses its racial isotope (Ark = Helium, Rhea = Nitrogen, Anshar = Oxygen, Nomad = Hydrogen) and has its own base consumption — Rhea/Anshar thirstiest, Nomad cheapest. ⚠️ Pull exact base rates per hull from DOTLAN jump planner rather than memory.

### Cost per m³ (the number that matters)
```
cost_per_m3_per_trip = (isotopes_burned × isotope_price + cyno_ship_writeoff_amortized) ÷ cargo_m3_actually_used
```
- **An empty or half-full JF has the same fuel bill** — cost per m³ doubles at 50% fill. Batch hauls to full loads.
- **Count both directions.** Goo inbound + product outbound; if one leg flies light, its cost loads onto the other leg.
- Allocate to product: `logistics_cost_per_unit = cost_per_m3 × unit_volume`. Composites are dense ISK/m³ so this is usually small — but raw goo inbound is the bulky leg. **Reacting near the goo source and hauling compressed value outward is almost always right.**
- Isotope price volatility is a real input cost — track it like any other material.

---

## 8. Tatara Batch Planning (400-slot scale)

Planning invariants:
- **Simple → Advanced pairing:** the standard composite chain is designed so simple (intermediate) reaction output feeds advanced reactions at matching rates with equal base times — the classic planning rule is ~1 simple slot feeding 1 advanced slot for a balanced line. ⚠️ VERIFY per-formula quantities in-game or on EVE Ref (everef.net) — do not trust memory or AI recall for exact input/output quantities; pull them from the live SDE.
- **Advanced reactions also consume fuel blocks** per run — fuel blocks are a *material input* to composites, separate from structure service fuel. Price them in.
- **Runs per slot per day** (from §4b): with full bonuses ~18 runs/day/slot for a 3h-base formula. Daily line capacity = slots × runs/day.
- **Sizing a balanced line:** pick the target composite output/day → back out advanced runs/day → back out intermediate runs/day → back out raw goo tonnage/day → that's your moon mining + import requirement and your JF schedule.
- Practical rhythm: set job durations to your real login cadence (multi-run jobs finishing when you're actually at the keyboard beats theoretical max throughput you can't restart).

---

## 9. Numbers to Re-Verify After Any Industry Patch
- [ ] SCC surcharge % on industry jobs
- [ ] Sales tax base % and Accounting reduction
- [ ] Broker fee base % / structure minimums
- [ ] Rig bonus values & security multipliers
- [ ] Tatara reaction time bonus
- [ ] Per-formula input/output quantities and base times (SDE / EVE Ref)
- [ ] JF base isotope consumption per hull

## 10. External Tools
- **EVE Ref (everef.net)** — canonical formula quantities, live SDE
- **DOTLAN jump planner** — route + exact fuel per jump for your skills
- **Fuzzwork** — blueprint/reaction calculators, adjusted prices (EIV source data)
- **Adam4EVE / EVE Tycoon** — margin tracking on composites
