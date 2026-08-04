# Audit Note — 2026-08-03

**Author:** Nagayoshi
**Purpose:** Confirm the accuracy of the numbers in my spreadsheet by comparing them against the class-provided spreadsheet's "Transaction Hedging (Receivable)" tab.

---

## Inputs Tab

- **FC-AMT display currency:** The FC-AMT value was displayed with a `$` sign. Since its unit is euros, it should be displayed with a `€` sign instead. **Corrected.**
- **Day adjustment:** The class-provided spreadsheet did not include a day adjustment, but mine did. **Removed** the day adjustment to match.

## Forward Tab

- Numbers match the class-provided spreadsheet.

## Money Market Tab

- After removing the day adjustment (per the Inputs tab fix above), numbers match.

## Options Tab

- My option hedge numbers differ from the class example. This is a difference in **strike price placement**, not a math error.

**Explanation of the difference:**

- My strike is set right at today's rate. If the rate drops, the hedge protects me; if it rises, I skip the option and take the better market rate.
- The class example sets the strike further away, so it never actually gets exercised in their examples — they end up paying the premium either way, with no protection kicking in.

---

## Summary of Changes Made

| Item | Status |
|---|---|
| FC-AMT currency symbol ($ → €) | Corrected |
| Day adjustment (Inputs/Money Market) | Removed |
| Forward tab | No change needed — matched |
| Options tab strike placement | No change — intentional design difference, documented |
