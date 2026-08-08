# Market-Data Memo — Stage 4 Population

| Field | Value |
| --- | --- |
| **Prepared by** | Yumi Nagayoshi |
| **Retrieval date** | 2026-08-07 |
| **Companion documents** | `2026-07-28-nagayoshi-eur-receivable-solar-importer-spec.md` (Stage 2), `2026-08-02-nagayoshi-eur-receivable-solar-importer-model.xlsx` (Stage 3) |

---

## 1. Inputs — source, value, timestamp

| Named Range | Value | Source | Retrieval Timestamp | Proxy / Computation Logic |
| --- | --- | --- | --- | --- |
| `S0_in` | 1.1542 | ECB euro foreign exchange reference rate (USD) | Fix published 06 Aug 2026, 14:15 CET — most recent ECB print available as of retrieval on 07 Aug 2026 (ECB publishes ~16:00 CET, so the 07 Aug fix had not yet posted at time of retrieval) | Cross-checked against live intraday quotes (~1.154–1.156 on 07 Aug 2026 per TradingEconomics, Myfxbook); consistent within normal daily range. No proxy — direct quote. |
| `R_USD` | 4.01% | U.S. Treasury Daily Par Yield Curve Rates, 1 Yr Constant Maturity Treasury | 08/07/2026 | Direct quote, no proxy. Chosen over SOFR/deposit rates because CMT is the standard USD funding-rate proxy for a corporate treasury model, consistent with the Stage 2 spec's original sourcing choice. |
| `R_FC` | 2.930% | 12-month Euribor | Published 05 Aug 2026 (most recent available; this source publishes with an approximate 1-day lag) | Direct quote, no proxy. Matches Stage 2 spec's stated instrument (12-month Euribor). |
| `F0_in` | 1.1665 | **Computed — CIP-implied forward**, not a counterparty quote | Computed 07 Aug 2026 from `S0_in`, `R_USD`, `R_FC` above | `F0 = S0 × (1 + R_USD×T/360) / (1 + R_FC×T/360)` = 1.1542 × 1.040657 / 1.029707 = 1.1665. **No live 1-year EURUSD outright forward quote was accessible through free sources** (Barchart, investing.com, and Bluegamma all required a paid subscription or terminal login). See §3 for why this materially changes what the parity check can show. |
| `K_PUT` | 1.1542 | = live `S0_in` | 07 Aug 2026 | Set at spot per scenario convention ("strike at/near spot"). |
| `K_CALL` | 1.1542 | = live `S0_in` | 07 Aug 2026 | Same. |
| `PREM_PUT` | $0.015 | Scenario-given (unchanged from Stage 2) | — | Kept as-is per Stage 4 guidance — retail-accessible option quotes are unreliable proxies for institutional deal terms. |
| `PREM_CALL` | $0.018 | Scenario-given (unchanged from Stage 2) | — | Same. |
| `FC_AMT` | €3,947,368 | Resolved assumption (see §2) | — | No underlying sales agreement exists to check this against. |
| `T_DAYS` | 365 | Scenario-given (unchanged) | — | — |

---

## 2. FC_AMT resolution (closing the Stage 1–3 open item)

The scenario states "$4,500,000 receivable in 1 year" with no accompanying sales agreement. Absent a contract that specifies a EUR face amount directly, `FC_AMT` is read as a USD-equivalent value, translated to EUR at the spot rate in force when the notional was originally fixed (`4,500,000 / 1.1400 ≈ €3,947,368`, per Stage 2 §2).

This is the resolution, not a continued open item: there is no sales agreement available in this exercise to confirm the alternative (a direct EUR face amount) against. If a real sales agreement existed, it would supersede this reading.

---

## 3. Parity check — why the Stage 1–3 signal collapses on live data

The forward-vs-money-market gap was the central finding of Stages 1–3: the counterparty's quoted `F0_in` (1.0875) sat materially below the parity-implied forward (~1.153), a real $258K dislocation worth flagging to Treasury.

**That signal does not survive Stage 4 population, and this is worth stating plainly rather than glossing over.** Because no live counterparty-quoted forward was available, `F0_in` here is computed as `S0_in × DF_USD/DF_FC` — the exact same formula used for the parity check itself (`F_implied`). Setting `F0_in = F_implied` by construction means `USD_FWD` and `USD_MM` reconcile to within $9 (rounding only). The parity check on this live dataset proves the formula is internally consistent, not that a real-world mispricing exists.

