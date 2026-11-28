# Lesson: an excerpt analysed as the whole source closes on a partial base

**Date:** 2026-11 · **Status:** mitigated by a closing step; no automated check is recorded · **Scope:** analysis reports built from downloaded video excerpts, using transcripts, frames and automatic speech recognition (ASR)

**In short.** A 57-second cut was analysed as if it were the full source. The video itself said at its end that the full version was elsewhere. The pipeline read the transcript and the frames only. It did not read the description or check the last seconds, where the pointers to the full source sit and where ASR is unreliable.

## Error

- A report treated a 57-second cut as the full source.
- The video said at its end that the complete version was in another place. The report did not follow that pointer.
- The same failure recurred within one batch: a repository mentioned in a source was not reviewed.

## Root cause

- The pipeline analyses only the transcript and the frames.
- It does not read the description, and it does not check the last seconds for pointers.
- ASR is not reliable for the final notices, which is where the pointers to the full source sit.

## Action

Before a report is closed:

1. **Read the description and the last 10 seconds.**
2. **Follow the pointers.** If they name a long video, a repository, a newsletter or a web page, locate it and analyse the span it names.
3. **Record the coverage.** Note what the pointed source adds and what remains unseen.
4. **State the limit.** If the full source cannot be analysed, say so in the report's analysis-limits section.

## Status

Mitigated by a closing step. The report carries a verification section. No automated check is recorded.

## Evidence

- A contact sheet of the last seconds of the excerpt.
- A verification section in the report.
- The full source runs 1:41:48, and the pointer is at 28:00.

## What this does not prove

- That ASR becomes reliable. The rule works around its weakness at the pointers; it does not fix it.
- That the full source is always available. It may not be.
- That the check is exhaustive. It checks the pointers, not a full comparison of the two sources.
