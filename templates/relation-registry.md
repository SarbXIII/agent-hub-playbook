# Template: relation registry with an evidence gate

> For hubs that accumulate evaluations, lessons, tools and decisions faster than anyone can reread them. The registry is an **index**, not a store: the readable files stay the source of truth and the database can be deleted and rebuilt. What it adds is the one thing files cannot: asking «what do I already know about X, and why is it linked to Y?»

## Schema

Two tables. Nothing else is needed to start.

```sql
CREATE TABLE records (
  id      TEXT PRIMARY KEY,   -- stable, prefixed by type: lec: sk: ev: vid: proj: rad: dec:
  type    TEXT NOT NULL,      -- closed list
  title   TEXT NOT NULL,
  date    TEXT,               -- YYYY-MM-DD when known
  state   TEXT,               -- alive · parked · discarded · adopted
  source  TEXT,               -- where it came from, in words
  path    TEXT,               -- file on disk, when there is one
  summary TEXT
);

CREATE TABLE relations (
  from_id  TEXT NOT NULL,
  kind     TEXT NOT NULL,     -- closes the list below
  to_id    TEXT NOT NULL,
  reason   TEXT NOT NULL,     -- why the edge exists; never empty
  PRIMARY KEY (from_id, kind, to_id)
);
```

**Closed list of edge kinds.** Keep it short enough that a human can hold it, and refuse anything else:

| kind | reads as | example |
|---|---|---|
| `from` | this came out of that | `sk:brand-review → ev:pack-2026-10-09` |
| `documents` | this file describes that | `ev:pack-2026-10-09 → sk:brand-review` |
| `applies_in` | this is used in that context | `lec:dispatch-without-credit → proj:billing` |
| `replaces` | this supersedes that | `sk:taste-v2 → sk:taste-v1` |
| `contradicts` | these two cannot both be true | `dec:a → dec:b` |

## Indexing rules

1. **Idempotent and non-destructive.** Indexing inserts or updates; it never deletes edges. Orphaned edges (endpoint missing) are swept, and that is the only deletion it performs.
2. **Ids are stable and human-readable.** `type:date-slug`, so a link survives a reindex and can be typed by hand in a Markdown file.
3. **Every record keeps its path.** The registry points at the file; it never becomes the only copy.
4. **Queries that pay for themselves**: *what do I know about X* (title and body search), *what is parked* (state + deadline), *what happened this week*, *what is unlinked*.
5. **The index is rebuildable by design.** If rebuilding loses something, the design is wrong, not the operator.

## The evidence gate (the part that matters)

A registry invites proposals: from a script, from a weaker model, from a past session. Most of them are plausible and some are wrong. The gate is not «is this true?» — no checker can answer that — it is **«can the claim be checked?»**.

A proposed edge is accepted only if its `reason` cites at least one artifact that exists:

- a **path** that exists on disk (`notes/eval-x-2026-10-09.md`);
- a **record id** that exists in the registry;
- a **session id** that exists in the agent's store.

Anything else is stored with state `doubtful` and **does not enter the graph**. Form is checked too: ids exist, kind is in the closed list, reason is at least a sentence, the edge is not a duplicate.

Two consequences worth stating:

- **A weak model can contribute.** It only has to produce four fields; all the guaranteeing is done by a deterministic checker. You do not need a strong model to write to the graph, you need a checker that says no.
- **The validator checks evidence, not meaning.** An edge can cite a real file and still be wrong. That is why every edge carries its `reason` and why a random sample is reviewed by hand after any bulk import — the sample is what catches «real source, wrong conclusion».

## Failure modes this template exists to prevent

- **Shared paths create phantom edges.** If every record points at the same file, matching on paths links everything to everything. Measured case: 81 edges from one shared root path, all false. Only distinctive file names may be used as evidence.
- **Bulk import inflates the graph.** A first pass produced 413 edges from a naive text search; almost all were noise because long-lived records shared a single log file. Cap the first import, sample 8 edges, then decide.
- **A silent delete in the rebuild.** `CHECK: PASS` on records while the edges are gone is the worst kind of green.

## Closing check

```
records > 0 · relations ≥ 1 per imported source · orphan edges = 0
proposed edges: accepted only with a checkable citation
random sample of 8 edges: read the reason, open the source, agree or remove
```
