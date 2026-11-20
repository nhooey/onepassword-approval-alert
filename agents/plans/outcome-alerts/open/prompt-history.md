---
title: Each ended prompt is recorded in a small state file that outlives the run
state: proposed
kind: implementation
part-of: outcome-alerts
touches: [Sources/OnePasswordDialogWatcherCore/PromptHistory.swift, Sources/OnePasswordDialogWatcherCore/Alerter.swift, Sources/OnePasswordDialogWatcherCore/Options.swift, Tests, README.md]
---

# Each ended prompt is recorded in a small state file that outlives the run

## Where

`${XDG_STATE_HOME:-~/.local/state}/onepassword-approval-alert/prompts.jsonl`, created with its folder, mode `u=rw`.
`--state-file FILE` names another, and `--no-state` keeps nothing and estimates from nothing.

## What a line holds

One JSON object a line, appended when a prompt ends:

```json
{"v":1,"started":"2026-11-20T09:37:06.5Z","duration":59.5,"ended":"closed","outcome":"timeout","signal":"duration","interrupted":false,"width":420,"height":260,"resized":false,"reappeared_after":null,"app":"8.12.46"}
```

| Field              | Holds                                                                                                     |
| ------------------ | --------------------------------------------------------------------------------------------------------- |
| `v`                | the format's version, 1                                                                                   |
| `started`          | when the window was first seen, in UTC                                                                    |
| `duration`         | seconds from the first sample that held the window to the first that didn't, to 0.1 s                     |
| `ended`            | `closed`, the window went; or `stopped`, the program was stopped while it was up                          |
| `outcome`          | `success`, `failure`, `timeout` or `unknown`, as the [outcome detection](outcome-detection.md) decided    |
| `signal`           | what decided it: `duration`, `assumed`, `log`, `reappeared`, `resized` or `none`                          |
| `interrupted`      | true when two samples in the prompt's life were more than 2 s apart: the Mac slept or the program stalled |
| `width`, `height`  | the window's size in points when first seen                                                               |
| `resized`          | whether its size changed while it was up                                                                  |
| `reappeared_after` | seconds until the next prompt appeared, when under 10 s, else null; written by rewriting the last line    |
| `app`              | 1Password's version, from its bundle, so a changed timeout can be told from a changed version             |

- **No title, no requester, no secret:** nothing in the file says what was approved or for whom.
- **Duration's precision** is the poll's: half a second. A duration is measured on the monotonic clock, so a clock change doesn't bend it.

## Keeping it small

- **At start** the program reads the file and, when it holds more than 500 lines, rewrites it with the newest 500, to a temporary file in the same folder, then renames it over.
- **A line it can't read** is skipped and counted, and the count is printed once on stderr. A last line cut short by a crash is such a line.
- **Two copies running** append whole lines of under 512 bytes with one `write`, so they don't tear each other's lines; the trim at start takes an advisory lock on the file.

## In the code

`PromptHistory.swift` in the core: the record type, pure functions to encode and decode a line and to trim a list, and a small type that reads and appends, with the file's path passed in. The alerter gets it as it gets its effects, so tests pass one in memory.

## Checks

- Encode then decode gives the record back, for each outcome and with null fields.
- A file with a cut last line, an unknown `v` and a blank line loads the good lines and reports three skipped.
- 600 lines trim to the newest 500, and the file is never absent during the trim.
- A prompt that is up when SIGTERM arrives is written with `ended: stopped` before the program exits.
- `--no-state` writes nothing: the test's folder stays empty.
