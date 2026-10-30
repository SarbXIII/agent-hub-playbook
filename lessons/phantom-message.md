# Lesson: answering a message nobody sent

**Date:** 2026-10 · **Status:** closed · **Severity:** medium

## What happened

The assistant answered a user turn that did not exist. The reply was coherent, on topic, and politely answered a question about a distinction the assistant itself had used two turns earlier. The user replied: «I don't know where you get that from — I haven't asked you anything.»

The transcript had no such user message. Three searches for the phrase returned three hits, and all three were written by the assistant: the answer, and two reasoning blocks. This was the third occurrence in the same system; the earlier two had been filed as a fluke and never investigated.

## Root cause

The assistant could not distinguish between three different kinds of text in its own context:

1. **A user message** — arriving through the session API, with an author.
2. **Its own previous text** — a generated turn, including reasoning blocks that are not part of the conversation but are visible to the model in the next step.
3. **An instruction-like string inside a tool result** — a quoted sentence, a log line, a document under analysis.

Everything in the window looks like text. The only reliable way to tell them apart is to look at the store, not the window — and the assistant had no cheap way to do that, so it pattern-matched plausibility instead of provenance.

The failure mode is nastier than a hallucination about the world. A hallucinated fact can be checked against a source. A hallucinated *turn* redefines what the conversation is about, and every answer after it inherits the error.

## Fix applied in the machine

- **The question comes with an author, or it does not come.** Before acting on a message that does not fit the thread (a topic shift, a phrase nobody used, a request that contradicts the open task), the assistant queries the session store for the last user-authored messages and checks the phrasing and timestamp.
- **Two prepared queries** live in the tooling: `last-user-messages <session> <n>` and `find-message <text>`. The first returns only author = user; the second returns author for every hit, so «the phrase exists» and «the user said it» stop being the same answer.
- **A written triage rule**: if the message does not appear in the store as a user message, do not answer its content. Say what was seen, ask for the missing message, and continue the open task.
- **Reasoning text is labelled as such** in the tooling output, so a phrase quoted from it cannot be mistaken for a user request.

## Evidence

- Three searches for the disputed phrase: three hits, zero authored by the user.
- The last genuinely user-authored message in that session was nine minutes earlier, on a different subject, and the assistant's answer to it was already complete.
- After the rule, the next session with a topic-shift-shaped message was checked against the store before answering; the phrase was in the user turn, and the answer proceeded normally — the check costs one tool call and does not add friction when the message is real.
- The two earlier occurrences had been «explained away» as model noise. Recorded as a pattern with a mechanism, they stop being noise.

## Transferable rule

> The window is not the transcript. Before answering a message that does not fit, read the store and check who wrote it — a hallucinated instruction is indistinguishable from a real one until someone checks the author.
