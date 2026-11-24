# Lesson: a verdict that is not persisted cannot close a gate

**Date:** 2026-11 · **Status:** mitigated for the closing path; the runner still does not persist the final text · **Scope:** closing gates that accept a delegated reviewer's verdict

**In short.** A closing gate read only the session store. A delegated reviewer had emitted its verdict and the caller had received it, but the stored final message had no content parts, and the gate refused the close twice. For the gate, a verdict that was emitted but never written down is no verdict. The workaround is a validated reviewer-text file, with its path and SHA-256 recorded at close.

## Error

The gate refused the close that carried the reviewer's `VERDICT: ACCEPT`. The message, translated here, read: "the message has no text; reasoning is not an emitted verdict". The reviewer's answer had reached the caller, so the verdict existed on the transport and was missing from the store.

## Root cause

- The final response of a native subagent is delivered to the invoker, but the runner does not persist it in the session store.
- In the store, the reviewer's final message has an empty content list and `finish: stop`. The turn ended; the text is absent.
- The gate reads only that store. An empty content list is read as "no verdict".
- Two surfaces, one gate: the transport carried the verdict and the store did not. A verdict that is not written down cannot be audited, and a gate cannot close on it later.
- Reasoning is not an emitted verdict, by the gate's own rule. A verdict rebuilt from reasoning or memory is a claim the gate cannot check.

## Action

1. **Implemented: validated reviewer-text file.** The close command accepts a JSON file with the task, the session, the message, the text, the declared model and the variant. The gate validates the file with seven checks before it uses it:
   1. the task identifier equals the task being closed;
   2. the session equals the reviewer session given at close;
   3. the message equals the last assistant message after the request that cites the task, the one that persisted no text;
   4. the declared model is contained in the effective model, which must name the base reviewer's model;
   5. the session's recorded model is the base reviewer's model;
   6. the session's variant is exactly the base reviewer's variant;
   7. the text passes the emitted-verdict parser without normalisation.

   If the stored message has text, the normal path decides and the file is ignored. On close, the gate records the file path and the SHA-256 of the bytes it validated, read in the same pass, and prints both.
2. **Open: persist the final text.** The runner would write the final text into the store, and the gate would need no side file.
3. **Interim rule, for a reviewer without a validated text file.** Close as `NO_REQUIERE` (not required), with a written reason and a written resolution, and keep the review record as evidence. Never reconstruct an `ACCEPT` from reasoning or memory.

## Status

- Mitigated for the closing path: a validated reviewer-text file is accepted, and its path and hash are recorded at close.
- Exercised twice: once on a temporary reproduction of the original rejected case, and once on a real close of a later review, which had a different task, session and message.
- Open: the runner still does not persist the final text of a native subagent. The file is a workaround, not the fix.

## Evidence

- Two rejections by the gate for the same case: the stored final message had no content parts.
- The validator suite passes 19 of 19 cases. One case uses the real session and message, whose stored content is empty, and it closes.
- Mutation check: with the model and variant comparisons disabled, three cases fail. After the comparisons are restored, the suite passes 19 of 19 again.
- The closing tool's self-test passes 167 of 167.
- The real close was recorded with the reviewer-text path and its SHA-256.

## What this does not prove

- That the model behind the text is the declared one. The checks compare the declared model with the effective model in the file, and the session's recorded model and variant with the base reviewer's configuration. They do not show which model produced the text beyond that record.
- That the transport and the store behave the same across runner versions. Each version needs the same check.
- That the file route covers other reviewers. The checks are fixed to the base reviewer's configuration; a second reviewer closes through a separate evidence route, which this lesson does not cover.
