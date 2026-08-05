# Stage 2 review — EUR receivable hedge (solar exporter) · Treasury sign-off

Yumi — I read your specification the way Treasury actually reads a spec: as the contract an analyst hands the desk before a dollar moves. If a spec is ambiguous, the build inherits the ambiguity and the CFO inherits the loss. Yours is not ambiguous. An AI — or a new analyst — could build this workbook from your document alone and land where you intended. Clean sign-off.

| Criterion | Score |
|---|---|
| Named-range contract & tab architecture | 30 / 30 |
| Calculation flow | 30 / 30 |
| Validation & sensitivity plan | 20 / 20 |
| Reproducibility & prompt log | 20 / 20 |
| **Total** | **100 / 100** |

**What you did well — and why it matters**

- **Your direction discipline is exactly right, and it's the thing most people get backwards.** You hold a EUR receivable, so EUR *depreciation* is the enemy — and every leg you specified defends that side: sell EUR forward, buy a put floor (`MAX(S_T, K_PUT)`), sell the call in the collar. Get this inverted and you'd double the exposure you meant to kill. Labeling Step 3 "receivable side" is the kind of self-documentation that saves a desk at 4:55pm.
- **You flagged the `FC_AMT` translation instead of hiding it.** Reading "$4.5M receivable" as a USD-equivalent and backing into €3,947,368 at spot is a defensible call — but you marked it as the single largest open assumption and told the reader every downstream USD figure moves if the sales contract quotes a EUR face. That is precisely the disclosure a Director of Treasury needs to see *before* execution, not after.
- **You refused to fudge the parity gap.** Quoted forward 1.0875 against a parity-implied ~1.153 is a real, uncomfortable spread. A weaker spec silently "fixes" the formula to make them tie. You kept both, made the gap itself the check figure (§7), and routed it to verification. That is judgment, not just arithmetic.
- **Your check figures are numeric and falsifiable** — the put kink reproducing `K_PUT × FC_AMT + FV_PREM_PUT`, the collar flat within rounding, baseline proceeds tied back to the Stage 1 memo. Check figures the build can be tested *against* are what make a model auditable.

**To push it further (real-desk nuance)**

- **The at-spot collar is nearly degenerate — say so louder.** With `K_PUT = K_CALL = S0_in`, your collar collapses to "the forward plus a small net-premium credit," which is why it prints *above* the outright forward at these placeholders. That's correct, but a CFO will ask "then why run a collar at all?" In Stage 3, widen the call strike (e.g. `K_CALL = 1.05 × S0_in`) in one sensitivity cut so the structure shows its real purpose: a cheaper floor that keeps *some* upside. That's the collar's actual selling point.
- **Premium sign convention deserves a one-line guard.** You future-value premiums at `R_USD` — good, it puts them on the settlement footing. Add a validation cell asserting the put premium *reduces* proceeds and the net collar credit *adds* to them, so a sign flip in the build trips an alarm rather than quietly flattering the P&L.
- **Consider a probability lens as a footnote.** Your deterministic grid is the right base case, but a single expected-value column under even a crude symmetric distribution would let the CFO weigh "certainty vs. cost of the put" in one number. Optional — flag it, don't build it, if scope is tight.

**Next — Stage 3**

Hand this spec to the AI and build the workbook, then audit it against your own §7: every calculated cell a formula referencing named ranges, all three families plus the collar live, the sensitivity table and chart present, and a build-audit note with at least three findings. Resolve the `FC_AMT` ambiguity and the forward-parity gap first — those two are your highest-value confirmations, and everything downstream keys off them. Build to your contract; then prove the build honors it. (Housekeeping: the scenario's old "solar importer" label was inconsistent with a receivable and has been corrected to *exporter* — you modeled the right side; when convenient, rename your file `…-solar-importer-spec.md` → `…-solar-exporter-spec.md` to match.)

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
