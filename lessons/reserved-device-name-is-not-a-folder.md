# Lesson: a reserved device name is not a folder

**Date:** 2026-11 · **Status:** closed for the reported case; the naming rule is a convention, not a guard · **Scope:** test fixtures and temporary folders named after a case, on Windows

**In short.** On Windows, `CON`, `PRN`, `AUX`, `NUL`, `COM1`–`COM9` and `LPT1`–`LPT9` name devices, with or without an extension. A test harness named a temporary folder after its case, `nul`, and writes into that folder were lost or failed. The fix is a naming rule: fixtures and temporary folders take a prefix, and no bare reserved name is used as a file or folder name.

## Error

- A test harness created a temporary folder named after its case: `nul`.
- Writes into that folder were lost or failed.
- The folder had to be renamed to `byte-nul` after the test was already set up to use it.

## Root cause

- Without the `\\?\` prefix, a Win32 path goes through string parsing, which resolves a reserved name to its device. The `\\?\` prefix is the documented way to switch that parsing off.
- Reserved names: `CON`, `PRN`, `AUX`, `NUL`, `COM1`–`COM9`, `LPT1`–`LPT9`. Microsoft's naming rules also cover the superscript forms `COM¹`–`COM³` and `LPT¹`–`LPT³`, which are "reserved in every directory".
- The extension does not change the reserved name: `NUL.txt` is equivalent to `NUL`.
- The case shows the same effect for `nul` used as a folder name.
- `NUL` is the worst case. A write that resolves to it reports no error and discards the data, so the failure appears later, when the expected file is read.
- The folder name came straight from the case label. Nothing rejected a reserved name.

## Action

- Give fixtures and temporary folders a prefix that is not a reserved name, such as `case-…` or `byte-…`. Build the name from the prefix and the case label; never use the bare label.
- Never use a bare reserved name as a file or folder name, with or without an extension.
- The prefix must come first. `byte-nul` is safe; `nul.byte` is not, because the extension leaves the base name reserved.
- The rule applies to any test that names artefacts after the byte or purpose of a case.

## Status

- Closed for the reported case: the folder is `byte-nul`, and the byte-level guard's suite passes 12 of 12.
- The rule is a convention. The evidence reported here includes no check that rejects a reserved name, so a new fixture can repeat the error until one exists.

## Evidence

- After the rename, the byte-level guard's test suite passed 12 of 12 cases.
- The `byte-nul` case exercises the guard's NUL-byte detection.

## What this does not prove

- Other platforms. Linux and macOS accept these names; the rule matters for repositories that are also used on Windows, but nothing was tested elsewhere.
- Extended-length paths. The `\\?\` prefix disables Win32 string parsing, so the behaviour described here may not apply to paths written that way; the case did not test them.
- Every name. The case exercised `nul` only; the other reserved names come from Microsoft's naming page, not from a test.
- That the rename alone fixed the failure. The evidence is one rename followed by a passing suite.
