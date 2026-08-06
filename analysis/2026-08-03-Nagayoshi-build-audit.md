# Audit Note — 2026-08-03 (revised)

I confirmed the accuracy of the numbers in my spreadsheet by comparing it with the class-provided spreadsheet's "Transaction Hedging (Receivable)" tab.

- **Forward vs. Money-Market gap:** My `USD_MM` ($4,550,980) does not match `USD_FWD` ($4,292,763). Gap is $258,217. This is expected, not an error — the quoted forward (1.0875) is well below the parity-implied rate (~1.153). Still an open item, need to verify the quoted forward before execution.
- **Day-count:** I kept ACT/360 on both legs, since that's what my spec says. The class sheet does not use day-count, which changes `USD_MM` by about $678.81. I'm noting this difference, not matching it.
- **FC-AMT currency symbol:** Was showing `$`. Corrected to `€` since the unit is euros.
- **Options tab:** My numbers are different by design, not by mistake. My strike is at spot — it protects me if the rate drops, and I just take the market rate if it goes up instead. The class example sets the strike far enough away that it never gets used in their scenarios, so they pay the premium but get no real protection.
- **Open item:** `FC_AMT` — confirmed. The $4,500,000 is today's USD-equivalent value of the receivable, converted to EUR at spot (1.14). So `FC_AMT` = 4,500,000 / 1.14 ≈ €3,947,368. Updated Cover, Inputs, and Notes tabs to mark this resolved instead of open.

## Spec validation checks (§7) — all pass

- `USD_MM`/`F_implied` are computed and shown, and the gap is documented above instead of quietly fixed.
- Kink check: `USD_PUT` at `S_T = K_PUT` = $4,438,370, which matches `K_PUT × FC_AMT + FV_PREM_PUT` exactly.
- `USD_COLLAR(S_T)` stays flat across the whole grid ($4,512,325 every row), since `K_PUT = K_CALL`.
- Baseline numbers match the Stage 1 memo: put ≈ $4,438K (after ~$61.6K premium), collar ≈ $4,512K (after ~$12.3K credit), collar cap ≈ $4.51M.
- No error cells anywhere on the Sensitivity tab.
- Every output cell is a formula using named ranges, nothing hardcoded.
- The grid is symmetric around `S0_in` and driven entirely by `STEP_PCT`.

## Still open

- Forward-vs-parity gap — verify the counterparty's quoted forward before execution.
- Rate basis is a single ACT/360 for both legs — revisit if the EUR-side convention turns out to be different at execution.
