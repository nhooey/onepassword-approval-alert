---
title: Five sound flags replace --first-sound and --second-sound
state: proposed
kind: implementation
part-of: outcome-alerts
needs: [default-sounds]
touches: [Sources/OnePasswordDialogWatcherCore/Options.swift, Sources/OnePasswordDialogWatcherCore/Alerter.swift, Sources/OnePasswordDialogWatcherCore/ProcessEffects.swift, Tests, README.md]
---

# Five sound flags replace --first-sound and --second-sound

## The flags

| Flag                    | Plays when                         | Default                                |
| ----------------------- | ---------------------------------- | -------------------------------------- |
| `--prompt-sound FILE`   | a prompt appears                   | the [default sounds](default-sounds.md) |
| `--success-sound FILE`  | a prompt ends approved             | the same                               |
| `--failure-sound FILE`  | a prompt ends refused or failed, first or third onwards in a row | the same |
| `--second-failure-sound FILE` | the second failure in a row     | the same                               |
| `--timeout-sound FILE`  | a prompt ends at the timeout       | the same                               |

- **Each works as `--first-sound` does today:** it takes a file path, an empty or missing value is a usage error naming the flag, and a file that can't be read stops the start with exit 2.
- **`--first-sound` and `--second-sound` go.** They are unknown arguments afterwards, with the usual usage error. The program has one user and no release, so nothing keeps the old names.
- **`--help`** lists the five with their default paths, and describes when each plays.

## The still-open alert goes

Today a prompt still open after 60 s plays the second sound.
1Password ends an unanswered prompt at 59.6 s to 60.0 s, [measured from its log](../../../reports/2026-11-20-sounds-outcomes-and-timeout.md#what-1passwords-log-shows), so that alert fires as the prompt closes, or not at all.
The timeout sound takes its place, and the second notification with it.

If Neil wants a reminder while there is still time to answer, it is a fifth alert at a share of the estimated timeout, and a node of its own.

## In the code

- **`Options`** holds `promptSound`, `successSound`, `failureSound`, `secondFailureSound` and `timeoutSound`, and the alerter counts failures in a row, reset by a success or a timeout.
- **`AlertEffect.playSound`** carries one of five cases: `prompt`, `success`, `failure`, `secondFailure`, `timeout`. `AlertCommands` holds the five paths.
- **The notification** is sent when a prompt appears and removed when none is left, as today. An outcome sends none: the sound is the message.
- **A sound still playing** when the next one starts is stopped first, so an outcome is never heard over the prompt sound.

## Checks

- The option parser's tests cover each flag, its missing value, and the two removed flags as unknown arguments.
- The alerter's tests cover each outcome playing its own sound once, with the effects stubbed as today.
- `--dry-run` prints `alert sound success (prompt approved after 4.5 s)` and its like, and plays nothing.
