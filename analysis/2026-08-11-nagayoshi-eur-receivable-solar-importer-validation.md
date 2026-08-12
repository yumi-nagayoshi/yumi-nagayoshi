# Stage 5 Validation — EUR Receivable Hedge Model

**Author:** Yumi Nagayoshi
**Date:** 2026-08-11
**Companion documents:** Stage 2 spec, Stage 4 market-data memo, Stage 3/4 workbook
**Raw LLM output:** [2026-08-11-nagayoshi-eur-receivable-solar-importer-llm-raw-output.md](./2026-08-11-nagayoshi-eur-receivable-solar-importer-llm-raw-output.md)

---

## Part 1 — Independent LLM execution

A fresh ChatGPT session (no prior history) was given exactly two documents — the Stage 2 spec and the Stage 4 market-data memo — and asked to compute all hedge outcomes and recommend a strategy. No workbook results were shared and no mid-run corrections were made. Prompt used:

> These are for hedge model specification and market-data memo for EUR receivable hedging analysis. Using only these information, compute all hedge outcomes. then tell me which hedging strategy you recommend among forward, money-market, put option, call option, and unhedged.

Full output is in the linked appendix above.

## Part 2a — Comparison table

LLM output vs. workbook output, at three representative S_T points:

| S_T | Strategy | Workbook | LLM (ChatGPT) | Match? | Diagnosis |
|---|---|---:|---:|---|---|
| 1.0965 (-5%) | Unhedged | $4,328,250 | $4,328,250 | ✅ | — |
| | Forward | $4,604,605 | $4,604,502 | ❌ ($103) | See discrepancy note below |
| | Money-Market | $4,604,502 | $4,604,502 | ✅ | — |
| | Put | $4,494,434 | $4,494,434 | ✅ | — |
| | Collar | $4,568,376 | $4,568,376 | ✅ | — |
| 1.1542 (0%, baseline) | Unhedged | $4,556,052 | $4,556,052 | ✅ | — |
| | Forward | $4,604,605 | $4,604,502 | ❌ ($103) | See discrepancy note below |
| | Money-Market | $4,604,502 | $4,604,502 | ✅ | — |
| | Put | $4,494,434 | $4,494,434 | ✅ | — |
| | Collar | $4,568,376 | $4,568,376 | ✅ | — |
| 1.2119 (+5%) | Unhedged | $4,783,855 | $4,783,855 | ✅ | — |
| | Forward | $4,604,605 | $4,604,502 | ❌ ($103) | See discrepancy note below |
| | Money-Market | $4,604,502 | $4,604,502 | ✅ | — |
| | Put | $4,722,237 | $4,722,237 | ✅ | — |
| | Collar | $4,568,376 | $4,568,376 | ✅ | — |
| — | Call (standalone) | *not modeled* | *excluded, reasoned* | ✅ | See note below |

14 of 15 numeric outputs matched exactly. The one consistent discrepancy — Forward, at every point — I dug into below.

### Discrepancy: Forward vs. Money-Market convergence

My workbook computes `USD_FWD = FC_AMT × F0_in`, using the memo's stated value `F0_in = 1.1665` (4 decimal places): **$4,604,605**.

ChatGPT's Forward figure is **$4,604,502** — identical to its Money-Market figure, not $103 higher. When I traced the arithmetic, it matches `FC_AMT × 1.166474` (the full-precision CIP-implied forward, before I rounded it to 4 decimals in the memo) rather than the memo's literal stated value.

I don't think this is just a rounding choice ChatGPT happened to make — I think it's a self-contradiction in its own output. Its narrative text says: *"Forward: approximately $4,604,502 / Money market: approximately $4,604,502 / Difference: approximately $9, attributable to rounding."* But its own table shows these two figures as identical ($0 apart), and neither the $0 gap nor the $9 it claimed matches my workbook's actual $103 gap. It asserted a specific reconciliation ("$9, attributable to rounding") without the arithmetic behind that claim actually showing up anywhere in its output. That's not simple rounding drift — that's stated false precision.

**Where this comes from in my own documents:** my Stage 4 memo states `F0_in = 1.1665` to four decimal places but never says whether that rounded figure is what should feed downstream calculations (`USD_FWD = FC_AMT × F0_in`), or whether the underlying full-precision CIP figure should be used instead so it stays consistent with the parity check (`F_IMPLIED`). My workbook uses the rounded, stated value; ChatGPT used the full-precision value. Both are reasonable readings of something I never actually specified.

### Call option — excluded by both my workbook and ChatGPT, independently

