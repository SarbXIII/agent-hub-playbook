---
project: "{PROJECTS_ROOT}/{PROJECT}"
title: "{TITLE}"
model: "{MODEL}"
prompt_file: "{HUB}/dispatch/{BRIEF_FILE}"
auto: false
dry_run: false
---

# Dispatch brief — {TITLE}

> Written by the hub before opening a project session. One brief per dispatch.
> This is the assignment, not the work: the worker does the work in the project session.
> The hub writes the assignment into its own dispatch folder; the worker reads it from there. The hub never writes into the project.

## Fields

- **`project`** — the **absolute** path of the project directory. This is where the worker session opens. It must not be the hub workspace.
- **`title`** — short and specific, so the dispatch is identifiable in the notes and the session list.
- **`model`** — an **explicit** model reference (`{MODEL}`). A missing model means the session falls back to the default and may lack credit or fit: treat a missing model as a failed validation.
- **`prompt_file`** — the assignment written to a file in the hub's dispatch folder (`{HUB}/dispatch/`), never inside the project. It contains task + acceptance criteria + what not to touch.
- **`auto`** — `true` starts the worker without further confirmation; `false` waits for approval. Only set `true` when the work is cheap, reversible and already approved.
- **`dry_run`** — `true` prints what would be dispatched without opening a session; `false` dispatches for real.

## Validation — fail the dispatch, do not fix it silently

- [ ] `project` is **absolute** and is **not the hub** workspace.
- [ ] `prompt_file` lives inside the **hub's dispatch folder** (`{HUB}/dispatch/`) and never inside the project.
- [ ] `model` is **explicit** (`{MODEL}`). Missing or default model = reject.
- [ ] The prompt states the task, the acceptance criteria and what **not** to touch.
- [ ] `auto: true` only for work that is cheap, reversible and already approved.

## After the dispatch

- The worker reads the brief from the hub's dispatch folder, does the work in the project session, writes the hand-off into the project notes, then closes.
- The hub exports the transcript, archives it and deletes the session. One live manager only.
- Record the dispatch in the project notes: session, date, result.
