# Lesson: ignoring the output folder and its index leaves one copy on disk

**Date:** 2026-11 · **Status:** mitigated for new analyses; the four analyses lost before the change are not restored · **Scope:** research output folders (reports, metadata, transcripts) and the state index that tracks them, in a git repository

**In short.** A `.gitignore` excluded both the output folder and the state index. The folder held the only copy of the deliverables. When it was deleted, four analyses were lost: git had no history of the folder, so nothing in the repository could restore them.

## Error

- Four analyses from earlier months disappeared from the output folder.
- `git log --all -- <folder>` returned nothing.
- Only references to the analyses remained, in a skill and in the index.

## Root cause

- The `.gitignore` excluded the artefact folder and the state index at the same time.
- With both ignored, the disk held the only copy of the deliverables. Git could not bring them back.

## Action

- **Version the text artefacts:** the report, the metadata and the transcript.
- **Keep a durable state log:** the state index is a JSON Lines file, written atomically and managed by a small queue.
- **Ignore only regenerable media:** video, audio, frame dumps and contact sheets. They can be rebuilt, as long as the source still exists.
- **Closing rule:** an analysis is closed only when its report is present and the reconciliation is green.

## Status

Mitigated for new analyses. The four analyses lost before the change are not restored; only references to them remain.

## Evidence

- An import read 9 items and left 7 pending.
- `git log --all` for the folder was empty before the change.
- The ignore patterns after the change exclude only regenerable media.

## What this does not prove

- That git is a backup. A local repository does not protect against disk failure.
- That media can always be regenerated. That depends on the source still existing.
- That the rule covers anything beyond text and state. Only text artefacts and the state record are covered.
