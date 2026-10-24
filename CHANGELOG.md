# Changelog

All notable changes to this playbook. The model is versioned; field lessons are added as they are validated.

## [Unreleased]

- New field lesson `lessons/self-imposed-restrictions.md`: a shell restriction the agent imposed on itself was documented as intent («commands must contain the workspace path») instead of mechanics (the engine checks every segment and ignores the working directory). One denied command was read as a broken tool, a task was completed without the capability, and the agent reported the tool as unavailable. The rule now carries dated pass/fail examples; self-imposed restrictions carry mechanics, date and owner, and are revocable.
- New field lesson `lessons/evaluating-tool-recommendations.md`: a five-item "free plugins" recommendation collapsed to one TRY, two WATCH and two REJECT when scored against real gaps, with the two most popular entries rejected. Adds four adoption gates and a written verdict with a reopening condition.
- `lessons/agent-shell-permissions.md` gained a section on the inverse failure: substring allows also match less than intended, a chained command is denied when any segment fails to match, and the result is indistinguishable from a missing tool.
- New field lesson `lessons/worker-digest.md`: a 48.4 MB JSONL log was processed by a local deterministic worker into a ~6 KB digest; main-thread context grew 0.5 pp and the digest recovered a final artefact the main thread had reported missing. Measured in one case, not a controlled study.
- New template `templates/worker-digest.md`: the reusable digest contract (source, worker, cost, operations, final state, errors, uncertainties, decision) with the `{SOURCE}`, `{WORKER}`, `{DIGEST}` and `{THRESHOLD}` placeholders.
- New field lesson `lessons/hub-project-boundary.md`: the hub executed a project's work in its own session; the boundary is now enforced on the hub side with a local role, hub-scoped permissions and a dispatch wrapper, with project-side isolation left as the recommended next layer.
- New template `templates/dispatch-brief.md`: a machine-readable dispatch brief (`project`, `title`, `model`, `prompt_file`, `auto`, `dry_run`) with validation rules (absolute project, not the hub, prompt in the hub's dispatch folder, explicit model).
- `templates/constitution.md` §3 now links the boundary lesson and the dispatch-brief template.

## [0.1.1] — 2026-10-19

- Repository visuals under `assets/`: header banner, hub + islands diagram and dispatch lifecycle diagram (light/dark variants for `prefers-color-scheme`), plus a 1280×640 social preview card.

## [0.1.0] — 2026-10-19

First public edition.

- English and Spanish editions of the specification: principles, phases, up-front interview, anti-patterns, glossary, operating protocol, personality, adapt checklist.
- Templates: constitution, read-only agent, reviewer agent, skill, memory, lesson, hand-off.
- Field lessons: shell allowlists match by substring and agent allows override global denies; announced skills are a cold-start cost.
