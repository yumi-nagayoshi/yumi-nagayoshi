# EUR Receivable Hedge Model — Technical Specification

| Field | Value |
|---|---|
| **Created by** | Yumi Nagayoshi |
| **Date Created** | 2026-07-28 |
| **Version** | 0.1 (draft) |
| **LLM Used** | Claude (Anthropic) — drafted from Stage 1 memo + `scenarios.md`, Scenario 1 |
| **Role** | Treasury Analyst |
| **Audience** | CFO / Director of Treasury |
| **Companion document** | `2026-07-21-Nagayoshi-eur-receivable-solar-importer-hedge-framing.md` (Stage 1 memo) |

---

## 1. Problem Statement

The company (a U.S. solar equipment importer) holds a EUR-denominated receivable settling in one year. At today's spot rate, the receivable is worth approximately $4.5M USD-equivalent; a depreciation of EUR against USD over the horizon would reduce realized USD proceeds and compress the margin on the underlying sale. This specification documents the analytical framework used to quantify and compare four approaches — **no hedge**, **forward hedge**, **money-market hedge**, and **option hedges (protective put and zero-cost-structured collar)** — and to generate the sensitivity evidence supporting the hedge recommendation carried forward from the Stage 1 memo.

**Decision context:** Treasury analysis for CFO review; deal terms are quoted by the counterparty bank (forward, put, call), and the money-market leg is priced off the company's own USD/EUR funding and investment rates.

---

## 2. Inputs — Named-Range Contract

All values below are **indicative — replaced with live market data at Stage 4.** Rates and spot reflect market close near July 20–21, 2026 as cited in the Stage 1 memo; forward and option premiums are the counterparty's quoted deal terms from `scenarios.md`, Scenario 1.

| Named Range | Description | Unit | Placeholder Value | Stage-4 Data Source |
|---|---|---|---|---|
| `FC_AMT` | EUR receivable notional | EUR | 3,947,368 | Confirm actual contracted EUR notional with the underlying sales agreement (see flag below) |
| `S0_in` | Spot rate at inception | USD per EUR | 1.1400 | Live EURUSD spot, e.g. Bloomberg/Reuters, at execution |
| `F0_in` | Forward rate, 1-year | USD per EUR | 1.0875 | Counterparty bank forward quote at execution |
| `R_USD` | USD interest rate | Annual %, ACT/360 | 4.03% | U.S. Treasury 1Y CMT (or equivalent USD funding rate) at execution |
| `R_FC` | EUR interest rate | Annual %, ACT/360 | 2.88% | 12-month Euribor at execution |
| `K_PUT` | Put option strike | USD per EUR | 1.1400 (at spot) | Set at/near live spot at execution |
| `K_CALL` | Call option strike | USD per EUR | 1.1400 (at spot) | Set at/near live spot at execution |
| `PREM_PUT` | Put premium per unit of FC | USD | 0.015 | Counterparty option quote at execution |
| `PREM_CALL` | Call premium per unit of FC | USD | 0.018 | Counterparty option quote at execution |
| `T_DAYS` | Days to settlement | Days | 365 | Actual days from trade date to settlement date |

**⚠ Flag carried from Stage 1:** `FC_AMT` is derived by translating the scenario's stated "$4,500,000 receivable" into a EUR notional at `S0_in` (4,500,000 / 1.1400 ≈ €3,947,368), on the reading that the scenario states a USD-equivalent value rather than a EUR face amount. This is the single largest open assumption in the model — if the underlying sales contract instead specifies a EUR face amount directly, `FC_AMT` and every downstream USD figure change. Confirm before Stage 3.

---

## 3. Tab Architecture

| Tab | Purpose |
|---|---|
| **Cover** | Scenario description, author, version, data sources with access dates |
| **Legend/Key** | Color convention: yellow = input, blue = assumption, black = formula, green = cross-tab link |
| **Inputs** | All ten named ranges above, laid out as the single source of truth for every calculation tab |
| **Forward** | Forward-hedge calculation (§5, Step 1) |
| **MoneyMarket** | Three-step money-market hedge calculation and the parity check (§5, Step 2) |
| **Options** | Protective put and collar payoff calculations across the settlement-spot grid (§5, Steps 3–4) |
| **Sensitivity** | Consolidated ±5% sensitivity grid and comparison chart across all four strategies (§6) |
| **Notes & Assumptions** | Rate basis, data sources with timestamps, and every simplification listed in §4 |

