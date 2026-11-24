# Lessons

Generic, field-tested lessons. Each one follows the same discipline: **the error, the mechanism, the fix in the machine, and the evidence** — not just the story.

| Lesson | One-line takeaway |
|---|---|
| [Agent shell permissions](agent-shell-permissions.md) | Shell patterns match by substring, and an agent allow beats the global deny. Read-only agents must not have the tool. A pattern that matches less than intended reads as a broken tool. |
| [Atomic skill update](atomic-skill-update.md) | A two-rename folder swap is not atomic: check integrity out of band, and recover by restoring rather than merging. |
| [Baseline that cannot fail](baseline-that-cannot-fail.md) | A baseline that never goes red measures nothing: write a skill only after a reproducible failing baseline and a failure that repeats in real work. |
| [Context budget](context-budget.md) | Announced skills are a cold-start tax. Measure it, then scope it per agent. |
| [Dead client is not a finished task](dead-client-is-not-a-finished-task.md) | A dead client proves nothing about the task: read the session state before declaring it done, and never close on a proxy alone. |
| [Evaluating tool recommendations](evaluating-tool-recommendations.md) | A recommendation is evidence that a tool works, never that you need it. Four gates, then a written verdict with a reopening condition. |
| [Guard that cannot go red](guard-that-cannot-go-red.md) | A guard that cannot be pointed at a copy or made to fail is decoration: take its target as a parameter and mutate every watched item. |
| [Hub/project boundary](hub-project-boundary.md) | A boundary that lives only in prose is not a boundary. Encode it with a local role, hub-scoped permissions and a dispatch wrapper. |
| [Local estimate is not server truth](local-estimate-is-not-server-truth.md) | A local estimate built on an old limit table is not the server's figure: publish a monthly number only from server rows, and never label a local value as server data. |
| [One writer per field](one-writer-per-field.md) | Several scripts writing the same state field make the report lie: give each field exactly one writer, and make rebuilds unable to lose data. |
| [Phantom message](phantom-message.md) | An assistant can answer a message nobody sent: check the author against the session store before acting on a message that does not fit the thread. |
| [Quoted error is not a failure](quoted-error-is-not-a-failure.md) | A string match counts mentions, not events: count shell-tool errors, not quotes, and treat expected denials as a separate case until they have their own rule. |
| [Redirected logs can be UTF-16](redirected-logs-can-be-utf16.md) | A Windows PowerShell 5.1 redirect writes UTF-16 with a BOM that UTF-8 readers fail on: set the encoding at the write, guard every log, and normalise only files that carry a BOM. |
| [Reserved device name is not a folder](reserved-device-name-is-not-a-folder.md) | `NUL`, `CON` and the other Windows device names are reserved in every folder: prefix fixtures and temporary folders, and never use a bare reserved name. |
| [Self-imposed restrictions](self-imposed-restrictions.md) | A rule the agent writes for itself is code. Without its mechanics written and tested, it gets read as a broken tool. |
| [Verdict that is not persisted](verdict-not-persisted.md) | A verdict the session store does not keep is invisible to the gate: close on a validated reviewer-text file with its hash, never on a verdict rebuilt from memory. |
| [Worker digests](worker-digest.md) | Raw sources stay out of the reasoning context: a worker turns them into a small digest with paths, final state and open questions. |

New lessons are added only if they pass the publication filter: generic, no personal data, tested in real use.
