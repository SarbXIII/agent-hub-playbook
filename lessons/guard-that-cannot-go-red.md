# Lesson: a guard that cannot go red is decoration

**Date:** 2026-10 · **Status:** mitigated; targets are parameters and harnesses exist · **Scope:** consistency guards over agent configuration and project folders

**In short.** Two guards read their own installation. Against the live tree both reported healthy, and no one had seen either one fail. A green result from such a guard can mean three different things: it reads the wrong place, the field it watches is gone, or the check cannot fail. The output looks the same in all three cases.

## Error

On the live tree, the agent-configuration guard printed:

```
CHECK: PASS
```

Its failure form is `CHECK: FAIL (n): ...` with exit code 1. Neither guard had been shown to fail on a tree that was broken on purpose.

Three faults, each green:

- **Wrong place.** The guard read the workspace copy of the rules, not the global rules file. A wrong count in the global file stayed unseen until that file was added to the audit.
- **Field gone.** A folder walk skipped an unreadable subtree without raising an error, and the folder passed clean.
- **Check that could not fail.** The guard checked a hand-kept list of routes instead of the paths the rule text cites. A cited path that had been removed still read as present.

## Root cause

A guard that reads only its own installation can be green for three reasons:

1. It points at the wrong place: a default path, or a copy that is not the one that matters.
2. The field it watches has disappeared, and the check treats absence as presence.
3. The check has become a no-op: it compares against something that cannot change, so it cannot fail.

Both guards defaulted to the live tree as their target. Nothing in their output showed which of the three cases held.

## Action

- **Target as a parameter.** The guard takes its target from an environment variable or a flag. The default is the live tree; tests pass a copy.
- **Unreadable is a finding.** An item that cannot be read is reported as a finding and never skipped.
- **Check what the text cites, on disk.** A path cited in a rule is checked against the file system. A cited path that does not exist fails.
- **Mutate every watched thing.** A harness copies a fixture tree, applies one mutation at a time, one per watched item, and requires the check that watches that item to fail. A mutation that stays green fails the harness.
- **Ship the harness with the guard.** A guard without a harness is decoration.

## Status

Mitigated. Both guards take their target as a parameter. A hub-copy harness and a project-copy harness exercise the same guard family in its two modes, and a mutation harness covers the agent-configuration guard. The limits are listed below.

## Evidence

- Hub-copy harness: builds a dirty copy of the hub in a temporary folder and plants one case per detector, each of which must be reported. Negative cases must stay clean. A clean copy must report zero findings and exit 0. A vanished file must be reported.
- Project-copy harness: the same pattern for the project mode, one planted case per detector. A clean project reports zero findings. A missing folder exits with code 2.
- Mutation harness for the agent-configuration guard: builds a minimal copy of the artefacts the guard reads, applies one mutation per variant, and requires the check that watches it to print FAIL. The last recorded run, in 2026-10, turned every mutation red (40 of 40). That run predates this entry and was not repeated for it.
- Expected result: every planted case is reported and every mutation turns its check red. One green mutation fails its harness.

## Promoted to

None in this repository. The guards and harnesses live in the private hub.

## What this does not prove

- That a guard reads every relevant field. Only the fields a harness mutates are covered. An unmutated field can stay unread while the guard stays green.
- That the 40-of-40 result still holds. It was recorded earlier and not repeated for this entry.
- That the mutation harness is portable. It still reads its source artefacts from fixed locations, so as written it runs only on the tree it was built for. Parameterising it is the same fix one level up, and it is not done.

## Checks to repeat

- Each guard on the live tree: it must print PASS.
- Each guard on a copy with one item removed or made unreadable: it must report a finding, never PASS.
- Each harness: every planted case and every mutation must be reported, and a single green mutation must fail the run.
