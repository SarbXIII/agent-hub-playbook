# Lesson: announced skills are a cold-start tax — measure it and scope it

**Date:** 2026-10 · **Status:** closed

## What happened

A subagent was created to answer two short questions. It cost **$0.11** before producing anything useful. The questions were not the problem: the first model call spent **$0.095** writing the cached prefix.

The prefix was made of the announced skills plus the global instruction file:

- **186 announced skills ≈ 31,500 tokens** (their descriptions are always visible to the model).
- Global instructions ≈ 6,000 tokens.
- Total cold start ≈ 37,800 tokens.

The chosen model for that agent had the most expensive cache write of the provider's catalogue (**$2.50 per million tokens**), so the first call paid ~$0.095 in cache writes alone. Every later turn looked cheap because the prefix was already cached.

## Root cause

Three factors multiplied:

1. **Skill descriptions are injected into every request**, not only when a skill is used. More installed skills means a bigger fixed prefix.
2. **Subagents start cold** (fresh context), so they pay the full cache write every time they are spawned — unlike a long conversation, where the prefix is warmed once.
3. **Model choice ignored the cache-write price.** The model had been picked on perceived quality, not on the actual cost structure of a cold-start agent.

## Fix applied in the machine

- Every agent declares a **skill allowlist**: `skill: deny *` followed by the role's skills. A denied skill is hidden from the listing and cannot be loaded. Measured result: cold start fell from ~37,800 to **~11,700 tokens** for the explorer.
- Niche skills are marked `autoinvoke: false` where the host supports it: hidden from the listing, still loadable by exact ID.
- Subagent models are chosen with the cold-start economics in mind: cache-write price matters more than the per-token price when the agent is spawned repeatedly.
- The hub keeps the full catalogue on purpose (it is a router: the list is its map), but the role agents do not.

## Evidence

- Token counters on the test session: first call ≈37.8k cache-write tokens, cost ≈$0.095; cache reads on the second call ≈37.8k.
- After scoping: explorer cold start ≈11.7k input tokens, total session cost ≈$0.002.
- All five role agents verified with their own allowlists; the reviewer's startup no longer carries the design or marketing catalogues.

## Transferable rule

> The catalogue is not free: you pay for every description on every cold start.
> Scope skills per role, measure the prefix, and price the cache write — not just the token.
