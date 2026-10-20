# Lesson: the hub/project boundary is prose until a task feels too small to dispatch

**Date:** 2026-10 · **Status:** closed · **Severity:** high

## What happened

The hub — the manager assistant that coordinates work and dispatches to project sessions — executed a project's work in its own session. Three tasks that belonged to a project ran inside the hub instead: a script, a model test and a piece of design work.

Each task was small enough that opening a separate project session looked like overhead, so the hub did the work where it already was. The outputs were fine. The failure was not the result — it was that the boundary had been crossed and nothing in the machine noticed or stopped it.

## Root cause

The boundary existed only as a sentence («the hub does not execute project work»). Prose is not a control:

1. **The hub had full access to every project folder.** Nothing in its permissions distinguished the hub's own workspace from a project, so doing the work locally was cheaper than dispatching — and the model does the cheaper thing.
2. **Dispatch was a discipline, not a mechanism.** The steps (open a project session, explicit model, task + acceptance criteria) lived in prose. Nothing forced them, so nothing could be skipped.
3. **Small tasks are where a boundary decays first.** A large task is obviously a dispatch; a one-line script is not. The habit eroded at the smallest end, exactly where it is easiest to justify doing it in place.

## Fix applied in the machine

**Applied and tested on the hub:**

- **A local role.** The hub's instruction file now states it coordinates and dispatches, and does not execute project work in its own session.
- **Hub-scoped permissions.** `edit` is allowed only inside the hub's own workspace and denied in project folders; `subagent` is denied except for read-only roles (explorer, reviewer); `shell` is limited to commands that reference the hub workspace, with explicit denies on the known project folders.
- **A dispatch wrapper.** Opening a project session goes through a wrapper that validates the project directory, the model and the brief location before spawning. Dispatch is no longer a list of steps to remember; it is the path that works.

**Recommended next layer, not applied here:**

- **Project-side isolation.** Scope each project session to its own folder through its local configuration, so the hub workspace is not reachable from the project. This closes the boundary in the other direction; it is the recommended next layer, not applied here.

## Evidence

- Direct checks on the hub: a `write` to a project folder was denied; launching an executor subagent was denied; a `shell` with a project working directory was denied; a `write` to the hub workspace was allowed; a `shell` mixing the hub workspace and a project path was denied.
- The original run wrote project files from the hub; after the fix, the same kind of write is denied.
- The wrapper validates the project directory, the model and the brief location before spawning; a dry run was used to verify the command. Invalid cases (missing project, missing model, brief outside the hub's dispatch folder) are still to be exercised.
- One real dispatch opened a project session correctly.

## Transferable rule

> A boundary that lives only in prose is not a boundary; it is an intention.
> Encode the hub/project split in permissions (a local role plus hub-scoped permissions), route dispatch through a wrapper, and test that the hub cannot do project work — especially the «too small to dispatch» tasks, where the boundary decays first. Close the other direction (project-side isolation) as the next layer.
