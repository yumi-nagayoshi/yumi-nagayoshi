# Stage 4 review — solar importer market data & population · Treasury sign-off

Yumi — §3 is the best piece of analytical writing in the cohort at this stage, and it is about your own finding disappearing.

Stages 1–3 rested on a real result: the counterparty's quoted `F0_in` of 1.0875 sat well below the parity-implied forward, a ~$258K dislocation worth escalating. At Stage 4 you could not get a live counterparty forward, so `F0_in` became CIP-implied — *the same formula as the parity check itself*. And you wrote: "That signal does not survive Stage 4 population, and this is worth stating plainly rather than glossing over... The parity check on this live dataset proves the formula is internally consistent, not that a real-world mispricing exists."

Then the closing line: "the audit finding from Stage 3 doesn't disappear because it was wrong — it disappears because Stage 4's live `F0_in` is a computed proxy rather than a quoted deal term."

That is exactly right, and it is the hardest thing to write. Your headline finding evaporated for methodological reasons, and rather than quietly letting a $9 reconciliation stand in as validation, you explained why the check had lost its power. Most of the cohort reported a passing parity check at Stage 4 as if it confirmed something. It confirms almost nothing, and you are the one who said so.

| Criterion | Score |
|---|---|
| Data quality & provenance | 50 / 50 |
| Model resolves cleanly | 33 / 33 |
| Lab cross-check | 17 / 17 |
| **Total** | **100 / 100** |

**What else you did well**

- **You closed the `FC_AMT` open item instead of carrying it forward.** §2 resolves the "$4,500,000 receivable" ambiguity as a USD-equivalent translated at the notional-fixing spot (≈ €3,947,368), states the alternative reading, and says plainly: "This is the resolution, not a continued open item... If a real sales agreement existed, it would supersede this reading." Naming an assumption *and* naming what would override it is how you keep an open question from silently becoming a fact.
- **You cross-checked your spot against two independent feeds.** ECB fix of 06 Aug against TradingEconomics and Myfxbook intraday (~1.154–1.156), "consistent within normal daily range." That is a genuine data-quality control, not a citation.
- **You explained the ECB publication window.** "ECB publishes ~16:00 CET, so the 07 Aug fix had not yet posted at time of retrieval." A reader wondering why a 07 Aug memo uses a 06 Aug rate has the answer before they ask.
- **You justified CMT over SOFR by consistency with your own Stage 2 spec.** Not just "it's standard," but "consistent with the Stage 2 spec's original sourcing choice." Holding to a documented decision across stages is what makes a model auditable.
- **You listed the sources that failed.** "Barchart, investing.com, and Bluegamma all required a paid subscription or terminal login." That is what turns a proxy into a defensible fallback.
- **You separated the two gaps correctly.** The 0.0790 difference between the scenario forward and the live CIP forward is labelled "stale scenario terms vs. current market, not a live dislocation." Distinguishing a stale input from a market signal is the whole discipline.

**To push it further (real-desk nuance)**

- **Name the number that would restore the check.** You say a real Stage 4 execution "would require a live counterparty or terminal forward quote to reproduce a genuine parity test." Go one step further: the gap between a quoted forward and your 1.1665 *is* the cross-currency basis plus dealer spread, and for EURUSD at one year that is typically single-digit to low-double-digit basis points. Sizing what you are missing tells the reader how much the missing data is worth.
- **Your `R_FC` is 12-month Euribor — the right instrument for the money-market leg.** Worth stating why explicitly: the EUR leg of your hedge is a *borrowing*, and Euribor is a borrowing rate, whereas a Bund yield would understate the cost. You chose correctly; make the reasoning visible.
- **`K_PUT = K_CALL = S0_in` means your collar has no width.** Both struck at spot is a clean convention, but a floor and cap at the same rate is not really a collar — it is a synthetic forward, minus the net premium. Check whether `USD_COLLAR_BASE` ($4,568,376) behaves that way; if so, say it, because it is an elegant result.

**Next — Stage 5**

Hand the workbook and your Stage 2 spec to an LLM, get its analysis, then break it. Recompute at least three outputs by hand with explicit arithmetic. Your spec retrospective has an unusually strong story ready: a headline Stage 3 finding that could not be reproduced at Stage 4, and exactly why.

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
