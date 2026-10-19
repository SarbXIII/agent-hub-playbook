# Lesson log

> One entry per failure. Without a concrete action there is no lesson, only a lament.
> Format: **date · error · root cause · action · status · evidence**.

---

## {YYYY-MM-DD} · {Short failure title}

- **Error:** what happened, with the literal message or symptom.
- **Root cause:** the mechanism, not the symptom. If it is a pattern, say so.
- **Action:** the concrete change in the machine — guard, test, contract, deterministic check, metric. Not «be more careful».
- **Status:** open / mitigated / closed.
- **Evidence:** command output, session ref, commit, screenshot, measurement.
- **Promoted to:** role brain or project note, if generic.

---

## Checklist before adding an entry

- [ ] Is it a pattern or an isolated case?
- [ ] Does the action prevent the repetition, or only document it?
- [ ] Can someone verify it without asking me?
- [ ] Does an existing lesson already cover it? Then this is a **regression** — fix the machine.
