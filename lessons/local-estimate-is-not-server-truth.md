# Lesson: a local estimate is not server truth

**Date:** 2026-11 · **Status:** mitigated; the monthly figure is withheld without server rows, and the limit table is not recalibrated · **Scope:** usage estimators that combine a local limit table with a threshold

**In short.** A usage script printed a monthly share computed from a local limit table. The table predated a plan change, and the printed figure was about 7.5× the provider console's figure for the same month. A generic "lower bound" caveat did not cover an error of that size, and a threshold in the watchdog read the same local figure. The fix withholds the monthly figure unless server rows exist for the cycle. The table was not recalibrated: on the same day, the monthly and weekly windows needed different factors against the console, so one correction cannot fit both.

## Error

- The script printed a monthly share. Minutes later it printed a higher one, with an exhaustion estimate a few days away.
- The provider console showed a much lower figure for the same month. The script's figure was about 7.5× the console's.
- A usage threshold in the watchdog was wired to the same local figure.

## Root cause

- The script converts usage into a share of a monthly limit using a limit table. The table predated a plan change, so its denominator was wrong.
- Without server rows for the cycle, nothing corrects the denominator, and the script still printed a number.
- The number was published under a monthly label with a generic lower-bound note. The note does not cover an error of about 7.5×.
- A threshold wired to an uncalibrated local figure raises alarms that the server data would not raise.
- The same mechanism had a second route. A measured reproduction of an edge case, a valid server file inside the cycle with no rows in the last seven days, showed the script computing a monthly equivalent and a days-to-full figure from the local table and labelling the source as server. The label followed the file's validity, not the rows that fed the figure.

## Action

1. **Implemented: monthly figure only from server rows.** A monthly share is published only when server rows for the cycle exist. Without a server CSV, the monthly figure is null, the source field is set to a "not verifiable" value that names the missing CSV, and days-to-full is null with a note.
2. **Implemented: labels follow the rows.** A local table value never carries a server origin; the server percentage has its own field. The five-hour window stays a local estimate by design, because the server CSV is daily, and is labelled as local.
3. **Implemented: windows without server rows.** With a valid server CSV inside the cycle but no server rows in the seven-day window, the seven-day and thirty-day windows show risk '?' (unknown), and the monthly equivalent, the percentage-per-day rate and days-to-full are withheld. The local daily spend stays as a measured local figure. Without any CSV, the five-hour and seven-day windows keep a local-table risk label, marked as local, and the monthly figure stays null.
4. **Implemented: explicit cycle start.** The cycle start is an explicit input. A malformed or future start date is rejected with exit code 1.
5. **Implemented: thresholds need a server anchor.** The threshold cannot fire on the local estimate. Its value is unchanged; only its source changed.
6. **Future: recalibration.** The limit table is not recalibrated. Any future update must be per window, dated, and checked against the console. Until then the table is marked as not verified against the console.

## Status

- Mitigated: the monthly figure is withheld without server rows; the edge case that labelled local figures as server figures is fixed; local windows carry a local label; the threshold reads only server-backed figures.
- Open: the limit table is not recalibrated. It goes stale at the next plan change, and the evidence shows no check that would notice that change.

## Evidence

- The script's output and a console reading were compared on the same day.
- Before the edge fix, the regression suite did not pass (90 of 95 checks). After it, the suite passes 106 of 106 checks.
- Edge case (valid server CSV inside the cycle, no rows in the seven-day window): no monthly equivalent, risk unknown, no days-to-full, no percentage-per-day rate from the local table, and a notice explains the missing rows.
- No CSV at all: the monthly figure is null, its source is marked not verifiable, and the text output shows neither a monthly figure nor a threshold result.
- The first estimate was about 7.5× the console figure for the same month. A later calculation, with consumption bounded to the cycle, gave different implied factors for the monthly and weekly windows on the same day, so no single correction fits both.

## What this does not prove

- That the withholding survives the next plan change. The rule stops a stale table from publishing a monthly figure; it does not make the table current.
- That the console's own figures are exact. The console is the comparison reference, not an audited measurement.
- That the 7.5× ratio is stable. It comes from one run on one day, and the later calculation gave different factors.
- That the rule holds for other plans or providers. The evidence covers one plan change and one comparison day.
- It is a snapshot, not a controlled study.
