# Worker digest — {SOURCE}

> Written to `{DIGEST}` by `{WORKER}` after processing `{SOURCE}`. The main thread reads this file, not the source.
> Stay under `{THRESHOLD}`. A digest over the cap is a failed digest: cut fields, never grow the context.

## 1. Source

- **Path:** `{SOURCE_PATH}`
- **Format:** {FORMAT}
- **Size:** {SIZE}
- **Records:** {RECORD_COUNT}
- **Range covered:** {RANGE}

## 2. Worker

- **What it does:** {WORKER}
- **Runs:** local | remote
- **Model calls:** none | {MODEL_CALLS}

## 3. Cost

- **Worker spend:** {WORKER_COST} — `none — no model call` is a valid answer.
- **Wall-clock:** {WALL_CLOCK} — `not measured` is a valid answer.
- **Not saved by this digest:** the underlying work the source represents.

## 4. Operations

- {OPERATION} — {WHAT_WAS_READ_OR_COMPUTED}

## 5. Final state

- **Outcome:** {OUTCOME}
- **Artefacts:** `{ARTEFACT_PATH}` — {WHAT_IT_IS}
- **Last activity:** {LAST_CALL_OR_ENTRY}

## 6. Errors

- `{ERROR_TEXT}` at `{ERROR_LOCATION}` — {STATUS}

## 7. Uncertainties

- {WHAT_COULD_NOT_BE_ESTABLISHED} → look at `{PATH}:{LINES}`

## 8. Decision

- **Main thread should:** {DECISION}
- **Targeted read needed:** `{PATH}:{LINES}` for {REASON} — or `none`.

## Checklist

- [ ] Digest is under `{THRESHOLD}`; no raw payload pasted in.
- [ ] Source path, size and record count stated.
- [ ] Worker and model usage stated (`none` when deterministic).
- [ ] Final state names the artefact it rests on.
- [ ] Errors quoted literally, with their location.
- [ ] Uncertainties named, each with somewhere to look.
- [ ] Decision given, with a targeted read or an explicit `none`.
