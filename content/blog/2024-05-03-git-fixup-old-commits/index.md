+++
title = "Edit/Fix-up Old Commits in Git"
+++

In this blog post, we'll walk through the process of creating a "fix-up" commit to amend an old commit identified by a specific hash, `<OLD_HASH>`.

First, we create the fix-up commit by adding our changes and committing with the `--fixup=<OLD_HASH>` flag:

```sh
git add <changes>
git commit --fixup=<OLD_HASH>
```

Next, we perform an interactive rebase starting from one commit before `<OLD_HASH>`:

```sh
git rebase --interactive --autosquash <COMMIT_BEFORE_OLD_HASH>
```

If `<COMMIT_BEFORE_OLD_HASH>` is the first or root commit, pass `--root`[^1] instead.

Upon executing this command, your default editor opens up (with a text similar to below), allowing you to save and exit. Done.

```
pick a43f263 Add initial interfaces
pick a643ac3 Add base types
fixup a08d5fa Add implementation
pick 01e156a fix typo in base types

# Rebase fba8887..a08d5fa onto fba8887 (3 commands)
#
# Commands:
# p, pick <commit> = use commit
# r, reword <commit> = use commit, but edit the commit message
# e, edit <commit> = use commit, but stop for amending
# s, squash <commit> = use commit, but meld into previous commit
# f, fixup [-C | -c] <commit> = like "squash" but keep only the previous
#                    commit's log message, unless -C is used, in which case
#                    keep only this commit's message; -c is same as -C but
#                    opens the editor

# [1]
```

The `--autosquash` option is crucial here as it automatically arranges fix-up commits immediately after the commit they are intended to fix, streamlining the rebase process.[^2]


However, it's important to note that these operations __rewrite your repository's history__, which can pose challenges in collaborative environments. Therefore, exercise caution when force-pushing your new commits to the remote repository, using `--force-with-lease`:


```sh
git push --force-with-lease
```

If the Git at `origin` successfully completes the exchange, you're all set. Otherwise, you can run `git fetch origin` to update your local repository, make any necessary adjustments, and attempt the force-push again.

---

[^1]: Only commits between the root and the tip of your current branch are included in the rebase. You can use `--root` to rebase your entire git tree. ([src](https://andrewlock.net/smoother-rebases-with-auto-squashing-git-commits/))

[^2]: For a more detailed explanation of `--autosquash`, you can read [this insightful article](https://andrewlock.net/smoother-rebases-with-auto-squashing-git-commits/).
