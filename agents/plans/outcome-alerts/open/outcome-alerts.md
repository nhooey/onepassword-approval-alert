---
title: Each 1Password prompt sounds when it appears and again when it ends, by outcome
state: proposed
kind: end-state
---

# Each 1Password prompt sounds when it appears and again when it ends, by outcome

When this is done:

- **Four alerts:** a prompt that appears plays the prompt sound. When it ends, the program plays the success sound, the failure sound or the timeout sound, one of the three, once.
- **A pair by ear:** the prompt sound is two beats, low then high, and the success sound is its inverse, high then low, so an approval is heard as the prompt answered.
- **Each sound is a flag:** the four have defaults, and `--prompt-sound`, `--success-sound`, `--failure-sound` and `--timeout-sound` each replace one.
- **The timeout is learned:** the program keeps each prompt's duration and how it ended in a state file, across runs, and from them estimates how long 1Password leaves a prompt up. A prompt that closes near that estimate is a timeout.
- **A dry run shows the reasoning:** `--dry-run` prints, for each prompt that ends, its duration, the outcome, the signal that decided it and the timeout estimate in force.
- **What it can't know, it says:** an outcome the program can't tell is reported as `unknown` in the dry run and the state file, and plays the sound the [outcome detection](outcome-detection.md) node names for it.

The research is in [the report](../../../reports/2026-11-20-sounds-outcomes-and-timeout.md).

This plan doesn't wait for the [move to this repository](../../own-repository/open/own-repository.md).
Built before the move, the same changes land in `commands/onepassword-dialog-watcher/` in nhooey/shell-skills, and the file paths in these nodes' `touches` are relative to the program's folder either way.
