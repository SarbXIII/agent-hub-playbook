# Lessons

Generic, field-tested lessons. Each one follows the same discipline: **the error, the mechanism, the fix in the machine, and the evidence** — not just the story.

| Lesson | One-line takeaway |
|---|---|
| [Agent shell permissions](agent-shell-permissions.md) | Shell patterns match by substring, and an agent allow beats the global deny. Read-only agents must not have the tool. A pattern that matches less than intended reads as a broken tool. |
| [Context budget](context-budget.md) | Announced skills are a cold-start tax. Measure it, then scope it per agent. |
| [Evaluating tool recommendations](evaluating-tool-recommendations.md) | A recommendation is evidence that a tool works, never that you need it. Four gates, then a written verdict with a reopening condition. |
| [Hub/project boundary](hub-project-boundary.md) | A boundary that lives only in prose is not a boundary. Encode it with a local role, hub-scoped permissions and a dispatch wrapper. |
| [Self-imposed restrictions](self-imposed-restrictions.md) | A rule the agent writes for itself is code. Without its mechanics written and tested, it gets read as a broken tool. |
| [Worker digests](worker-digest.md) | Raw sources stay out of the reasoning context: a worker turns them into a small digest with paths, final state and open questions. |

New lessons are added only if they pass the publication filter: generic, no personal data, tested in real use.
