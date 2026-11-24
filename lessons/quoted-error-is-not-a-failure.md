# Lesson: a quoted error is not a failure

**Date:** 2026-11 · **Status:** partly mitigated; quotes are excluded and tested, expected denials still count · **Scope:** watchers that warn on repeated errors across agent session results

**In short.** A watcher warned that the same shell error had happened three times in a delegated session. Two kinds of result had been counted that should not have been: completed results that only quoted the error text, and shell denials that the session profile applies on purpose. The fix excludes quotes and counts only shell-tool errors. It does not yet separate expected denials, so the warning can still fire on them. A string match counts mentions, not events.

## Error

The warning, translated here, read: "three times the same error: Permission denied: shell". In that session the profile had denied three shell commands on purpose, and several completed results contained the same text inside file or tool content. The warning named a phrase, not a session or a tool.

## Root cause

- The watcher searched for the error phrase in the output of every tool result, not only in failed shell results.
- A completed result that quotes the error, such as a file read or a summary result, contains the phrase. The result succeeded, and it was counted as a failure.
- A denial that the profile applies on purpose is a shell error with the same text, so it counted toward the repetition. An expected behaviour was reported as a repeated failure, which reads as an architecture problem.
- Mechanism: a string match reads neither status nor origin. A quote and an event look the same to the matcher; status and origin are what separate them.

## Action

1. **Implemented: count shell-tool error results only.** A result counts when its status is error and its origin is the shell tool. Completed results that quote the phrase are ignored, and a denial from another tool, such as a write, does not count as a shell error.
2. **Implemented: citation fixture.** Three completed reads that cite the phrase must give zero signatures and no warning. One shell error must give one signature and no warning, because the warning needs three repetitions.
3. **Pending: expected denials.** A shell denial the profile applies on purpose is still a shell-tool error, so the current rule counts it. Pending: a rule that separates the profile's expected denials from failures, with a fixture of three expected denials that must produce zero warnings.
4. **Pending: warning content.** A repetition warning should name the session and the originating tool, not only the phrase.

Until the expected-denial rule exists, a repetition warning is read with its session in front.

## Status

- Partly mitigated. The citation case is fixed and covered by a fixture: completed reads that quote the phrase give zero, and one real shell error gives one.
- Open: expected denials. The rule still counts them, so the repetition warning can still fire on the profile's own denials.
- Open: warning content. The evidence does not show that the warning names the session and the originating tool.

## Evidence

- A real run of the watcher's log: three shell errors (the profile's expected denials), one denied write, and completed reads and summary results that quote the phrase.
- Before the fix, the warning counted quotes as well as denials.
- After the fix, the watcher counts the three real shell errors and none of the quotes.
- The fixture covers both sides of the rule: three completed reads that cite the phrase give zero signatures, and one shell error gives one signature with no warning.
- One real run and one fixture set, in one profile.

## What this does not prove

- That every runner labels the tool origin the same way. The rule depends on the origin field; another runner could label it differently and change the count.
- That an expected denial is harmless in every profile. In this profile the denials were intended. Another profile might treat the same denial as a fault, and the rule does not yet tell them apart.
- That one run represents other sessions.
