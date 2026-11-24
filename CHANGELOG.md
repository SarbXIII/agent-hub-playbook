# Changelog

All notable changes to this playbook. The model is versioned; field lessons are added as they are validated.

## [Unreleased]

- New field lesson `lessons/verdict-not-persisted.md`: a closing gate rejected a delegated reviewer's ACCEPT twice because the session store kept the final message without text. Adds a validated reviewer-text file, with its path and SHA-256 recorded at close; persisting the final text in the runner stays open.
- New field lesson `lessons/local-estimate-is-not-server-truth.md`: a usage script published a monthly share from a stale limit table, about 7.5× the provider console's figure, and an edge case labelled local numbers as server data. Adds withholding of the monthly figure without server rows, and labels that follow the rows. The limit table is not recalibrated; recalibration stays open.
- New field lesson `lessons/quoted-error-is-not-a-failure.md`: a watcher warned about a repeated shell error because it counted completed results that only quoted the error text. It now counts shell-tool error results only, with a citation fixture that gives zero; expected denials still count, so that half stays open.
- New field lesson `lessons/dead-client-is-not-a-finished-task.md`: a task watcher declared a turn finished from a quiet log and a dead client process, while the turn kept running on the server. Session state is now read first, and a transport error means "client lost", not "done". Two gaps stay open: the no-state path still closes on proxies, and a CHECK line seen while the session is busy ends the watch early.
- New field lesson `lessons/baseline-that-cannot-fail.md`: a three-scenario baseline passed 15 of 15, so the scenarios could not show the failures that happened at scale. Adds the rule: no skill without a reproducible red baseline, seam cases in every baseline, and a skill only after the same failure recurs twice in real work.
- New field lesson `lessons/guard-that-cannot-go-red.md`: two consistency guards read their own installation and could report PASS while pointing at the wrong place, reading a missing field, or running a check that cannot fail. Adds a target parameter, unreadable items reported as findings, and mutation harnesses that require each watched item to turn red.
- New field lesson `lessons/self-imposed-restrictions.md`: a shell restriction the agent imposed on itself was documented as intent («commands must contain the workspace path») instead of mechanics (the engine checks every segment and ignores the working directory). One denied command was read as a broken tool, a task was completed without the capability, and the agent reported the tool as unavailable. The rule now carries dated pass/fail examples; self-imposed restrictions carry mechanics, date and owner, and are revocable.
- New field lesson `lessons/evaluating-tool-recommendations.md`: a five-item "free plugins" recommendation collapsed to one TRY, two WATCH and two REJECT when scored against real gaps, with the two most popular entries rejected. Adds four adoption gates and a written verdict with a reopening condition.
- `lessons/agent-shell-permissions.md` gained a section on the inverse failure: substring allows also match less than intended, a chained command is denied when any segment fails to match, and the result is indistinguishable from a missing tool.
- New field lesson `lessons/worker-digest.md`: a 48.4 MB JSONL log was processed by a local deterministic worker into a ~6 KB digest; main-thread context grew 0.5 pp and the digest recovered a final artefact the main thread had reported missing. Measured in one case, not a controlled study.
- New template `templates/worker-digest.md`: the reusable digest contract (source, worker, cost, operations, final state, errors, uncertainties, decision) with the `{SOURCE}`, `{WORKER}`, `{DIGEST}` and `{THRESHOLD}` placeholders.
- New field lesson `lessons/hub-project-boundary.md`: the hub executed a project's work in its own session; the boundary is now enforced on the hub side with a local role, hub-scoped permissions and a dispatch wrapper, with project-side isolation left as the recommended next layer.
- New template `templates/dispatch-brief.md`: a machine-readable dispatch brief (`project`, `title`, `model`, `prompt_file`, `auto`, `dry_run`) with validation rules (absolute project, not the hub, prompt in the hub's dispatch folder, explicit model).
- `templates/constitution.md` §3 now links the boundary lesson and the dispatch-brief template.
- New field lesson `lessons/one-writer-per-field.md`: three scripts wrote the same three state fields; the report then claimed ten packs were outdated minutes after updating them, a folder map was overwritten by a count, and a routine reindex destroyed most edges of a relation graph while still reporting `CHECK: PASS`. Adds the one-writer-per-field rule, a rebuild that cannot lose data, and provenance printing.
- New field lesson `lessons/atomic-skill-update.md`: a two-rename folder swap is not atomic; a service restart landed between the renames and left an installed procedure with no real name. Adds an out-of-band integrity check, a recovery script that restores instead of merging, and a post-crash checklist.
- New field lesson `lessons/phantom-message.md`: the assistant answered a user message that was never sent; the phrase existed only in its own output and reasoning. Adds an author check against the session store and a triage rule before acting on a message that does not fit the thread.
- New template `templates/relation-registry.md`: a two-table index (records + typed edges with a mandatory reason) whose rebuild is non-destructive, plus an **evidence gate** — a proposed edge is accepted only if its reason cites a path, a record, or a session that exists. Written after 413 and 81 false edges were caught by sampling.
- New template `templates/role-brain.md`: the fixed shape for a role skill (`SKILL.md` with five sections plus standards, skill map, checklists, lessons, sources) and the deterministic verifier that keeps three brains from drifting into three shapes.

## [0.1.1] — 2026-10-19

- Repository visuals under `assets/`: header banner, hub + islands diagram and dispatch lifecycle diagram (light/dark variants for `prefers-color-scheme`), plus a 1280×640 social preview card.

## [0.1.0] — 2026-10-19

First public edition.

- English and Spanish editions of the specification: principles, phases, up-front interview, anti-patterns, glossary, operating protocol, personality, adapt checklist.
- Templates: constitution, read-only agent, reviewer agent, skill, memory, lesson, hand-off.
- Field lessons: shell allowlists match by substring and agent allows override global denies; announced skills are a cold-start cost.
