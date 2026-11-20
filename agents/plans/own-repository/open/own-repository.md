---
title: The approval alert lives in its own repository, under its new name
state: proposed
kind: end-state
---

# The approval alert lives in its own repository, under its new name

Nothing here is done now: Neil has decided the move, and that it waits.

When this is done:

- **The repository:** nhooey/onepassword-approval-alert on GitHub holds the Swift program at its root, with the history of `commands/onepassword-dialog-watcher/` from nhooey/shell-skills. `git log --follow Package.swift` reaches 0babd60 "Sound an alert only while a 1Password approval dialog is up".
- **The name:** the program, its package, its `--help`, its error prefix and its README say `onepassword-approval-alert`. `git grep -i dialog-watcher` finds nothing outside this plan's closed nodes and the reports.
- **The agent:** the flake's nix-darwin module runs the program as a launchd user agent. With the module enabled, `launchctl print gui/$(id -u)/org.nixos.onepassword-approval-alert` shows it running after login.
- **shell-skills:** no longer builds or holds the program, and its commit that removes it names this repository.

The program's state today is in [the report](../../../reports/2026-11-20-sounds-outcomes-and-timeout.md#the-program-today).
