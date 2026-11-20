---
title: The program tells success, failure and timeout from what it can observe
state: proposed
kind: implementation
part-of: outcome-alerts
needs: [timeout-estimator, outcome-observations, alert-configuration]
touches: [Sources/OnePasswordDialogWatcherCore/Outcome.swift, Sources/OnePasswordDialogWatcherCore/Alerter.swift, Sources/OnePasswordDialogWatcherCore/LogSignals.swift, Sources/onepassword-dialog-watcher/main.swift, Tests, README.md]
---

# The program tells success, failure and timeout from what it can observe

What is known and what is guessed is set out in [the report](../../../reports/2026-11-20-sounds-outcomes-and-timeout.md#how-a-prompt-ends).
The rules below rest on some of the guesses, each marked, and the [outcome observations](outcome-observations.md) confirm or change them before this node is approved.

## What the program can observe

| Signal                    | From                                             | Tells                                                    | Standing |
| ------------------------- | ------------------------------------------------ | -------------------------------------------------------- | -------- |
| How long the prompt was up | the window list, twice a second                 | a timeout, against the estimate                          | known    |
| How it closed             | the window went, or the program was stopped      | whether the duration is whole                            | known    |
| A change of size          | the window's bounds between samples              | a move from Touch ID to a password field, perhaps        | guessed  |
| The same prompt again     | a new window of the same size within 3 s         | a client retrying after a failure, perhaps               | guessed  |
| 1Password's log           | `1Password_rCURRENT.log`, read from where it ended at the last sample | a refusal or a timeout of an SSH prompt; nothing for an approval | known for SSH, guessed for `op` |
| The title                 | the window list, only with Screen Recording permission | the kind of request, perhaps                        | guessed  |

The program asks for no permission. A title is used only when it happens to be readable, and the log only when the file can be opened.

## The rules

Decided once, when a prompt's window is gone from a sample, in this order:

1. **Stopped:** the program is stopping. No outcome, no sound; the record says `ended: stopped`.
2. **Timeout:** the duration is within the tolerance of the [timeout estimate](timeout-estimator.md), or the log gained `ssh authorization prompt timed out` or `AppCancel invoked by a timed-out prompt` since the prompt appeared.
3. **Failure, by the log:** the log gained `Session was not authorized`, `received error from SSH auth prompt` or `prompt error response from CLI rich auth prompt` since the prompt appeared, and rule 2 didn't match.
4. **Failure, by what follows:** a prompt of the same size appears within 3 s. The verdict waits those 3 s, and the new prompt plays the failure sound in place of the prompt sound, once, so a retry isn't heard as a new request. Guessed.
5. **Success:** none of the above, and the log was readable. An approval is the one ending the log is silent about.
6. **Unknown:** none of the above, and the log wasn't readable. The window list can't tell an approval from a cancel: both close the prompt early. It plays the success sound, since most prompts that close early were approved, and records `unknown`.

A failed attempt that leaves the prompt up, a wrong finger or a wrong password, ends nothing, so it plays nothing. If the observations show the window changes size at that moment, a seventh rule plays the failure sound on the change, once per prompt.

## A decision for Neil: is a cancel a failure

Rule 3 calls a cancel a failure, since `Session was not authorized` is all the log says of either.
A cancel is Neil's own act and may not want a sound at all.
If the observations show the log tells the two apart, a cancel plays nothing.

## Several prompts at once

Each window id has its own start, record and verdict.
Log lines carry no window id, so with two prompts up, a line is given to the one that closes at the next sample, and to the older when both do.

## In the code

- **`Outcome.swift`:** the outcome type and one pure function from a prompt's facts, its duration, how it ended, whether it resized, what followed, the log lines seen, the estimate, to an outcome and the signal that decided it.
- **`LogSignals.swift`:** a pure function from new log text to the lines above, and in the shell a reader that remembers its offset, follows the file when 1Password rotates it, and gives up quietly, once, with one line on stderr, when it can't open it. `--log-file FILE` names another log and `--no-log` reads none.
- **`Alerter`:** holds each prompt until its verdict, then plays the outcome's sound and appends the [record](prompt-history.md).

## Checks

- The pure function's tests cover every rule, their order where two match, and each signal absent.
- The alerter's tests, with a stubbed clock: a prompt closing at 59.5 s plays the timeout sound; one closing at 4 s with a readable, silent log plays success; the same with an unreadable log is `unknown`; one followed at 2 s by a same-size prompt plays failure and no prompt sound; one up at `stop()` plays nothing.
- The log reader's tests: a rotation mid-prompt, a missing file, a line split across two reads.
- With the real program in `--dry-run`, Neil's sitting from the observations is repeated, and each case prints the outcome he produced.
