---
title: shell-skills no longer holds the program
state: proposed
kind: implementation
part-of: own-repository
needs: [launchd-agent-module]
touches: [shell-skills commands/onepassword-dialog-watcher, shell-skills flake.nix, shell-skills README.md]
---

# shell-skills no longer holds the program

Once the agent from this repository is what runs on Neil's Mac, one pull request in nhooey/shell-skills removes its copy.

- **Removed:** `commands/onepassword-dialog-watcher/`, and in `flake.nix` the `onepassword-dialog-watcher` package and the `onepassword-dialog-watcher-native-arm64` check, with any README line that names the program.
- **Recorded:** the commit message names nhooey/onepassword-approval-alert as where the program went, and the last shell-skills commit that held it, so the retired copy can be found.
- **Skills that mention it:** any skill in shell-skills or elsewhere that tells an agent to run `nix run .#onepassword-dialog-watcher` is changed to name the new program. `git grep` over the skill repositories finds them.

It needs the module, not just the move, so no day passes with no working alert.

## Checks

- `nix flake check` passes in shell-skills.
- `git grep -i onepassword-dialog-watcher` in shell-skills finds nothing.
