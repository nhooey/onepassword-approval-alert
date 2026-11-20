---
title: The account's configuration repository declares the onepassword-approval-alert repository
state: proposed
kind: implementation
part-of: own-repository
touches: []
---

# The account's configuration repository declares the onepassword-approval-alert repository

The repository exists on GitHub once the account's configuration repository holds its entry and the entry is applied.

- **The entry:** name `onepassword-approval-alert`, a description, and the topics `1password`, `launchd`, `macos`, `nix-darwin`, `nix-flake` and `swift`. It sets nothing else, so it is public, on `master`, with the default-branch ruleset of every public repository.
- **What is left:** merge the entry in the configuration repository, then apply it.

Check: `gh repo view nhooey/onepassword-approval-alert` shows the description and topics.