---

## 4. Assumptions & Constraints

- **Quote convention:** All rates are USD per EUR; a higher quote means EUR appreciation.
- **Rate basis:** `ACT/360` applied to both `R_USD` and `R_FC` (single-basis simplification — a more rigorous build would split `BASIS_USD`/`BASIS_FC` if EUR-side conventions differ; flagged here rather than built, per scope).
- **Horizon:** Single-maturity model, `T_DAYS = 365`. No partial-year or multi-tranche hedging.
- **Parity expectation:** The money-market hedge is assumed to replicate the forward hedge under covered interest-rate parity. Note: at the placeholder rates above, parity implies a forward near 1.153 (`S0_in × (1+R_USD×T/360)/(1+R_FC×T/360)`), materially above the counterparty's quoted `F0_in` of 1.0875. This gap is carried forward from the Stage 1 memo as an open item — verify the quoted forward before execution rather than treating the model's parity check as evidence of a formula error.
- **Premium treatment:** Option premiums are paid (put) or received (call) upfront in USD, quoted per 1 unit of FC, no contract multiplier. Premiums are carried forward to the settlement date at `R_USD` so they sit on the same footing as settlement-date proceeds.
- **Collar structure:** Both `K_PUT` and `K_CALL` are set at spot (`S0_in`), consistent with the scenario's "strike at/near spot" instruction. This produces a near-zero-width collar: proceeds are effectively capped and floored at the same level, offset by the net premium (call premium received minus put premium paid).
- **Transaction costs, bid-ask spreads, counterparty/credit risk, and tax treatment:** excluded from this base-case model.
- **Scenario construction:** `S_T` is varied deterministically on a grid; no probability weighting or implied-volatility distribution is applied.

---

## 5. Calculation Flow

All formulas in named-range notation; no cell addresses.

**Step 0 — Derived accumulation factors**
```
DF_USD = 1 + R_USD × T_DAYS/360
DF_FC  = 1 + R_FC × T_DAYS/360
```

**Step 1 — Forward hedge**
```
USD_FWD = FC_AMT × F0_in
```
Locked at inception; constant across the sensitivity grid.

**Step 2 — Money-market hedge (three explicit steps)**
```
Step 2a  Borrow EUR today:   Borrow = FC_AMT / DF_FC
         (sized so the receivable exactly repays the loan at settlement)
Step 2b  Convert at spot:    USD_now = Borrow × S0_in
Step 2c  Invest in USD:      USD_MM = USD_now × DF_USD
```
**Parity check:** `F_implied = S0_in × DF_USD / DF_FC` should ≈ `F0_in`. Per §4, this deal shows a material gap — document it, don't silently reconcile it.

**Step 3 — Protective put (option hedge, receivable side)**
```
FV_PREM_PUT = −PREM_PUT × FC_AMT × DF_USD
USD_PUT(S_T) = MAX(S_T, K_PUT) × FC_AMT + FV_PREM_PUT
```
Below `K_PUT`, the put pays the difference (floor holds); above it, the option expires worthless and the receivable converts at the better market rate, net of the premium paid either way.

**Step 4 — Collar (put purchased + call sold, extension beyond the minimum three hedges)**
```
FV_NET_PREM_COLLAR = (PREM_CALL − PREM_PUT) × FC_AMT × DF_USD
USD_COLLAR(S_T) = MIN(MAX(S_T, K_PUT), K_CALL) × FC_AMT + FV_NET_PREM_COLLAR
```
Since `K_PUT = K_CALL = S0_in` in this scenario, proceeds are effectively fixed near `FC_AMT × S0_in`, adjusted by the net premium credit — narrower and, at these placeholder rates, higher than the outright forward.

**Step 5 — No-hedge baseline**
```
USD_NO_HEDGE(S_T) = S_T × FC_AMT
```

