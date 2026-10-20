# Lesson: worker digests — keep raw data out of the main context

**Date:** 2026-10 · **Status:** measured in one case, not a controlled study · **Severity:** medium

## What happened

A session had to analyse a structured log: **48.4 MB** in **393 JSONL lines**, with a few very large payloads. Reading it in the main thread would have pulled most of the file into the context that was doing the reasoning — and the reasoning was exactly what that context had to keep room for.

Instead the file went to a **worker**: a deterministic local parser, no model call. It read the raw file, computed the aggregates and wrote a **~6 KB digest** (`{DIGEST}`). The main thread read only the digest. Main-thread context moved from **18.6 % to 19.1 %** (+0.5 pp) across the whole analysis.

Two things the digest proved beyond the budget:

- It recovered the **final state** of the run it summarised: the operations performed, errors and their literal text, the last calls, and the artefacts written.
- It surfaced a **final output file (MP4) that the main thread believed did not exist**. The main thread had inferred absence from an incomplete read; the digest, working over the whole file, contradicted it.

## Root cause

The failure mode is not «the model read too much». It is that raw data and reasoning share one context window, and the raw data is **bigger, less useful and re-readable from disk**:

1. **Raw files are the worst tenants.** A log or dump is large, repetitive and mostly irrelevant; the few lines that matter are a rounding error of it. Holding the whole thing evicts the reasoning it was meant to serve.
2. **One read is not the real cost.** A large read stays in the history and is re-sent on every later turn of the session; a second look at the same file pays for it again.
3. **Incomplete reads produce false conclusions.** A truncated or partial look at a big file yields confident statements («there is no output file») that the data contradicts.
4. **The model does the cheap thing.** If the raw file is one tool call away, it gets read. The machine — not the discipline — has to decide where the data is processed.

## The pattern

> **Source → worker → digest → main thread.**

The worker is deterministic wherever the data has structure (a parser, a query, a script) and calls a model only when the data is unstructured. The main thread never reads the source: it reads the digest and opens the raw file only for a **directed** look at a path and line range the digest names.

Cost note: the worker's own work is what it is. What was measured here is **context and re-reads**, not the analysis the source represents — do not claim the digest makes the underlying job cheaper.

## The digest contract

Every digest answers the same eight questions, in this order. `{THRESHOLD}` is the size cap: a digest over it is a failed digest, not a request for more context.

1. **Source** — absolute path, format, size, record count, range covered.
2. **Worker** — what processed it, where it ran, whether it called a model.
3. **Cost** — what the worker consumed (model spend if any, wall-clock if measured). «No model call» is a valid answer.
4. **Operations** — what was read or computed, so the digest can be reproduced.
5. **Final state** — the end state of the run, naming the artefact it rests on.
6. **Errors** — failures with their literal message, not a paraphrase.
7. **Uncertainties** — what could not be established, with a path to look at.
8. **Decision** — what the main thread should do, or which targeted read is needed (`none` is a valid answer).

`templates/worker-digest.md` is the fill-in version of this contract.

## Evidence

- Source: one JSONL log, **48.4 MB**, **393 lines**.
- Digest: **~6 KB**, written to disk; the main thread read the digest, not the log.
- Main-thread context: **18.6 % → 19.1 %** (+0.5 pp) over the session.
- Worker: deterministic parser, local, **no model call**.
- The digest recovered the run's final state and surfaced an MP4 artefact the main thread had reported as missing.
- One case, one source type. No run of the same task with the raw file in context was measured, so this is an observation with a mechanism — not a benchmark.

## Limits

- **The digest is lossy by construction.** It is a summary with pointers, not the data. Anything outside its fields is gone until someone reads the raw file.
- **It does not replace deep QA.** Verifying a deliverable still needs the deliverable; a digest that says «done» is not a check.
- **It is only as good as the worker.** A parser that misses a field produces a confident, wrong digest. That is why uncertainties are mandatory and the final state must name its artefact.
- **The threshold is a heuristic.** Above `{THRESHOLD}`, digest first; below it, a direct read is fine and the ceremony costs more than it saves.
- **Targeted reads still happen.** The win is that they are targeted — a path and a line range — not that the raw file is never opened again.

## Transferable rule

> If a source is too big to hold, do not read it into the reasoning context: process it with a worker and read a small digest that names the source, the final state, the errors and the open questions.
> Judge the digest by what it lets the main thread decide without opening the source — and keep the raw file on disk for the directed read it will still occasionally need.