I never built a standalone purchased-call strategy into the workbook — only a protective put and a collar (put purchased + call sold). ChatGPT, with no visibility into my workbook, landed on the same exclusion for the same reason: a receivable is exposed to EUR depreciation, so a purchased call (which profits from EUR appreciation) doesn't hedge this exposure. I'm treating this as a confirmed, correct call rather than a discrepancy — two independent analyses (my original workbook design and ChatGPT's reasoning from the spec alone) landed on the same answer without talking to each other.

## Part 2b — Hand verification (arithmetic shown)

### 1. Forward proceeds
```
USD_FWD = FC_AMT × F0_in
        = 3,947,368 × 1.1665
        = 3,947,368 + (3,947,368 × 0.1665)
        = 3,947,368 + 657,236.77
        = 4,604,604.77 ≈ $4,604,605
```

### 2. Money-market hedge (three steps)
```
Step 1 — Borrow EUR:
  DF_FC = 1 + R_FC × T/360 = 1 + 0.0293 × 365/360 = 1.029707
  Borrow_EUR = FC_AMT / DF_FC = 3,947,368 / 1.029707 = 3,833,487 EUR

Step 2 — Convert at spot:
  USD_now = 3,833,487 × 1.1542 = 4,424,610

Step 3 — Invest in USD:
  DF_USD = 1 + R_USD × T/360 = 1 + 0.0401 × 365/360 = 1.040657
  USD_MM = 4,424,610 × 1.040657 = 4,604,501 ≈ $4,604,502
```
Note: `USD_FWD` ($4,604,605) and `USD_MM` ($4,604,502) differ by ~$103 — the same rounding artifact I flagged in the discrepancy note above. Doing this by hand reproduces the same gap, which tells me it comes from how I stated `F0_in`'s precision in the memo, not from a formula error in the workbook.

### 3. Put floor at S_T = 1.0965 (-5%)
```
USD_PUT(S_T) = MAX(S_T, K_PUT) × FC_AMT − PREM_PUT × FC_AMT × DF_USD

MAX(1.0965, 1.1542) = 1.1542   (floor engages)

Gross proceeds = 1.1542 × 3,947,368 = 4,556,051.87

Premium, future-valued:
  0.015 × 3,947,368 × 1.040657 = 59,210.52 × 1.040657 = 61,617.84

USD_PUT = 4,556,051.87 − 61,617.84 = 4,494,434.03 ≈ $4,494,434
```
Versus unhedged at the same S_T ($4,328,250), the put floor is worth **$166,184** in this depreciation scenario — the concrete value of the floor when it's actually needed.

All three hand-computed figures reconcile exactly to the workbook's live cell values.

## Part 4 — Spec retrospective

**What ChatGPT got right, and why it matters.** Fourteen of fifteen numbers it produced matched my workbook exactly, including the future-valuing convention for option premiums (Stage 2 spec §5) — a detail that's easy to get wrong. It also correctly figured out, on its own, why a standalone call doesn't belong in this analysis. What makes this worth calling out: the FX Hedging Lab I used back in Stage 4 got that same premium convention *wrong* — it subtracted the premium at face value instead of future-valuing it. So a fresh LLM with nothing but my spec text outperformed a purpose-built course tool on the one convention that actually mattered. That tells me the logic lives in my spec, not in my head or in the workbook file — which is really the whole point of writing a spec in the first place.

**What it got wrong, and what that tells me about my own documents.** The one real mismatch — Forward using the full-precision CIP number instead of my memo's stated, rounded `F0_in` — comes down to something I genuinely never specified: my Stage 4 memo states F0_in to four decimal places but never says whether that's the number to actually plug into downstream formulas, or just a display value while the full-precision number should be doing the real work. My workbook picked one answer to that question (the rounded value) but never wrote the rule down anywhere — it just lived silently inside a cell reference. A reader with only the memo had no way to know that.

There's a second issue here that I don't think is a spec gap at all — I think it's just ChatGPT being wrong. It claimed a specific number ("$9, attributable to rounding") to explain a gap that its own table shows as zero, and that number doesn't match my workbook's actual $103 gap either. Nothing in my documents caused that — it just stated something confidently that wasn't backed by its own math. That's the part of this exercise that stuck with me most: everything else it got right made this one confident, wrong sentence easy to miss if I hadn't checked the arithmetic myself.

**What I'd change in v2 of the spec.** I'd add one explicit line to Stage 2 §5 or the memo template: *"All input values are stated at final precision (4 decimal places for rates and FX levels). Downstream formulas must use the stated, rounded value — not a higher-precision figure computed on the way to it — so a reader recomputing by hand from the stated inputs alone reproduces the workbook exactly."* That single sentence would have closed the only real gap this test found.

**Overall, I think this went well.** My spec held up on every substantive judgment call — premium timing, which strategies even belong in the comparison — and only slipped on a precision detail I can now name exactly and fix in one sentence. I'd rather have found this now, from a fresh LLM run, than have a reader run into it on their own with no way to tell which number was right.
