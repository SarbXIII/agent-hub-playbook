# Lesson: a dead client is not a finished task

**Date:** 2026-10 · **Status:** mitigated; two gaps open (see Status) · **Scope:** watchers that report when a long agent turn ends

**In short.** A watcher declared a task finished because its log had gone quiet and the client process had exited. The model turn ran on the application server, not in the client. The client's death stopped the log, not the task. A client-side signal can prove that the client is gone. It cannot prove that the task ended.

## Error

The watcher sent a notice, translated here, reading "process ended; no CHECK visible". The log held one line before it:

```
Transport: ECONNRESET: The socket connection was closed unexpectedly
```

The session store still showed the session busy: a busy-since timestamp, no idle timestamp, and new messages after the notice. A report file was written while the session kept working on its checklist.

## Root cause

The watcher used two proxies: the tail of the log and the process ID of the client.

- A dead client stops writing. A quiet log and a missing PID look exactly like a finished task.
- The turn runs on the server. Losing the client cuts the output; it does not stop the turn.
- The watcher declared the task finished 8 seconds after the client died. The grace period exists so the client can flush its last log text.
- An earlier fix covered a client killed by hand. It did not cover a client that died on its own. That is the same root cause, and it produced the same false finish.
- An output file can exist while the turn still runs. A file is not proof of completion either.

Mechanism in one line: when the client dies, the observable surface (log and PID) disappears while the task keeps running on the server, so both proxies read calm.

## Action

The watcher now decides in this order:

1. **Session state first.** A session that is busy since a timestamp and has no idle time is not finished, whatever the log or the PID say.
2. **Idle means finished,** with or without a CHECK line. The session state decides the finish; the CHECK line, when present, supplies the text of the notice.
3. **Proxies only when no state can be read.** Then the log and the PID decide, with one rule added: a transport error (connection reset, socket closed, transport prefix) means "client lost", not "task done". The later idle notice says the turn may still be running on the server.
4. **Session id without the recipient.** When the id is taken from the log, exclude the id of the hub that receives the notice. Otherwise the watcher watches itself.
5. **Read the store, not the client's command line.** The state reader reads the session store directly. A listing script that depended on the client's command line broke when that command moved.

## Status

Mitigated. The state-first order and the transport rule are in the watcher. Two gaps remain. Both come from reading the code; neither was run as a test.

- **No-state fallback still closes on proxies.** With the PID gone and no transport message, the watcher sends the "process ended" notice and exits once it is delivered. With a transport message, it logs the event. Once the log has been quiet for the idle period, it sends a warning and exits if the client is still dead. The transport rule prevents the false finish; it does not keep the watcher alive. Pending action: keep watching, or report "state unknown", until the state can be read.
- **A CHECK line while the session is busy ends the watch on the first tick.** The grace timer starts at zero, so the sixty-second wait is skipped. If the notice is delivered, the watcher exits at once. Pending action: start the timer when the line is first seen, and keep watching while the session is busy.

## Evidence

- A busy session with zero minutes idle, watched by the new code: still alive at 38 seconds. The previous code closed that session.
- A finished session: the watcher exited at 12 seconds.
- The changed script passed a syntax check.
- One measured case, in one environment. Timings vary between runs.

## Promoted to

None in this repository. The watcher lives in a private hub. The generic rule is stated here; the two open gaps are not closed by it.

## What this does not prove

- That the completion notice reaches the hub. The tests pointed at a hub that does not exist, so delivery is untested.
- That every runner exposes session state in a readable store. Each runner and client version needs its own check before the state reader is trusted.
- That the two open gaps are rare. Code reading shows they can occur. No run has counted how often.

## Checks to repeat

- Busy session with no idle time and a dead client: the watcher must still be running at 38 seconds, the point of the measured case, and must send no completion notice.
- Finished session: the watcher must exit with the outcome read from the session state.
- Transport error with no state available: the watcher must log that the client was lost to a transport error and must not send a "task done" notice.
- CHECK line while the session is busy: the watcher must keep watching until the session is idle. This check is expected to fail, from code reading; it has not been run. See Status.
