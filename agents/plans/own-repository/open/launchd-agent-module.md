---
title: The flake builds the program and offers a nix-darwin launchd agent module
state: proposed
kind: implementation
part-of: own-repository
needs: [rename-to-approval-alert]
touches: [flake.nix, flake.lock, package.nix, modules/darwin.nix, README.md, AGENTS.md]
---

# The flake builds the program and offers a nix-darwin launchd agent module

## The flake

- **Outputs:** `packages.<system>.default` and `.onepassword-approval-alert` from `package.nix`, for `aarch64-darwin` and `x86_64-darwin`; `darwinModules.default`; `checks` holding the package, whose build runs the tests, and the native-arm64 check that shell-skills' flake runs today; a formatter.
- **Conventions:** those of the `nix-flakes` skill, as shell-skills' flake follows them.

## The module

`modules/darwin.nix`, exported as `darwinModules.default`:

| Option                                            | Type            | Default                 |
| ------------------------------------------------- | --------------- | ----------------------- |
| `services.onepassword-approval-alert.enable`      | boolean         | `false`                 |
| `services.onepassword-approval-alert.package`     | package         | this flake's package    |
| `services.onepassword-approval-alert.sounds.prompt`, `.success`, `.failure`, `.timeout` | null or path | `null`, the program's default |
| `services.onepassword-approval-alert.extraArgs`   | list of strings | `[ ]`                   |
| `services.onepassword-approval-alert.logFile`     | null or string  | `null`                  |

When enabled it sets `launchd.user.agents.onepassword-approval-alert`:

- **`ProgramArguments`:** the package's program, a `--<name>-sound` flag for each sound that is set, then `extraArgs`.
- **`RunAtLoad` and `KeepAlive`:** both true, so it starts at login and restarts if it dies. The program exits 0 only on SIGINT or SIGTERM.
- **`LimitLoadToSessionType = "Aqua"`:** it needs the window server, which only a graphical login has.
- **`ProcessType = "Interactive"`:** its half-second timer must not be throttled as a background job's is.
- **`StandardOutPath` and `StandardErrorPath`:** `logFile` when set, else unset.

The sound options match the flags of the [alert configuration](../../outcome-alerts/open/alert-configuration.md). If this node is built before that one, it offers the two sounds the program has then.

## Agents stop sounding before commands

Once the agent runs at login, it is the only sound for a 1Password prompt. Coding agents no longer play Antic-EncoreInfinitum, nor the second Text-Message-Acknowledgement-Exclamation alert, before a push, fetch, `secretspec`, `op` or SSH command: the rule leaves the agents' sound rule and every agent brief. A command that needs no approval makes no sound, and one that does sounds once, when 1Password shows the prompt.

## Checks

- `nix flake check` passes on an Apple-silicon Mac.
- A nix-darwin configuration that imports the module and enables it evaluates, and its agent's `ProgramArguments` are as above; a Nix test compares them.
- After `darwin-rebuild switch`, the agent is running, and a real prompt sounds once, not twice: the hand-started copy from shell-skills is stopped first.
