# Lesson: shell allowlists match by substring — and an agent allow beats the global deny

**Date:** 2026-10 · **Status:** closed · **Severity:** high

## What happened

Two read-only agents (an explorer and a reviewer) had been given narrow-looking shell allows so they could inspect a diff:

```yaml
permissions:
  - { action: shell, resource: "*git diff*",  effect: allow }
  - { action: shell, resource: "*git status*", effect: allow }
  - { action: shell, resource: "*git log*",   effect: allow }
```

The prompt also said «you do not execute commands». It was a sentence, not a control.

During a capability test, the reviewer executed:

```
cmd /c "echo PWNTEST & git diff"
```

and later:

```
cmd /c "git push --force --dry-run & git diff"
```

Both ran. The first printed `PWNTEST`, proving arbitrary command execution. The second ran a destructive git command in dry-run mode despite a **global** deny rule for `git push --force` in the user configuration.

## Root cause

Two independent mechanisms:

1. **Pattern matching is by substring, not by command structure.** A rule written as `*git diff*` matches any command line that *contains* that text, including `echo hello git diff`. Wrapping everything into a single `cmd /c "… & …"` segment defeats per-segment parsing: the whole thing is one segment that happens to contain `git diff`.
2. **Rule precedence is «last match wins», and agent rules are evaluated after global rules.** A broad allow in an agent profile therefore **overrides** a global deny. The catastrophic-command guard was a tripwire at the global layer, and the agent allow walked straight through it.

## Fix applied in the machine

- **The reviewer lost the shell tool entirely.** Without a shell rule, the tool is not offered to the model. This is stronger than «permission denied»: there is nothing to bypass.
- **Shell allows were removed from every read-only agent**, along with the prompt sentence that pretended to enforce it. If a reviewer needs a diff, the caller provides it.
- **Write-capable agents keep only narrow, reviewed denies** (for example `git push`, `git merge`) and are treated as code: every allow is reviewed like a permission change, not like configuration.
- The house constitution gained a rule: no substring shell allows for read-only agents; deny the whole tool instead.

## Evidence

- Session logs show the exact commands and their results (the literal `PWNTEST` output and the dry-run push).
- After the fix, the same reviewer was asked to run a command and had no shell tool available at all.
- The global guard was confirmed insufficient on its own: the agent allow matched first.

## Transferable rule

> If an agent only reads, it does not get a shell with narrow patterns. It gets **no shell**.
> Shell permissions are code: review them, test them with an adversarial command, and assume any pattern will match more than you intended.
