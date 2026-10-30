# Lesson: when two scripts own the same field, the state lies in the shape of a success

**Date:** 2026-10 · **Status:** closed · **Severity:** high

## What happened

A hub keeps a manifest of what it has installed: one entry per external pack, with a copy of its contents and a field for *when it was last copied*. Three scripts wrote to that file. One copied the packs and stamped today's date. One re-mapped which local folders belong to which pack — and rewrote the date from its own hard-coded table. One regenerated the provenance table, also with dates.

Each script was correct in isolation. Run in the wrong order, the last one won, and the manifest said a pack had been copied two weeks earlier — minutes after it had been copied. The weekly report, reading that field, printed `HAS CHANGES` for ten packs that had just been updated. Nothing was broken. The report was faithfully reporting a number that another component had overwritten.

The same shape appeared three times in one day, in three fields:

- **Copy date** — overwritten by the mapper, so "update available" was permanently true.
- **Folder map** — the mapper wrote a list; a later run wrote a *count* in the same slot. The list was gone, and every lookup by pack fell back to "no owner".
- **Relations** — re-indexing the registry ran a delete of all edges before rebuilding, so a routine reindex silently discarded most of them. The rebuild only recreated the ones derivable from the source files; everything declared by hand disappeared.

In all three cases the symptom looked like *missing work*: packs unowned, updates pending, a graph suddenly thin. The actual cause was *duplicate ownership*.

## Root cause

A field that two writers can set has no author. The failure is not a race — the order is deterministic and still wrong — it is an **unstated ownership rule**. Three forces:

1. **The field was named for its meaning, not its writer.** "Last copied" reads like a fact about the world; it is a fact about one script's last run. Any script that knows a plausible value feels entitled to write it.
2. **Writes were additive and idempotent-looking.** Insert-or-ignore, a single `UPDATE`, a full rewrite of one JSON key: each looks safe. Idempotent is not the same as non-destructive, and "safe to re-run" was read as "safe to run from anywhere".
3. **The readers never checked.** A report that prints a number with no provenance cannot tell "nothing happened" from "someone overwrote the evidence".

The cost is subtle: the system keeps working, so nobody investigates. The agent then "fixes" the symptom — re-copies the packs, re-maps the folders, re-adds the edges — and the next reindex undoes it again. Twice the work, and a growing suspicion that the tooling is unreliable when the tooling is merely unowned.

## Fix applied in the machine

- **One writer per field, written next to the field.** The copy date is written only by the copy step; the folder map only by the mapper; the provenance table only by the provenance step. Comments in each script name the owner of every key it touches.
- **Reindexing preserves what it cannot reconstruct.** The registry no longer deletes edges: it upserts records, keeps edges, and removes only edges whose endpoints no longer exist. Rebuilding an index must be runnable at any time without costing data.
- **A reconciliation step** runs after the writers and enforces the invariant that a folder with an owner cannot also be listed as unowned; it reports both counts, and they must add up.
- **Provenance is printed.** The report shows the manifest's own update date next to each value, so a stale number is visible as stale.
- **A quality gate**: any component that rewrites a whole document must read the current value first and carry forward the keys it does not own.

## Evidence

- Three fields, three incidents, one day. Measured outcomes: ten packs reported as outdated minutes after being copied; a set of folders lost their owner after a run wrote a count where a list belonged; most edges of a relation graph were destroyed by a reindex that then reported `CHECK: PASS`, because the check validated the records and not the edges.
- After the fix: the same reindex ran twice with no loss, the manifest reconciled every installed folder to an owner, and the weekly report returned to reporting real deltas.
- The two false-positive audits that followed produced a second guard — every edge must cite a checkable source — documented as its own pattern in `templates/relation-registry.md`.

## Transferable rule

> A field with two writers has no author. Name the writer next to the field, make the index rebuildable without loss, and print the provenance of every number a report shows — otherwise the success of one script silently rewrites the truth of another.
