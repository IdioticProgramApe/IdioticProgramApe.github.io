---
title: Commonly Used Commands in Git
author: ipa
date: 2023-04-06
categories: [Version Control]
tags: [code management, ci/cd, git]
---

## First-Time Git Setup

| Command                                        | Description                                                  |
| ---------------------------------------------- | ------------------------------------------------------------ |
| `git config --list --show-origin [param name]` | used to view all/given param name the existing git settings and their setting files' whereabouts |
| `git config [--global] user.name <username>`   | used to setup the submitter's username for current repo without `--global`, otherwise for all repos |
| `git config [--global] user.email <email>`     | used to setup the submitter's email for current repo without `--global`, otherwise for all repos |

Other parameters can get from the `.gitconfig` files, use the same pattern to overwrite it.

## Setup Repository

| Command                         | Description                                                  |
| ------------------------------- | ------------------------------------------------------------ |
| `git init`                      | initialize a git repo, will create a .git folder in the cwd  |
| `git clone <url> [path]`        | clone a remote git repo to cwd or the given path if provided |
| `git push origin <branch name>` | push the local content to the remote repo's branch           |

## Helpers

| Command                     | Description                                                  |
| --------------------------- | ------------------------------------------------------------ |
| `git status`                | show the current status of the current repo, usually to check how many files are added/deleted/modified |
| `git stash`                 | save the uncommitted local changes to an invisible place which sometimes can be useful when conflicts happen during `pull` |
| `git stash show`            | check the current stash status                               |
| `git stash pop`             | reapply the previously stashed changes to the working area, this also remove the changes from the stash area |
| `git branch`                | show current branch alias                                    |
| `git remote -v`             | show the branch alias and its remote repo                    |
| `git log [--oneline]`       | show the commit history of the current repo, use `--oneline` to make the history more compact |
| `git log --oneline --graph` | show the commit history graph in the terminal                |
| `git help <cmd>`            | show the help information of the given cmd, e.g. `init`      |
| `git <cmd> -h`              | same as the previous cmd                                     |
| `git show <commit ID>`      | show a commit detail based on the given information, `any` can be SHA-1, tags, branch alias, `HEAD`. <br>use `~`(backward displacement), `^` and commit numbers to get the commit ID |
| `git tag`                   | show all the tags created in the current repo                |

## Repository Manipulation

### Local Operations

| Command                                     | Description                                                  |
| ------------------------------------------- | ------------------------------------------------------------ |
| `git add <file/folder>`                     | add the given file/folder to the pending commit list (stage the changes), wildcards are allowed |
| `git diff`                                  | show any uncommitted changes since the last commit           |
| `git diff [--color-words] <A> <B>`          | show the [colored] differences between `A` and `B`, could be files, could be branches |
| `git commit -m <some message>`              | commit the current pending commit list to local repo with message |
| `git commit --amend -m <some message>`      | modify the most recent commit, it will combine staged changes with the previous commit <br>if no file is staged, it will only modify the commit message |
| `git commit --amend --no-edit`              | only update files without touching commit messages           |
| `git tag -a -m <some message> <tag name>`   | annotate a tag named `tag name` at `HEAD`                    |
| `git tag -d <tag name>`                     | delate a tag named `tag name`                                |
| `git branch <branch name>`                  | create a new branch named `branch name`                      |
| `git branch -d <branch name>`               | delete the branch named `branch name` <br>will warn before deleting if the branch has not been merged into its upstream branch |
| `git branch -D <branch name>`               | force delete the branch named `branch name`, BE CAREFUL, no pre-warning |
| `git reflog`                                | used to know which branch was accidently deleted previously  |
| `git checkout <branch name> [commit ID]`    | update the working tree to match the specific working tree defined by the given branch<br>will also update `HEAD` to make the given branch as the current branch at the given commit or the latest one |
| `git checkout -b <branch name> [commit ID]` | equivalent to `git branch` and `git checkout`, can be also used to restore one branch's commits |

#### Merge Branches

| Command                                 | Description                                                  |
| --------------------------------------- | ------------------------------------------------------------ |
| `git merge [target name] <source name>` | use fast-forward method to merge the branch `source name` to current branch or the branch `target name` |
| `git merge --squash <source name>`      | same as `git merge` and will additionally squash all the commits to merge into one commit on target branch |
| `git merge --no-ff <source name>`       | manual merge operation, need to edit a message for the merge |
| `git rebase <target name>`              | use rebase to move the current branch to the tip of the branch `target name`, this will change the history tree |
| `git rebase --continue`                 | let rebase operation to continue after solving the conflicts <br>usually used when conflicts occurred during a rebase process |

Normally we prefer the `merge` commands than the `rebase` commands, since `rebase` will clear the source branch and lose the track of repo activities.

Never rebase public branches!

### Remote Operations

| Command     | Description                                                  |
| ----------- | ------------------------------------------------------------ |
| `git push`  | push the local commits to the remote repo                    |
| `git fetch` | fetch the remote repo's most recent information, no change to local files |
| `git pull`  | pull/synchronize the local repo up to the remote repo        |

