---
title: The prompt timeout is estimated from the recorded durations
state: proposed
kind: implementation
part-of: outcome-alerts
needs: [prompt-history]
touches: [Sources/OnePasswordDialogWatcherCore/TimeoutEstimator.swift, Tests, README.md]
---

# The prompt timeout is estimated from the recorded durations

1Password ends an unanswered prompt after a fixed time, so timed-out prompts pile up at one duration, and nothing lasts longer.
[Its log on Neil's Mac](../../../reports/2026-11-20-sounds-outcomes-and-timeout.md#what-1passwords-log-shows) shows the pile: 439 of 450 timed-out SSH prompts ended between 59.30 s and 60.00 s.
The estimator finds that pile in the program's own records.

## The input

The [prompt history](prompt-history.md)'s records, newest 200, leaving out:

- **`ended: stopped`:** the program went first, so the duration is cut short.
- **`interrupted: true`:** a sleep stretched it.
- **`outcome: success` decided by a signal other than duration:** an approval is no evidence of the timeout.

Records whose outcome is `unknown`, `failure` or `timeout` all go in, with their label ignored. The estimate never rests on its own earlier verdicts.

## The estimator

A pure function from the durations to an estimate or none:

1. **Windows:** sort the durations. For each duration `d`, count those in `[d − 2.0, d]`: a window 2 s wide, four polls, which holds the measured pile many times over.
2. **Dense:** a window is dense when it holds at least 5 durations.
3. **The ceiling:** take the dense window with the largest `d`. Timeouts are the longest a prompt lasts, so the pile of quick approvals at a few seconds, however dense, sits below it.
4. **Nothing above:** one duration above that window is allowed, as a stray. If more than one lies above it, and they are more than 1 in 10 of all the durations, there is no ceiling in the data, and no estimate.
5. **The estimate** is the median of the window's durations. Its **tolerance** is 1.0 s, or three times the window's median absolute deviation plus half a poll, whichever is larger.

A prompt whose duration is within the tolerance of the estimate closed at the timeout.

## Before there is enough

- **Fewer than 5 durations in any window:** the estimate is the assumed one, 59.6 s with a tolerance of 2.0 s, the figure measured for 1Password 8.12.46. The dry run and the state file's `signal` say `assumed`, not `duration`.
- **`--timeout SECONDS`** sets the estimate and stops the learning, for a 1Password whose timeout is known to differ.
- **Why 5:** the measured pile's deviation is 0.05 s against the 0.5 s poll, so five durations within 2 s of each other at the top of the range are no accident, while one or two long prompts might be a person who was slow.

## When 1Password changes

The estimator runs again each time a prompt ends, over the newest 200 records.

- **A longer timeout** wins once 5 prompts have ended at it: theirs is then the dense window with the largest `d`.
- **A shorter timeout** would leave the old pile above it, and step 3 would keep the old one. So only records of the newest record's 1Password version are used: a new version starts from the assumed estimate and learns its own.
- **A setting changed within one version** is the case this misses, until the old pile leaves the newest 200. `--timeout` covers it.

## Checks

Tests in the plain test executable, on lists of durations:

| Durations                                                                 | Expected                                   |
| ------------------------------------------------------------------------- | ------------------------------------------ |
| none                                                                      | assumed 59.6, tolerance 2.0                |
| four at 59.5                                                              | assumed                                    |
| five at 59.0, 59.5, 59.5, 59.5, 60.0                                      | 59.5, tolerance 1.0                        |
| forty between 1.5 and 6.0, and five between 59.0 and 60.0                 | about 59.5: the quick pile doesn't win     |
| five near 59.5, and one at 180                                            | about 59.5: one stray is allowed           |
| five near 59.5, and three between 100 and 300                             | assumed: no ceiling                        |
| thirty near 59.5, and three between 100 and 300                           | about 59.5: three in thirty-three          |
| five near 59.5 that are `interrupted`, `stopped` or logged successes      | assumed: none of them counts               |
| five near 59.5 at version A, then five near 30.0 at version B             | about 30.0                                 |
| a fixture drawn to the report's figures: 439 between 59.30 and 60.00, 11 under 40 | 59.6 ± 0.1, tolerance 1.0          |

- A property test: shuffling the input never changes the result.
- The classification at the edges: with 59.5 ± 1.0, a prompt of 58.5 s is a timeout and one of 58.0 s isn't.
