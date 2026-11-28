# Lesson: a lesson log without a promotion step is a dead file

**Date:** 2026-11 · **Status:** mitigated by rule; no mechanical check · **Scope:** agent lessons, dated lesson logs and role brains in a multi-agent setup

**In short.** A lesson log that nothing promotes is a dead file: its lessons never reach the rules that agents read. The rule "the agent records its own lesson" could not run either, because a subagent cannot edit its own role brain. The child now reports the lesson, and the parent records and promotes it.

## Error

- The lesson log held entries from five dates, while the role brain still carried five entries from a month earlier and was dated before the backfill.
- A child subagent tried to edit its role brain and got `Permission denied: edit`. It reported the denial instead of forcing a variant.

## Root cause

- Promotion was a step the parent remembered, not part of closing a block. Nothing in the closing forced it, so the log grew and the brain did not change.
- Subagents are confined outside their workspace and end when they close. They cannot edit configuration, including their own role brain, so a rule that asks them to record their lesson cannot be executed.

## Action

- The child declares its lesson in a dedicated field of its final response: error, cause and action.
- The parent writes the lesson to the dated lesson log.
- The parent promotes it to the role brain only if the behaviour changes.
- Promotion is a step of block closing, not a reminder.

## Status

Mitigated by rule. The promotion step has no mechanical check: it depends on the parent's judgement.

## Evidence

- The role brain is dated before the backfill of the lesson log.
- A grep of the agent configuration finds no edit path outside the workspace.
- The diff that changed the rule.

## What this does not prove

- That promotion always happens. It depends on judgement at closing time.
- That the rule will hold. It has no mechanical check, so it can decay the way the earlier rule did.
- That the pattern generalises. It was measured in one ecosystem only.
