[2026-07-21-nagayoshi-eur-receivable-solar-importer-hedge-framing.md](https://github.com/user-attachments/files/30253503/2026-07-21-nagayoshi-eur-receivable-solar-importer-hedge-framing.md)
# EUR Receivable Hedge Framing — U.S. Solar Equipment Importer

**Created by:** Yumi Nagayoshi  

**Updated by:** Yumi Nagayoshi 

**Date Created:** 2026-07-21

**Date Updated:** 2026-07-21

**Version:** 1.0

**LLM Used:** Claude (Anthropic)

---

## Executive Summary
The importer holds a $4.5M USD-equivalent EUR receivable due in one year, exposed to EUR depreciation. We compared unhedged, outright forward, protective put, and collar strategies using July 21, 2026 market data (spot EURUSD 1.14; USD 1Y rate 4.03%; EUR 1Y rate 2.88%) against the deal's stated forward (1.0875) and option premiums (put $0.015, call $0.018). The put and collar both beat the outright forward on a floor/locked basis, because the deal's quoted forward sits well below what today's rate differential implies. Recommend the put or collar over the forward, and verify the forward quote before execution.

---

## Background & Objectives
The company will collect a EUR-denominated receivable worth $4.5M today, in one year. Objective: protect USD proceeds from EUR depreciation while retaining reasonable upside, at acceptable cost.

---

## Methods
EUR notional = $4.5M ÷ 1.14 = €3.95M. Modeled USD proceeds if year-end spot is 1.05 / 1.14 / 1.25:

| Strategy | 1.05 | 1.14 | 1.25 |
|---|---|---|---|
| Unhedged | $4.14M | $4.50M | $4.93M |
| Forward (1.0875) | $4.29M | $4.29M | $4.29M |
| Put (K=1.14) | $4.44M | $4.44M | $4.88M |
| Collar (K=1.14) | $4.51M | $4.51M | $4.51M |

The put/collar's implied synthetic forward (~1.143) tracks the covered-interest-parity forward (~1.153) far more closely than the deal's stated 1.0875 — worth flagging.

---

## Limitations & Next Steps
Assumes the receivable is spot-translated USD-equivalent (confirm actual EUR notional). Ignores time value of premiums, transaction costs, and credit risk. The quoted forward rate appears stale relative to current rates. Next steps: pull live forward and option quotes, confirm the EUR notional, choose between the put ($59K premium, uncapped upside) and the collar (net $12K credit, capped at $4.51M), and route the selection through Treasury for hedge-accounting sign-off.

---

## References
Spot/rates as of July 20–21, 2026: Yahoo Finance, Investing.com, TradingView (spot); U.S. Treasury 1Y CMT via ycharts (USD rate); Euribor-rates.info 12-month Euribor (EUR rate). Deal terms: `scenarios.md`, Scenario 1 — U.S. Solar Equipment Importer.
