# Git's cheat sheet

A highly opinionated [Git](http://git-scm.com/) cheat sheet:

* [Branches](#branches)
* [Tags](#tags)

Take a look at the [Git tips](../git/tips.md).

## Branches

* listing existing branches: `git branch -avv`
* creating a new branch and switch to it: `git checkout -b <branch-name>`
* creating a branch from a remote one and track it: `git checkout -t <remote>/<branch-name>`
* tracking a remote branch: `git branch --set-upstream <branch-name> <remote>/<branch-name>`
* pushing the branch and tracking it: `git push -u <remote> <branch-name>`
* removing the remote branch: `git push <remote> :<branch-name>` (note the `:` before the branch name)
* removing a local branch: `git branch -d <branch-name>` (won't remove unmerged branches)
* removing branches that doesn't exist anymore on the repository: `git fetch -p`
* finding merged branches: `git branch --merged`
* finding unmerged branches: `git banch --no-merged`
* renaming the current branch: `git branch -m <current-branch-name> <new-branch-name>`

## Tags

* listing existing tags: `git tag`
* creating a new tag: `git tag -a <tag-name> -m '<message>'`
* creating an alias: `git tag <existing-tag-name> <new-tag-name>`
* pushing the tag: `git push <remote> <tag-name>`
* removing a remote tag: `git push <remote> :<tag-name>` (note the `:` before the tag name)
* removing a local tag: `git tag -d <tag-name>`
* changing the tag's commit to the current one: `git tag -f <tag-name>`
* renaming a tag: `git tag <old-tag-name> <new-tag-name>; git -d <old-tag-name>`
