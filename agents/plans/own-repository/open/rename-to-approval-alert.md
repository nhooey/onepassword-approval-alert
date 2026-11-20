---
title: The program is named onepassword-approval-alert throughout
state: proposed
kind: implementation
part-of: own-repository
needs: [move-with-history]
touches: [Package.swift, Sources, Tests, package.nix, check-native-arm64.sh, README.md]
---

# The program is named onepassword-approval-alert throughout

One commit renames everything that carries the old name.

| Today                                                       | Becomes                                                |
| ----------------------------------------------------------- | ------------------------------------------------------ |
| the package and executable `onepassword-dialog-watcher`     | `onepassword-approval-alert`                           |
| the module `OnePasswordDialogWatcherCore`                   | `OnePasswordApprovalAlertCore`                         |
| `Sources/onepassword-dialog-watcher/`                       | `Sources/onepassword-approval-alert/`                  |
| the tests target `onepassword-dialog-watcher-tests`         | `onepassword-approval-alert-tests`                     |
| `pname` and `mainProgram` in `package.nix`                  | `onepassword-approval-alert`                           |
| the usage lines and examples in `helpText`                  | the new name                                           |
| the error prefix `onepassword-dialog-watcher:` on stderr    | `onepassword-approval-alert:`                          |
| the README's title, commands and layout section             | the new name                                           |
| "dialog" in the help, the README and the output lines       | "prompt", the word 1Password and this plan use         |
| the comments `Compromise: plan node onepassword-dialog-watcher` | the id of the node here that records the compromise |

- **Folders move with `git mv`,** so `git log --follow` crosses the rename.
- **The notification group** `1password-approval` and the notification's text keep their values: they never held the old name.
- **The four `Compromise:` comments** point at a plan node that shell-skills doesn't hold at `origin/master`. This commit adds an implemented node here that states the compromise they mean, polling the window list and telling Quick Access by its width, and points the comments at it.

## Checks

- `git grep -i -e dialog-watcher -e DialogWatcher` finds nothing outside `agents/`.
- `onepassword-approval-alert --help` names only the new program, and the option parser's tests pass.
