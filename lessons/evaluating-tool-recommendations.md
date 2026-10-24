# Lesson: a recommendation list is not an adoption plan

**Date:** 2026-10 · **Status:** closed

## What happened

A 41-second video recommended five "free" plugins to take a coding agent "from junior to senior". Every one of them existed and was actively maintained — verified against the public code-host API, not against the video, which is the first thing a recommendation never gives you.

Scored against the hub's real gaps, the list collapsed:

- **one** was worth trying (an up-to-date-documentation server, because stale API knowledge is the most common junior failure);
- **two** were conditional (a browser-automation server and a pentesting agent: useful the day a project has end-to-end tests or a public surface to audit, useless before that);
- **two** were rejected — and they were the most popular entries in the list.

The most-starred item was a memory plugin the hub had already parked months earlier: it duplicates a memory the hub already owns (a memory file, a lessons book and per-project notes), and it was known to break on the current agent host. Installing it would have created a second source of truth, which is worse than having none.

The video's own metrics pointed the same way: a few dozen likes, almost as many saves, and a single comment. The audience was filing the list for later — not acting on it, and certainly not buying the "junior to senior" promise.

## Root cause

Three forces push a recommendation straight into an install:

1. **Novelty plus creator authority.** A confident list from someone who clearly uses the tools reads as a decision already made by someone else.
2. **"Free" conflates licence with cost.** Open-source code still asks for an account, an API key, model tokens per run, an update treadmill, and a new failure surface inside the agent.
3. **Popularity measures attention, not fit.** Star counts and saving behaviour tell you the list was interesting. They tell you nothing about whether the problem exists in your hub.

## Fix applied in the machine

Four gates, in order, before anything recommended by a third party is installed:

1. **Does the problem exist here?** Name the concrete failure it prevents. If you cannot name one you have actually had, stop — the other three gates are irrelevant.
2. **Is it already covered?** Check the existing tool, skill or file that does the same job. A second tool that overlaps is a second source of truth.
3. **What does it really cost?** Licence *and* service: account, key, tokens per run, background processes, update cadence, and how it fails when it fails.
4. **What would reopen the decision?** Write the condition. A verdict without a reopening condition becomes permanent by accident.

The verdict is written down, dated, as **TRY / WATCH / REJECT** with the gate that decided it, and it is backed by data fetched from the source of truth (repository API, changelog, release) rather than from the recommendation itself.

Two smaller rules came out of the same case:

- **Verdicts are about the gap, not the tool.** A well-maintained project with a hundred thousand stars can still be the wrong answer, and saying so is not a criticism of the project.
- **Read the format's real signal before choosing a call to action.** Saves mean "file this for later"; comments mean "give me a reason to reply". A keyword-comment call to action on a list people are quietly saving converts nobody.

## Evidence

| Item | Popularity | Verdict | Deciding gate |
|---|---|---|---|
| Up-to-date documentation server | High | **TRY** | Gate 1: stale APIs are a recurring, named failure |
| Memory plugin | Highest in the list | **REJECT** | Gate 2: duplicates existing memory; known host breakage |
| Browser-automation server | High | **WATCH** | Gate 1: no project yet has tests to maintain |
| UI component generator | Lower, least active | **REJECT** | Gate 3: produces the generic aesthetic the design rules exist to avoid |
| Pentesting agent | High | **WATCH** | Gate 1/3: needs a public surface and a runtime to audit |

The two rejections were the two most popular entries. That is the whole point.

## Transferable rule

> Ask what is missing before asking what exists.
> A recommendation is evidence that a tool works, never evidence that you need it — and the most popular item on the list is the one to check first.
