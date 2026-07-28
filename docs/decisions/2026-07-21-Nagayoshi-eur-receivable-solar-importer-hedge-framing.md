---
title: FX Receivable Exposure - Hedge Framing
author: Yumi Nagayoshi
date: 2026-07-28
version: 1.1
---

## Executive Summary 
The importer holds a $4.5M USD-equivalent EUR receivable due in one year, exposed to EUR depreciation. We compared unhedged, outright forward, protective put, and collar strategies using July 21, 2026 market data (spot EURUSD 1.14; USD 1Y rate 4.03%; EUR 1Y rate 2.88%) against the deal's stated forward (1.0875) and option premiums (put $0.015, call $0.018). The put and collar both beat the outright forward on a floor/locked basis, because the deal's quoted forward sits well below what today's rate differential implies. Recommend the put or collar over the forward, and verify the forward quote before execution.

---

## Background & Objectives
The company will collect a EUR-denominated receivable worth $4.5M today, in one year. Objective: protect USD proceeds from EUR depreciation while retaining reasonable upside, at acceptable cost.

---

## Methods
EUR notional = $4.5M ÷ 1.14 = €3.95M. Modeled USD proceeds if year-end spot is 1.05 / 1.14 / 1.25:

Forward contract: Locks a fixed EURUSD rate today for delivery in one year. Pro: no upfront cost, proceeds fully known. Con: forfeits all upside if EUR strengthens.

Money market hedge: Borrow EUR against the receivable, convert to USD now, invest until maturity. Pro: priced off our own funding/investment rates, independent of dealer forward pricing. Con: draws on balance-sheet/credit capacity and adds three transactions instead of one.

Options (EUR put or collar): A put sets a floor while keeping upside; a collar (put plus a sold call) narrows the range at little or no net cost. Pro: keeps upside participation, unlike a forward. Con: a standalone put costs a real premium; a collar caps the upside it's protecting.

---

## Limitations & Next Steps
Assumes the receivable is spot-translated USD-equivalent (confirm actual EUR notional). Ignores time value of premiums, transaction costs, and credit risk. The quoted forward rate appears stale relative to current rates. Next steps: pull live forward and option quotes, confirm the EUR notional, choose between the put ($59K premium, uncapped upside) and the collar (net $12K credit, capped at $4.51M), and route the selection through Treasury for hedge-accounting sign-off.

Beyond that, the build proceeds in stages for CFO approval: Stage 2 — write a full model specification for each hedge, with formulas and required inputs. Stage 3 — build the model with AI assistance and audit every output by hand. Stage 4 — replace placeholder rates with live market data at execution. Stage 5 — validate results and deliver a single recommended hedge with sizing.

---

## References
Spot/rates as of July 20–21, 2026: Yahoo Finance, Investing.com, TradingView (spot); U.S. Treasury 1Y CMT via ycharts (USD rate); Euribor-rates.info 12-month Euribor (EUR rate). Deal terms: `scenarios.md`, Scenario 1 — U.S. Solar Equipment Importer.