**Recomputed outputs (live data):**

| Output | Value |
| --- | --- |
| `F_implied` (CIP-implied forward) | 1.1665 |
| Original scenario `F0_in` (Stage 1–3, for reference) | 1.0875 |
| Gap between the two forwards | 0.0790 USD/EUR (~$312K on notional) — reflects **stale scenario terms vs. current market**, not a live dislocation |
| `USD_FWD` (live) | $4,604,502 |
| `USD_MM` (live) | $4,604,502 |
| Forward-vs-MM gap (live) | ~$9 (rounding only — parity holds by construction) |
| `USD_PUT_BASE` (S_T = S0_in) | $4,494,434 |
| `USD_COLLAR_BASE` | $4,568,376 |

**Conclusion:** the audit finding from Stage 3 doesn't disappear because it was wrong — it disappears because Stage 4's live `F0_in` is a computed proxy rather than a quoted deal term. A real Stage 4 execution would require a live counterparty or terminal forward quote (Bloomberg/Refinitiv) to reproduce a genuine parity test; that data source was not accessible here. This limitation is recorded rather than hidden.

---

## 4. FX Hedging Lab cross-check

**Performed 2026-08-07.** Live inputs from §1 were entered into the course FX Hedging Lab and its outputs compared against this workbook.

| Output | Workbook | Lab | Match? |
| --- | --- | --- | --- |
| `USD_FWD` | $4,604,605 | $4,604,605 | ✅ |
| Borrow (EUR, MM step 1) | 3,833,487 | 3,833,487 | ✅ |
| Convert at spot (MM step 2) | $4,424,610 | $4,424,610 | ✅ |
| `USD_MM` | $4,604,502 | $4,604,502 | ✅ |
| Implied forward (parity check) | 1.1665 | 1.1665 | ✅ |
| Parity gap | ~$0 | -$103 (rounding) | ✅ |
| `USD_PUT_BASE` (S_T = S0_in) | $4,494,434 | $4,496,842 | ❌ Δ = $2,408 |

**Forward and money-market outputs matched exactly** (within rounding). This confirms the workbook's core hedge mechanics are structurally sound on live data.

**The put-hedge baseline did not match**, and the discrepancy was traced to premium treatment, not a calculation error:

- This workbook future-values `PREM_PUT` to the settlement date at `DF_USD`: `USD_PUT = MAX(S_T,K_PUT)×FC_AMT − PREM_PUT×FC_AMT×DF_USD`. This follows Stage 2 spec §5 directly: *"Premiums are carried forward to the settlement date at R_USD so they sit on the same footing as settlement-date proceeds."*
- The lab appears to subtract the premium at face value with no time-value adjustment: `USD_PUT = MAX(S_T,K_PUT)×FC_AMT − PREM_PUT×FC_AMT`. This held consistently across every row of the lab's sensitivity grid, not just the baseline.
- The $2,408 gap equals `PREM_PUT × FC_AMT × (DF_USD − 1)` exactly — the time value of money on the premium between trade date and settlement. There is no error in either build; they encode two different, defensible conventions.

**Resolution: the workbook's convention is retained.** Time-valuing the premium is the more rigorous treasury practice — the premium is paid at trade date, so comparing it against settlement-date proceeds without adjusting for the time value of that cash is not apples-to-apples. The Stage 2 spec reasoned through this choice explicitly rather than defaulting to the simpler convention, and that reasoning still holds.

**This does not change the strategic conclusion.** At baseline, the put hedge underperforms the forward/money-market hedge under either convention ($4,494K or $4,497K vs. $4,605K); the lab's own "Best" column confirms Forward/MM wins at every S_T from -5% to +1%. The discrepancy affects precision, not the recommendation.

---

## 5. Rate-basis note (carried from Stage 2 §4, Open Item 3)

`ACT/360` applied to both `R_USD` and `R_FC` in this build, per the Stage 2 spec's single-basis simplification. Not revisited at Stage 4 — flagged as a known simplification, not resolved with live data, since Euribor and CMT quote conventions were not separately re-derived here.
