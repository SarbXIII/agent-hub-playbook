# Lesson: a folder swap has a window, and that window is where the work is lost

**Date:** 2026-10 · **Status:** closed · **Severity:** medium

## What happened

The hub updates installed packs by swapping folders: download the new copy into `<folder>.new`, move `<folder>` to `<folder>.old`, then rename `<folder>.new` to `<folder>`. Two renames, milliseconds apart, and the documentation said «atomic replace».

It is not atomic. The service restarted between the two renames — the process died mid-swap and took the session with it. When the agent came back, the pack no longer existed under its real name. What remained was `<folder>.new` (the full new copy) and `<folder>.old` (the previous one), and the tool listing installed procedures did not show the pack at all.

Nothing had been corrupted. Everything the user needed was on disk, under a name nothing was looking for.

The same update path had a second, quieter problem: deleting a folder and copying into its place while the application was running left the procedure registered but unavailable for about a minute, because the loader had cached the old path.

## Root cause

Three properties were assumed and none had been checked:

1. **Two renames are one operation.** They are two, and the machine can stop between them. The safe sequence has to tolerate being interrupted *at every step*, not just at the ends.
2. **A backup is the same as a rollback.** `<folder>.old` preserved the data and gave nobody a way back: no step looked for it, so the recovery depended on a human remembering what the script does.
3. **Integrity was verified after success only.** The updater checked that the new copy had its entry file before swapping — good — but nothing checked the tree after an abnormal exit.

The real cost was not the lost minute; it was the moment of doubt. The agent saw a missing pack and had to reconstruct the swap sequence from the script source, in the middle of a session whose context had just been cut.

## Fix applied in the machine

- **Every update ends with an integrity check that runs out of band**, not only inside the updater: count folders, assert each one has its entry file, and fail if any leftover `.new`/`.old`/`.old-<suffix>` exists. A `CHECK: PASS/FAIL` line, callable on its own.
- **Recovery is a separate, boring script**: if the real name is missing and the new copy has its entry file, rename it back into place; if the real name exists and the old copy is still around, delete the leftover. The recovery does not try to be clever — it restores, it does not merge.
- **Leftovers are named for what they are**, and the recovery accepts the variants a human actually types (`.new`, `.old`, `.old-1`, `.old-copy`). The first version only accepted two spellings and refused to clean the third, which is how a leftover survived a "clean" run.
- **The swap copies to a sibling and renames in the same filesystem**, so the final step is a metadata operation, not a transfer.
- **A post-crash checklist** exists and is run before anything else: integrity, recovery, cleanup, then resume the task.

## Evidence

- One pack lost its real name in a real service restart. The new copy (6 files) and the old one were both intact; only the name was missing. Recovery took one rename; the copy was byte-identical to the downloaded archive.
- A second leftover survived the first cleanup because the cleaner only matched two suffixes. Fixed, re-run: 135 folders, zero without entry file, zero leftovers, `CHECK: PASS`.
- The manifest cross-check reported 135/135 folders with an owner after recovery, and the missing pack was listed and usable again in the same session.

## Transferable rule

> Any multi-step filesystem swap is a window, and a restart can land in it. Make every step recoverable from what is on disk *after* the crash, ship the recovery beside the updater, and verify the tree — not the code path — before telling anyone it worked.