**Step 6 — Sensitivity grid row logic**
For each `S_T` on the grid (§6): compute `USD_NO_HEDGE`, `USD_FWD` (constant), `USD_MM` (constant), `USD_PUT(S_T)`, `USD_COLLAR(S_T)`; then `HEDGE_PROFIT_k(S_T) = USD_k − USD_NO_HEDGE(S_T)` for each strategy `k`; then label the row's `ARGMAX` across all strategies and the `ARGMAX` restricted to active hedges only (excluding no-hedge).

---

## 6. Sensitivity Plan

- **Grid:** `S_T` from `0.95 × S0_in` to `1.05 × S0_in` in 1% steps → 11 points: 1.0830, 1.0944, 1.1058, 1.1172, 1.1286, **1.1400 (baseline)**, 1.1514, 1.1628, 1.1742, 1.1856, 1.1970.
- **Strategies plotted:** no hedge, forward, money market, protective put, collar.
- **Chart:** one line chart, `S_T` on the x-axis, USD proceeds on the y-axis. Forward and money-market lines are flat by construction; no-hedge is a straight line through the origin; put is piecewise-linear with a kink at `K_PUT`; collar is nearly flat given the at-spot strikes.
- **What it should let the CFO see:** the trade-off between certainty (forward/MM, flat), bounded optionality (put, kinked floor with open upside; collar, floor and cap near the same level), and unbounded exposure (no hedge) — and visually, how far the quoted forward sits below every other strategy across the grid, which is the crux of the Stage 1 recommendation.

---

## 7. Validation Rules (Check Figures)

- [ ] `USD_MM` and `F_implied` (§5, Step 2) are computed and displayed even though they are **not** expected to match `USD_FWD`/`F0_in` closely in this scenario — the gap itself is the check figure Treasury needs to see, not an error.
- [ ] `USD_PUT(S_T)` at `S_T = K_PUT` equals `K_PUT × FC_AMT + FV_PREM_PUT` exactly (kink verification).
- [ ] `USD_COLLAR(S_T)` is flat (within rounding) across the full grid, since `K_PUT = K_CALL`.
- [ ] At baseline (`S_T = S0_in`): `USD_PUT` ≈ $4,438,000 (FC_AMT × spot, less the future-valued $0.015 premium ≈ $61.6K) and `USD_COLLAR` ≈ $4,512,000 (FC_AMT × spot, plus the future-valued $0.003 net credit ≈ $11.8K) — both should reproduce to within rounding of the Stage 1 memo's cited figures ($59K put premium, ~$12K collar credit, ~$4.51M cap).
- [ ] No error cells (`#DIV/0!`, `#REF!`, etc.) anywhere on the Sensitivity tab.
- [ ] Every output cell is a formula referencing named ranges — no hard-coded results.
- [ ] Sensitivity grid is symmetric around `S_T = S0_in` and driven by a single step-size input, not hard-coded per row.

---

## 8. Outputs

| Output | Description | Location |
|---|---|---|
| `USD_FWD`, `USD_MM`, `F_implied` | Forward and money-market summary figures, plus the parity comparison | Forward / MoneyMarket tabs |
| `USD_PUT_BASE`, `USD_PUT_FLOOR` | Put proceeds at baseline (`S_T = S0_in`) and worst-case floor (`MIN` across the grid) | Options tab |
| `USD_COLLAR_BASE` | Collar proceeds at baseline | Options tab |
| Sensitivity table | USD proceeds for all five strategies across the `S_T` grid, plus `HEDGE_PROFIT_k` columns | Sensitivity tab |
| Winner / best-active-hedge labels | Per-row `ARGMAX` labels | Sensitivity tab |
| Sensitivity chart | Line chart, USD proceeds vs. `S_T`, one series per strategy | Sensitivity tab |
| Strategy summary block | Base-case USD proceeds and hedge profit vs. no-hedge, one row per strategy | Sensitivity tab, top |

---

## Open Items Carried Into Stage 3

1. **`FC_AMT` ambiguity** (§2) — highest-priority confirmation before build.
2. **Forward-vs-parity gap** (§4, §7) — verify the counterparty's quoted forward rather than assuming a model error.
3. Rate basis simplified to single `ACT/360` for both legs; revisit if EUR-side convention differs materially at execution.
