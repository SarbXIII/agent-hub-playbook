# Lesson: a shell redirect can write UTF-16 into a log corpus

**Date:** 2026-11 · **Status:** mitigated for the reported corpus; recurrence is possible on any plain redirect · **Scope:** log files written by shell redirection on Windows, and the tools that read them

**In short.** In Windows PowerShell 5.1, a plain `>` redirect writes UTF-16LE with a byte-order mark (BOM). Readers that assume UTF-8 fail or drop the NUL bytes, while the command itself looks correct. This was the fourth recurrence of the same failure. The pattern is a write route that sets the encoding, a guard over every log, and a normaliser that changes only files with a BOM, followed by a second pass that must change nothing.

## Error

- In a 190-file log corpus, 66 files were offenders: 57 in UTF-16LE and 9 in UTF-8 with a BOM (`EF BB BF`).
- Parsers and `grep` returned unexpected results on the corpus.
- The command that produced the files was correct.

## Root cause

- In Windows PowerShell 5.1, `>` is functionally equivalent to piping to `Out-File` with no extra parameters. The default encoding of `Out-File` is `unicode`, which is UTF-16 little-endian, and the file starts with the BOM `FF FE`.
- The 57 UTF-16LE files match this default.
- A UTF-16 file has a zero byte after every ASCII character. Readers that assume UTF-8 fail or drop those bytes, and `grep` results become unreliable.
- The defect sits in the write route, not in the command text, so the symptom returns on every log written through the same route.
- Nothing checked the encoding of the logs before readers consumed them.

## Action

The pattern has four steps.

1. **Write through a route that sets the encoding.** Use the producing tool's own output-file option when it has one, or an explicit UTF-8 writer without a BOM (in Windows PowerShell, `New-Object System.Text.UTF8Encoding $false`, passed to `[System.IO.File]::WriteAllText`). A `cmd /c … >` redirect avoids PowerShell's UTF-16 writer but sets no encoding: the bytes are whatever the child program writes. Do not adopt `Out-File -Encoding utf8` until the guard in step 2 has checked its output, because a BOM would fail that check.
2. **Guard every log.** A deterministic guard flags a UTF-16LE or UTF-16BE BOM, a UTF-8 BOM, a NUL byte, and any byte sequence that is not valid UTF-8. It prints one line per offender, `FALLA <reason>: <file>`, and exits non-zero when any offender exists.
3. **Normalise only the files that carry a BOM.** Each conversion is atomic: write a temporary file in the same folder, flush it with `fsync`, reread it and compare the decoded text with the previous copy or with git HEAD, then `os.replace` it over the original. Files without a BOM are never touched.
4. **Run a second pass.** The normaliser must make 0 conversions, and the guard must report 0 offenders.

## Status

- Mitigated for the reported corpus: all 66 offenders were normalised, and the guard reports 0.
- Not closed at the source: a plain redirect still writes UTF-16. The guard detects a recurrence; it does not prevent one.

## Evidence

- 190-file corpus: 66 offenders normalised (57 UTF-16LE, 9 UTF-8 with BOM).
- Text matched git HEAD for 189 of the 190 files; one file had no earlier blob to compare with.
- After normalisation the guard reports 0 offenders, and a second normaliser pass makes 0 conversions.
- The guard's test suite passed 12 of 12 cases.

## What this does not prove

- It is not a controlled study. It is one corpus, one shell version and one repair.
- Other shell versions and terminals may encode redirected output differently; the behaviour above is the observed behaviour of Windows PowerShell 5.1.
- The documentation is inconsistent on this point. The `about_Redirection` reference says the redirection operators use UTF8NoBOM, while the `Out-File` 5.1 reference gives `unicode` as the default. This lesson relies on the observed output and does not resolve the conflict.
- That every offender came from a redirect. The origin of the 9 UTF-8-with-BOM files was not traced.
- NUL bytes without a BOM and invalid UTF-8 are detected but not repaired; a person or the producing tool has to fix them.
