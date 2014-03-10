# Git's cheat sheet

A highly opinionated [Git](http://git-scm.com/) cheat sheet:

* [Branches](#branches)
* [Tags](#tags)
* [Stashes](#stashes)

Take a look at the [Git tips](../git/tips.md).

## Branches

* listing existing branches: `git branch -avv`
* creating a new branch and switching to it: `git checkout -b <branch-name>`
* creating a new branch while cloning and switching to it: `git clone <repository> -b <branch-name>`
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
* showing the message: `git tag -n<number-of-lines>`
* pushing the tag: `git push <remote> <tag-name>`
* removing a remote tag: `git push <remote> :<tag-name>` (note the `:` before the tag name)
* removing a local tag: `git tag -d <tag-name>`
* changing the tag's commit to the current one: `git tag -f <tag-name>`
* renaming a tag: `git tag <old-tag-name> <new-tag-name>; git -d <old-tag-name>`

## Stashes

* listing current stashes: `git stash list`
* creating a new stash: `git stash`
* creating a new stash with a description: `git stash save '<description>'`
* applying the stash and remove it: `git stash pop <stash>`
* applying the stash: `git stash apply <stash>`
* removing all current stashes: `git stash clear`
* removing the stash: `git stash drop <stash>`

### The `<stash>` argument

* refering to a stash by its number: `stast@{<number>}`
* most recent stash: `stast@{0}`
* when ommiting the `<stash>` argument, `stash@{0}` is assumed

References: [git ready](http://gitready.com/beginner/2009/03/13/smartly-save-stashes.html)
