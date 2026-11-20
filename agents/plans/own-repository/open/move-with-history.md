---
title: The program's files and history move from shell-skills
state: proposed
kind: implementation
part-of: own-repository
needs: [repository-declared]
touches: [Package.swift, Sources, Tests, package.nix, check-native-arm64.sh, README.md]
---

# The program's files and history move from shell-skills

The files of `commands/onepassword-dialog-watcher/` arrive at this repository's root with the commits that made them, joined to the history that already holds these plans.

## Approach

1. **Split, in a scratch clone:** clone nhooey/shell-skills to a temporary folder, never the working checkout, and run `git subtree split --prefix=commands/onepassword-dialog-watcher --branch approval-alert-history` on its `master`. The branch holds only the program's commits, with its files at the root.
2. **Join:** in this repository, `git fetch <scratch clone> approval-alert-history`, then `git merge --allow-unrelated-histories FETCH_HEAD`. The merge commit says where the history came from: shell-skills, its path and the commit it was split at.
3. **Push** `master` to nhooey/onepassword-approval-alert, with an SSH remote.

The names stay as they are in this node. The rename is its own node, so the history shows a move and then a rename, not both at once.

## Why a merge, not a rewrite

This repository already has commits, the plans. Rebasing them onto the split history would change their hashes for no gain; a merge keeps both lines whole.

## Checks

- `git log --oneline -- Package.swift` lists 0babd60's split counterpart, with its author, date and message unchanged.
- `git diff <scratch clone's master>:commands/onepassword-dialog-watcher HEAD -- . ':!agents'` is empty.
- `swift build` and the tests executable pass from the root.
