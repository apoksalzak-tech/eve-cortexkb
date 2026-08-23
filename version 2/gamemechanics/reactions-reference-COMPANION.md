# Companion Notes — EVE Reactions & T2 Industry Reference

## 1. Why this file exists
Created 2026-08-23 from a request to consolidate scattered reaction knowledge into one document usable by both David and an AI assistant: broker fee vs sales tax confusion (a recurring forget-point), Tatara bonus math, production balancing for reactions, profitability framing, and JF logistics costing. The trigger was wanting a single "full context" doc rather than re-deriving these numbers every planning session. It connects directly to the ongoing 400-slot T2 manufacturing operation planning (PI supply chains, T2 module candidate list, alt empire scale).

## 2. Suggested placement & next steps
- **Placement:** Obsidian vault, alongside the T2 manufacturing planning notes — e.g. `EVE/Industry/reactions-reference.md`. If it will feed AI context (Claude Projects, Konshus, or Claude Code on the Hetzner box), keep the ⚠️ VERIFY flags intact so any AI consuming it knows which numbers are load-bearing vs. patch-volatile.
- **Immediate verification pass (30 min in-game):** open the Industry window on 2–3 formulas you actually run and confirm (a) base time, (b) input/output quantities, (c) the material reduction shown vs. the §4a formula. Do the same for one live JF route on DOTLAN. Once checked, replace the VERIFY flags with dated confirmations.
- **Natural extension:** a small spreadsheet or script that takes formula quantities + Jita prices + your SCI and outputs margin per slot-day — the doc gives the equations; automating them is the next step. This would pair well with the existing Claude Code setup on apokavkos.com.
- **Open questions for future review:**
  - Is splitting reaction load across 2–3 systems worth the logistics overhead to keep SCI down at your scale? (§4c flags this but doesn't answer it.)
  - Molecular-forged reactions were only sketched — if capital component production enters the plan, that section needs real depth.
  - Booster (biochemical) reactions are named but not covered — deliberately out of scope unless you're running gas chains.

## 3. Honest caveats
- **Specific per-formula quantities are intentionally absent.** Exact input/output numbers for individual reactions (e.g., how much Titanium Chromide per run) were not included because AI recall of SDE data is unreliable and CCP adjusts it. The doc gives the *math structure* and points to EVE Ref/in-game as the source of truth. Don't let a future AI "helpfully" fill those tables from memory — pull from the SDE.
- **Tax/fee numbers were verified against current sources (Aug 2026)** — sales tax 7.5%/3.375%, broker 3%/1.5%, Tatara 25%, rigs 2%/2.4% with 1.1 null multiplier, JF skill stacking. The SCC surcharge (stated 4%) had the weakest sourcing and is the most likely number to be wrong or stale.
- **The ~18 runs/slot/day figure** assumes a 3h base time and full bonuses; it's illustrative, not authoritative — real formulas vary.
- **Profitability section is a framework, not a calculator.** It won't tell you *which* composites are profitable this month; it tells you how to evaluate any of them without fooling yourself (replacement-cost pricing, endogenous SCI, both-direction logistics).
- No coverage of risk costing beyond a passing mention — JF gank exposure, structure vulnerability windows, and war-dec/entosis risk to the Tatara itself are real EV modifiers not modeled here.
