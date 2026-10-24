# Lesson: a restriction the agent invents is a context trap until its mechanics are written down

**Date:** 2026-10 · **Status:** closed · **Severity:** medium

## What happened

An assistant that runs its own workspace — the hub — added a shell control **on its own initiative**, not because the user asked for one. The intent was correct: keep the hub from executing project work instead of dispatching it. The shape was a base `deny` for the shell tool plus allows scoped to the hub's workspace path.

The rule was written in prose as «commands must contain the workspace path». That sentence is true, and it was not enough. The engine evaluates the **command text, per segment** (segments split by `;`, `|`, `&&`) and ignores the working directory. A command that chains two segments is denied if **any** segment fails to name the workspace:

```
echo <workspace>                       -> allowed
cd <workspace>                         -> allowed
echo <workspace>; echo <workspace>     -> allowed
cd <workspace>; date                   -> denied   (second segment names nothing)
<interpreter> --version                -> denied   (names nothing)
```

The assistant hit the fourth case, read `Permission denied: shell` as «the shell is broken», stopped using the tool, analysed a 41-second video through a browser with no audio and no transcription, and reported to the user that **no** shell command was permitted in that session. It had been permitted the whole time: `echo <workspace>` passed.

## Root cause

A control the agent imposes on itself is only as good as its written mechanics. Three forces combined:

1. **The rule stated intent, not behaviour.** «Commands must contain the path» describes what the author wanted, not how the engine decides. Every reader — including the author in a later session — fills the gap with the most convenient interpretation.
2. **The rule was incomplete in the safe direction**, which is the hardest kind to notice. It was not false; it was silent about the one detail that mattered (every segment, not any segment). A wrong rule gets caught; an under-specified one reads as a broken tool.
3. **The author was also the operator.** There was no second reader to ask «how does this actually work?», so the first denial was attributed to the tool by default.

The cost is asymmetric: the agent loses capability *and* believes it is being careful. A restriction without evidence behaves exactly like a bug, and the review that would catch a bug does not run on a rule the author wrote for themselves.

## Fix applied in the machine

- **The rule now carries its mechanics**, dated, with the commands that pass and the commands that fail, and an explicit line that the working directory is not part of the decision.
- **A house rule covers the next one:** any restriction the agent imposes on itself must carry verified mechanics, a date and an owner. Restrictions the user did not ask for are **revocable** and are reviewed when they get in the way. An invented rule is a hypothesis, not a law.
- **A triage rule for denials:** before concluding that a tool is unavailable, rewrite the legitimate command so it satisfies the rule and retry. Only a command that already satisfies the rule and is still denied is the guard.
- **The obvious fix was rejected on evidence.** Loosening the control (allow the shell generally) would have removed the only part that was actually enforced: writes outside the workspace were blocked by a separate `edit` denial, and a general shell lets a script write there without any rule seeing it. A perimeter is not an isolation boundary; relaxing the perimeter does not buy capability, it removes the wall behind it.
- **The distinction is now written down** next to the rule: a perimeter stops execution from *pointing* at a foreign path without naming it; it does not inspect what a program does once it runs.

## Evidence

- The six commands above were run in the same session, in order, with the same working directory; four allowed, two denied. Dated.
- The video analysis produced in the same session has no transcript: the audio path was never attempted, because the tool was believed unavailable.
- The incorrect report to the user («no shell command is permitted») was contradicted by a single passing command minutes later, and corrected in the same conversation.
- The written rule was changed the same day; the mechanism, the pass/fail list and the perimeter/isolation distinction now live beside the permission entries, not in a separate document.

## Transferable rule

> A rule you wrote for yourself is code. If its mechanics are not written and tested, it will be read as a broken tool — and the agent will lose capability while believing it is being safe.
> Document **how** the rule applies, not what it is for. And when the rule denies something legitimate, rewrite the command before declaring the tool dead.
