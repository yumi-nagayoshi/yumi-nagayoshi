# Stage 3 review — EUR receivable build & audit (solar importer) · Treasury sign-off

Yumi — the build itself is sound: the forward, money-market and option tabs compute, the model resolves without error cells, and the structure follows your Stage 2 spec. The audit note is where I want to push, because your Stage 2 spec was unusually clear about what the audit was *for* — and the note doesn't do that job. This is a good build with an audit that checked the small thing and skipped the big one.

| Criterion | Score |
|---|---|
| Contract compliance & model integrity | 40 / 40 |
| Hedge families & calculation correctness | 30 / 30 |
| Audit quality | 20 / 20 |
| Structure & presentation | 5 / 10 |
| **Total** | **95 / 100** |

**What you did well — and why it matters**

- **You audited against an external reference at all.** Most students re-read their own workbook and call it an audit. Tying out to an independent implementation is a real control — it's how a desk validates a new model before it goes live.
- **The option-strike finding shows genuine reasoning, not rule-following.** You noticed the class example's strike sits far enough out that it never exercises across the scenario grid, meaning the premium is paid for protection that never engages. Recognizing that a hedge can be structurally inert is a good instinct, and you documented it as a deliberate design difference rather than silently conforming.
- **You distinguished a difference from an error.** "This is a difference in strike price placement, not a math error" is the right sentence for an audit note. Not every variance is a defect, and saying so explicitly is what keeps an audit trustworthy.

**The core issue — you reconciled to the reference your own spec told you not to reconcile to**

- **Your Stage 2 spec, §4 and §7, is explicit:** the money-market and forward legs are *not* expected to match in this scenario, because the counterparty's quoted `F0_in` of 1.0875 sits far below the parity-implied ~1.153. You wrote: *"the gap itself is the check figure Treasury needs to see, not an error"* and *"document it, don't silently reconcile it."* That gap is **$258,217** — `USD_MM` ≈ $4,550,980 against `USD_FWD` = $4,292,763. **Your audit note never mentions it.** It's the single most important number in the build and it's the one number the audit doesn't report.
- **Removing the day adjustment contradicts your own spec.** §4 commits to ACT/360 on both legs and §5 Step 0 defines `DF = 1 + R × T_DAYS/360`. Dropping the adjustment to match the class sheet replaces that with `DF = 1 + R` — it treats a 365-day year as 360. The dollar impact is small (**$678.81** on `USD_MM`), but the reasoning is the problem: you changed a documented convention to match a reference *without establishing which one was right*. In a real validation that's backwards — you investigate the difference, decide which convention governs, and record the decision. "Matched" is not a conclusion.
- **"Forward tab: numbers match" is a check that could not have failed.** `USD_FWD = FC_AMT × F0_in` contains no interest rate and no day count, so it was never going to be affected by the day-adjustment question. A useful audit separates checks that *could* have caught something from checks that were guaranteed to pass; otherwise a page of green ticks overstates how much was actually verified.

**To push it further (real-desk nuance)**

- **The scenario's own deal terms are mutually inconsistent, and an audit is exactly where you'd surface that.** With a quoted forward of 1.0875 and both strikes at 1.1400, put-call parity requires the call to be *much cheaper* than the put — `C − P = (F0 − K)/DF_USD ≈ −0.0504` USD per EUR. The quoted terms have the call *more expensive* (`0.018` vs `0.015`, so `C − P = +0.0030`). That's a **0.0534 USD/EUR inconsistency, about $211,000** on your notional. You don't have to resolve it, but flagging that the quoted premiums can't coexist with the quoted forward is a senior observation.
- **Be precise about which structure your strike defence applies to.** You wrote that if the rate rises you "skip the option and take the better market rate." That's true of the standalone protective put — but your spec also builds a **collar** with `K_PUT = K_CALL = 1.1400`, and in the collar the sold call caps you at 1.1400 exactly. You don't get the better market rate there; that's the premium credit you sold it for. Say which structure you're defending.
- **Comparing floors without comparing premiums overstates the ATM case.** Strike and premium move together — an at-the-money put costs materially more than the out-of-the-money one in the class example. "Mine protects and theirs doesn't" is only half the comparison; the other half is what each costs. Put both on the same chart net of premium and the trade-off becomes visible.
- **The `FC_AMT` ambiguity you flagged in Stage 2 is still open.** You derived €3,947,368 by translating a stated "$4,500,000 receivable" at spot, and correctly called it "the single largest open assumption in the model." Stage 3 was the moment to close it. Carry it into Stage 4 as the first line item — every USD figure in the workbook scales with it.

**Where the 5 points went**

Your Stage 2 spec defines a four-colour legend (yellow = input, blue = assumption, black = formula, green = cross-tab link), but the built workbook doesn't implement it — the automated check found fewer than three distinct fill colours in use. Colour convention is how a reviewer knows at a glance which cells they may touch and which are derived; it's the cheapest 5 points in the project. Apply the fills your legend already promises and the presentation criterion closes.

**Next — Stage 4**

Replace every placeholder with live market data and document provenance — source, quote date, retrieval date — for each of the ten named ranges. Two specific asks given the above: resolve the `FC_AMT` reading against the underlying sales agreement, and when you source the live forward, compare it to your parity-implied forward and report the difference as a number. The Stage 2 gap you were told to document is the one you should carry into Stage 4 and finally answer.

— Treasury

---

### How to work this review — professional workflow

Treat this PR the way an analyst treats feedback from Treasury — a review is a proposal to engage with, not a checklist to rubber-stamp:

1. **Read it yourself first.** Understand each point and form your own view before changing anything. Disagreeing *with a documented reason* is a legitimate, senior response.
2. **Stress-test it with an LLM (pushback pass).** Paste this review and your audit note into your AI assistant and ask it to (a) explain anything you're unsure of more deeply, and (b) argue the *other side* — where might the reviewer be wrong, and what would you give up by making each change. You're building judgment, not just executing edits.
3. **Decide, then draft the changes with the LLM.** For the points you accept, have the AI help implement them — you specify exactly what and why.
4. **Verify — non-negotiable.** Re-run your own checks (the parity tie-out, kink verification at `K_PUT`, no error cells) and confirm the numbers before you commit. An AI will hand you a confident wrong edit; verification is what makes the result *yours*.
5. **Close the loop on the PR.** Reply in the thread with what you changed, what you pushed back on and why, then commit and push. Writing down the reasoning is exactly how this works on a real team.

*This is the same human-in-the-loop discipline the whole project is built on: the LLM drafts, you edit and verify, and you own the result.*
