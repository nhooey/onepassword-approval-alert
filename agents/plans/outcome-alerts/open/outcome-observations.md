---
title: What each outcome looks like in the window list is observed, not guessed
state: proposed
kind: implementation
part-of: outcome-alerts
touches: [agents/reports]
---

# What each outcome looks like in the window list is observed, not guessed

The [report](../../../reports/2026-11-20-sounds-outcomes-and-timeout.md#known-and-guessed) marks seven statements about failed and timed-out prompts as guessed.
This node turns each into a known fact, in one sitting with Neil at the keyboard, and writes a new dated report.

## The sitting

`--dry-run` already prints every change to a 1Password window, with a timestamp.
With it running and its output saved, Neil does each of these, for an SSH signature and for an `op` read:

| Case                   | What Neil does                                             |
| ---------------------- | ---------------------------------------------------------- |
| Approve                | Touch ID, at once                                          |
| Cancel                 | press Escape or Cancel                                     |
| Failed Touch ID        | a wrong finger, three times                                |
| Wrong password         | choose the password, type a wrong one                      |
| Timeout                | leave the prompt alone                                     |
| Locked app             | lock 1Password first, then trigger the prompt              |

After each, the matching lines of 1Password's log are copied beside the dry run's lines.

## What it settles

- **Whether the window closes on a timeout,** and how long after the log's line.
- **Whether a failed attempt changes the window:** its size, its layer, a second window.
- **Whether a prompt reappears** after a failure or a cancel, how soon, and with which window id and size.
- **Which log lines mark a cancel, a failed attempt and a wrong password,** and whether an `op` prompt logs as an SSH prompt does.
- **Whether a launchd agent can read the log folder** without a privacy prompt: the same read, run from a throwaway launchd job.
- **What titles the prompts have,** if Neil chooses to grant Screen Recording permission for the sitting. The program still never asks for it.

## Checks

- A report `agents/reports/<date>-prompt-outcomes-observed.md` holds a table of case against window changes and log lines, and marks each of the seven guesses confirmed or refuted.
- The [outcome detection](outcome-detection.md) node's rules are edited to match before it is approved.
