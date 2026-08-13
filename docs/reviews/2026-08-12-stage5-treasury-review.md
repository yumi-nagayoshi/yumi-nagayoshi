# Stage 5 review — solar importer LLM analysis & validation · Treasury sign-off

Yumi — this is the sharpest piece of LLM auditing in the cohort, and the reason is a distinction almost nobody drew:

> "There's a second issue here that I don't think is a spec gap at all — I think it's just ChatGPT being wrong. It claimed a specific number ('$9, attributable to rounding') to explain a gap that its own table shows as zero, and that number doesn't match my workbook's actual $103 gap either. Nothing in my documents caused that."

Fourteen of fifteen numbers matched, so the easy move was to write "one rounding discrepancy, immaterial" and move on. Instead you split the single mismatch into two different failures with two different owners — *my spec never stated a precision rule* (yours to fix) and *the model asserted a reconciliation it never performed* (not yours to fix). Refusing to absorb the model's error into your own retrospective is a judgement call, and it is the right one.

| Criterion | Score |
|---|---|
| LLM execution & comparison | 25 / 25 |
| Hand verification | 25 / 25 |
| Recommendation & executive voice | 25 / 25 |
| Spec retrospective | 17 / 17 |
| Repo polish | 6.4 / 8 |
| **Total** | **99 / 100** |

**What you did well — and why it matters**

- **Your diagnosis of the LLM's forward figure is correct, and I checked it.** You claimed its USD 4,604,502 comes from `FC_AMT × 1.166474` — the full-precision CIP forward — rather than your memo's stated 1.1665. Recomputing: 3,947,368 × 1.166474 = 4,604,501.63. That is its number exactly. You did not just notice a mismatch, you reverse-engineered which input produced it, which is what turns a discrepancy into a finding.
- **Every hand-verified figure reconciles.** Forward USD 4,604,604.77; money-market USD 4,604,501.63 through the three-step walk (`DF_FC` 1.029707, borrow EUR 3,833,487, convert at 1.1542, invest at `DF_USD` 1.040657); put floor USD 4,494,434.31 with the premium future-valued to USD 61,617.84. All to the cent, all independently recomputed here.
- **Both of your crossover rates are right.** The put stops beating unhedged at 1.13859 — that is exactly `USD_PUT / FC_AMT`, as it must be while the floor is engaged — and overtakes the forward at 1.18211, or +2.42%. Your memo says 1.1386 and +2.4%. Correct on both, and correct for the right reasons.
- **"Certainty is bought with the upside, not with cash."** That is the forward's entire economics in ten words, and it is the sentence a CFO will still have in mind a week later. Genuinely good executive writing.
- **You killed your own collar.** You built it, tested it across all eleven scenarios, found it never beat the plain forward, and recommended against a structure you had already done the work on. Sunk-cost discipline is rarer than it should be.
- **You reported that a fresh LLM beat the course's own tool.** The FX Hedging Lab deducts the premium at face value; your spec requires it future-valued; the model read your spec and got it right where the Lab did not. Drawing the conclusion — "the logic lives in my spec, not in my head or in the workbook file" — is precisely the point of the exercise.

**The one thing to fix — the forward premium is not a discount on certainty**

Finding 1 in §B, echoed in §D:

> "Because U.S. rates (4.01%) exceed EUR rates (2.93%), covered interest parity puts the one-year forward at 1.1665 — above the 1.1542 spot. For a receivable, that's favorable: it means locking in today costs us nothing relative to spot and in fact improves on it slightly, before any market move."

The first sentence is correct. The second reads a free lunch out of it.

The forward sits above spot for exactly the reason you gave — the interest differential — and that premium is the arbitrage-free compensation for it, not a gain the hedge delivers. Selling EUR forward at 1.1665 rather than spot at 1.1542 is what you get for waiting a year to be paid in a currency whose rates are 108bp lower; you have not been handed 1.23 cents, you have been quoted the price of time. Under CIP that premium is *always* there whenever `R_USD > R_FC`, in every market, on every day. It therefore carries no information about whether hedging is attractive now versus last month.

Test it by flipping the sign. If EUR rates exceeded USD rates, the forward would sit *below* spot, and the same sentence would read "locking in today costs us relative to spot" — arguing against a hedge whose economics are identical. The exposure has not changed, the reason to hedge has not changed, only the rate differential has. A criterion that reverses when nothing about the risk reverses is not a criterion.

This matters practically because "the forward is above spot, so hedging is cheap right now" is a real way desks talk themselves into sizing a hedge on carry instead of on exposure — which is a position, not a hedge. Your §E gets it right ("certainty is bought with the upside"); §B and §D briefly argue the other way. Cut the "improves on it slightly" framing or relabel it for what it is: the forward premium reflects the rate differential and is not evidence about the merits of hedging.

Worth noting your collar conclusion survives this cleanly — it is dominated because its net premium credit (about USD 12,300) is far smaller than the forward's USD 48,553 advantage over spot, which is a comparison between two hedges rather than a claim about cheapness. That argument is fine as written.

**Repo polish — 1.6 points**

Only gap is a missing `LICENSE`. Add one (MIT is fine for coursework) and you are at 100.

**One last thing**

Your v2 spec line — *"Downstream formulas must use the stated, rounded value — not a higher-precision figure computed on the way to it"* — is a better rule than the one most published model specs carry. Put it in the spec rather than leaving it in the retrospective, and this repo becomes something you can hand a reviewer without commentary.

— Treasury

---

### How to work this review — professional workflow

Treat this PR the way an analyst treats feedback from Treasury — a review is a proposal to engage with, not a checklist to rubber-stamp:

1. **Read it yourself first.** Understand each point and form your own view before changing anything. Disagreeing *with a documented reason* is a legitimate, senior response.
2. **Stress-test it with an LLM (pushback pass).** Paste this review and your spec into your AI assistant and ask it to (a) explain anything you're unsure of more deeply, and (b) argue the *other side* — where might the reviewer be wrong, and what would you give up by making each change. You're building judgment, not just executing edits.
3. **Decide, then draft the changes with the LLM.** For the points you accept, have the AI help implement them — you specify exactly what and why. Your spec is the prompt; precise in, correct out.
4. **Verify — non-negotiable.** Re-run your own checks (`scripts/recalc.py`, the parity tie-out, sensitivity continuity, no error cells) and confirm the numbers before you commit. An AI will hand you a confident wrong edit; verification is what makes the result *yours*.
5. **Close the loop on the PR.** Reply in the thread with what you changed, what you pushed back on and why, then commit and push. Writing down the reasoning is exactly how this works on a real team.

*This is the same human-in-the-loop discipline the whole project is built on: the LLM drafts, you edit and verify, and you own the result.*
