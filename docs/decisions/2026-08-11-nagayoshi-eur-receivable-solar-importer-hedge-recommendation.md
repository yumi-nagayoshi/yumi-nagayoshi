# Hedge Recommendation — EUR Receivable, Solar Equipment Sale

**To:** Chief Financial Officer
**From:** Treasury Analysis, Yumi Nagayoshi
**Date:** 2026-08-11
**Re:** Recommended hedge for the €3,947,368 receivable settling in one year

---

## A. Exposure Summary

We are owed €3,947,368 in one year, arising from a solar equipment sale currently booked at a $4,500,000 USD-equivalent value. Left unhedged, what we actually collect in USD depends entirely on where EUR/USD settles a year from now — and live market pricing puts that outcome anywhere between **$4.33M and $4.78M**, a swing of **$455,605**, before we've made any hedging decision at all. This memo compares our options and recommends one.

## B. Hedge Outcomes

Using live market data retrieved 2026-08-07 (spot 1.1542, USD 1yr rate 4.01%, EUR 1yr rate 2.93%), four strategies were modeled:

| Strategy | Outcome | Character |
|---|---|---|
| **Unhedged** | $4.33M – $4.78M | Full exposure both directions |
| **Forward / Money-Market** | **$4,604,605, fixed** | Certainty — same number every time |
| **Protective Put** (K=spot) | $4.49M floor, participates above breakeven | Floor + upside, at a cost |
| **Collar** (put bought + call sold, both at spot) | $4,568,376, fixed | Certainty, but *worse* certainty than the forward |

Three findings drive the recommendation:

1. **The forward locks in a rate above today's spot.** Because U.S. rates (4.01%) exceed EUR rates (2.93%), covered interest parity puts the one-year forward at 1.1665 — above the 1.1542 spot. For a receivable, that's favorable: it means locking in today costs us nothing relative to spot and in fact improves on it slightly, before any market move.
2. **The put's floor rarely pays for itself.** The premium (1.5¢/EUR, future-valued to $61,618) means the put only beats the forward when EUR falls *and* only beats staying unhedged when EUR falls further still — specifically below **1.1386** (roughly **-1.35%** from spot). Between roughly -1.35% and +2.4%, the put is the worst of the hedged choices: you're paying for protection you're not using.
3. **The collar is structurally dominated in this build.** Because both strikes are set at spot rather than at the forward rate, the collar effectively locks in *today's* rate, not the more favorable forward rate. Across all eleven scenarios tested, the collar never outperforms the plain forward — the "protection" it offers is protection from an outcome (spot) that's already worse than what the forward guarantees outright. It is not recommended as constructed.

## C. Sensitivity Interpretation

The ranking of strategies changes with the direction and size of the EUR move:

- **EUR depreciates 2% or more:** Forward/MM is best, Put is second, Unhedged is worst. This is the scenario the hedge exists for.
- **EUR moves within ±1% of spot:** Forward/MM still wins, but Unhedged edges out the Put — the put's premium cost outweighs a floor that barely engages.
- **EUR appreciates 2.4% or more:** Unhedged overtakes the Put, and the Put overtakes the Forward. In a strong rally, giving up the forward's certainty starts to cost real money — **$179,250** forgone at a +5% move, relative to staying unhedged.

The trade a locked hedge makes is explicit: it converts up to **$276,355** of downside protection (the forward's advantage over unhedged in a -5% move) into a hard ceiling on the upside. There is no version of a forward or money-market hedge that avoids this trade — certainty is bought with the upside, not with cash.

## D. Recommendation

**Hedge the full €3,947,368 receivable with a forward contract**, locking in $4,604,605.

This is not solely a "safest choice" recommendation — the numbers favor it structurally. The forward rate (1.1665) sits above current spot because of the interest-rate differential, so we are not sacrificing a favorable spot rate to buy certainty; we are locking in a rate that is already better than today's spot, with zero premium outlay. The put and collar were also evaluated and are not recommended: the put's floor only pays off in a depreciation scenario larger than -1.35%, and the collar is dominated by the plain forward at every point in the range tested.

The money-market hedge (borrow EUR, convert now, invest USD) produces an economically near-identical outcome ($4,604,502, a $103 gap attributable to rounding — see Appendix note) and is offered as an operational alternative if there's a reason to prefer an on-balance-sheet structure over an off-balance-sheet forward contract (e.g., existing EUR credit lines, counterparty limit considerations).

## E. Executive Justification

- **Cash-flow certainty:** A single, known USD figure for financial planning, eliminating a $455,605 swing that would otherwise be pure noise in next year's cash forecast.
- **Budget certainty:** Whatever budget or covenant calculations depend on this receivable can be built against $4,604,605 today, not a range.
- **No premium cost:** Unlike the put ($61,618 future-valued cost) or collar, the forward requires no upfront or ongoing cash outlay — the entire hedge is captured in the locked rate itself.
- **Liquidity:** The forward is off-balance-sheet until settlement; the economically equivalent money-market alternative requires immediate EUR borrowing and USD investment, using balance sheet capacity the forward does not. This is the deciding factor between the two if liquidity is tight in the interim.
- **Optionality forgone:** The one real cost of this recommendation is upside participation. If EUR rallies more than roughly 2.4%, we will have locked in less than an unhedged position would have delivered — up to $179,250 less at a +5% move. If management holds a strong, specific view that EUR is likely to strengthen materially, that view should be weighed against this memo's recommendation before execution; it would argue for the put instead, accepting its premium cost as the price of that participation.
- **Accounting note (optional):** A forward designated as a cash-flow hedge would generally qualify for hedge accounting treatment, with effective-portion gains/losses recognized in OCI rather than P&L until settlement — worth confirming with accounting before execution if that treatment is desired.

---

*Supporting detail — full sensitivity grid, hand-verified arithmetic, and comparison against an independently-run LLM analysis — is available in the companion validation document.*
