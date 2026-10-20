# Changelog

All notable changes to this playbook. The model is versioned; field lessons are added as they are validated.

## [Unreleased]

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
