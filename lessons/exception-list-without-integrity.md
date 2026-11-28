# Lesson: an exception list without integrity can grow in silence

**Date:** 2026-11 · **Status:** mitigated for the reported guard; the rule covers every exception list, and no audit of the other lists is recorded · **Scope:** plain-text allowlists and baselines of exceptions that a guard reads

**In short.** An encoding guard kept its exception list as a plain file. Its expected count was checked only by a test that the normal run does not execute. Adding one artefact path made the guard print `WARN` and exit with code 0: the check stayed green while its exceptions had grown. An exception list needs the same protection as the check that reads it.

## Error

- The encoding guard, which a reviewer had accepted, kept its baseline as a loose file.
- The figure of 26 paths lived only in a test that the normal run does not execute.
- Adding the path of a new artefact to the file printed `WARN` and exited with code 0.

## Root cause

The guard demanded an exception list but did not protect it. The normal run never compared the list with its expected count, and nothing pinned its content, so a change to the list did not show up as a failure. This is a criterion that requires the list without protecting it. Any allowlist or baseline kept as plain text has the same shape.

## Action

Every allowlist or baseline of exceptions needs mechanical integrity:

- **Count and hash in code.** The expected entry count and the sha256 of the file are constants in the guard's code. They change only in a diff that a reviewer can see.
- **Checked in the normal pass.** The count and the hash are compared on every normal run, not only in a test that the run skips.
- **Tested for FAIL.** A test alters the list and requires FAIL in each case: an extra line (27 entries against the pinned 26), and an edited path (a different sha256, with the full hash printed in the message).

## Status

Mitigated for the reported guard. The rule covers every exception list; no audit of the other lists is recorded here.

Known risk: without a `.gitattributes` file, a clone with `core.autocrlf=true` rewrites CRLF line endings, and the sha256 changes. The failure is loud, not silent: the guard reports a mismatch.

## Evidence

- A FAIL message reports the count mismatch: 26 pinned, 27 found after one extra line.
- A FAIL message reports a different sha256 after one path is edited, and prints the full hash.

## What this does not prove

- That the hash fixes intent. It fixes content: code and baseline can change together in one commit, and the pinned values change with them.
- That the protection reaches anything the guard does not watch. It covers only the watched artefacts.
