# Lessons

Generic, field-tested lessons. Each one follows the same discipline: **the error, the mechanism, the fix in the machine, and the evidence** — not just the story.

| Lesson | One-line takeaway |
|---|---|
| [Agent shell permissions](agent-shell-permissions.md) | Shell patterns match by substring, and an agent allow beats the global deny. Read-only agents must not have the tool. |
| [Context budget](context-budget.md) | Announced skills are a cold-start tax. Measure it, then scope it per agent. |
| [Hub/project boundary](hub-project-boundary.md) | A boundary that lives only in prose is not a boundary. Encode it with a local role, hub-scoped permissions and a dispatch wrapper. |

New lessons are added only if they pass the publication filter: generic, no personal data, tested in real use.
