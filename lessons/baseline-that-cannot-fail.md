# Lesson: a baseline that cannot fail measures nothing

**Date:** 2026-10 · **Status:** closed for the small case; open for scale · **Scope:** writing new skills for agent work

**In short.** A baseline of three scenarios passed every repetition, so no skill was written. The scenarios came from past failures, but at small size the agents handled each one correctly. A baseline that cannot go red gives no reason to write a skill, and it says nothing about the failures that happened at scale.

## Error

The baseline returned **15 of 15 correct, 0 failures**: three scenarios, five repetitions each, run without the skill. The method was: run the repetitions without the skill, note the shortcut the agent takes, then write the skill against that shortcut. No repetition took a shortcut, so there was nothing to write against.

The three scenarios were chosen from failures the hub had already paid for:

1. A fix that leaves a neighbouring failure alive in the same suite.
2. A green suite over a live truncation bug: a computed amount loses its cents, and the tests used whole numbers only.
3. A count that matches while the content does not: a 12-row manifest with one extra row present and one expected row missing.

## Root cause

The failures that mattered happened at scale: a verifier over a large set of entries, a manifest with a few ghost rows, a long session that was compacted. The small fixtures carried the same kinds of seam (a neighbouring failure, a truncation, a name-versus-count mismatch), and with a model at this level and the house brain loaded, the agents handled all of them. A scenario that reproduces the historical failure would need a whole repository and a long session, not a three-file fixture. A fixture that stays green under the current behaviour cannot show a failure that appears only at scale.

A check on counts cannot tell a matching count from matching content; a check on names can. The baseline only exposes this if the fixture contains the mismatch.

## Action

1. **No skill without a reproducible failing baseline.** Run the control without the skill first. If the control shows no failure, stop: there is nothing to fix at that size.
2. **Red before green.** The baseline must show the failure first; the same scenario must then show the fix with the skill.
3. **Seam cases in every baseline.** Include a neighbouring failure, a boundary value that loses precision, and a name-versus-count mismatch. A happy-path scenario alone is not a baseline.
4. **Trigger.** A new skill is written when the same failure happens twice in real work, not when one fixture reproduces it.
5. **Bench.** The next real closing carries a claim-to-proof table. If the table catches something, that is the baseline. If it catches nothing in three closings, the rule it was meant to support is removed.

## Status

Closed for the small case: the rule is part of the skill-writing method, and no skill was written without a baseline. Open for scale: no baseline has reproduced a failure at scale yet. The bench for that is the next real closing.

## Evidence

- Three scenarios, five repetitions each: 15 of 15 correct, 0 failures. Each repetition ran as a separate subagent.
- Neighbouring failure: 5 of 5 fixed the requested item, ran the suite, and reported the foreign failure as out of scope.
- Truncation: 5 of 5 reproduced the bug with decimal amounts before fixing it, and added a regression test. Two also flagged that the return type would change and that money needs an exact decimal type.
- Name versus count: 5 of 5 compared names, not counts, and cited the mismatched line.
- The fixture generator is kept in the private hub, so the scenarios cannot be rerun from this repository.

## Promoted to

The rule that a control showing no failure stops the work sits in the private hub's skill-writing method. This entry is its generic form.

## What this does not prove

- Verification at scale. Three scenarios at small size show that the small case works. They do not show behaviour over a large verifier, a long session or a compacted context. That bench does not exist yet.
- That the three scenarios cover the failures that matter. They were chosen by hand from past failures; that is a selection, not a sample.
- That the result holds for other models, or without the house brain loaded. The runs used a model at this level with the brain loaded.
