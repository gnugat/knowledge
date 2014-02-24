# Git tips

Take a look at the [Git cheat sheet](../cheat-sheets/01-git.md).

Resources:

* [genomewiki](http://genomewiki.ucsc.edu/index.php/Resolving_merge_conflicts_in_Git)
* [Git scm](http://git-scm.com/book)
* [Justin Hileman](http://justinhileman.info/article/changing-history/)
* [Daniel Kummer](http://danielkummer.github.io/git-flow-cheatsheet/)
* [Git Cheatsheet](http://ndpsoftware.com/git-cheatsheet.html)
* [Resources for learning Git](http://speckyboy.com/2013/06/03/resources-for-learning-git)
* [Git tips from the pros](http://net.tutsplus.com/tutorials/tools-and-tips/git-tips-from-the-pros/)

## Solving merge conflicts

When merging with Git, conflicts can happen. To resolve them, simply:

1. find the conflicted files with `git status`
   (they'll have the `unmerged` mention)
2. edit the conflicted files and make the necessary modifications
3. `git add` the resolved files
4. `git commit` to finish the merge

## Managing ref updates (checkout, reset, commit, merge, etc)

1. checking the history of ref updates: `git reflog [show <branch>]`
2. undoing last ref update: `git reset HEAD@{1}`

Source: [Stack overflow: Undoing git reset](http://stackoverflow.com/questions/2510276/undoing-git-reset)

## Splitting a commit

1. select the commits: `git rebase -i HEAD~<number>`
2. mark the commit: change `pick` to `edit`
3. save and quit the editor
4. undo the commit: `git reset HEAD^`
5. stage and commit: `git add <file>; git commit -m <message>`
6. apply the split: `git rebase --continue`

Source: [Git scm: Splitting a commit](http://git-scm.com/book/en/Git-Tools-Rewriting-History#Splitting-a-Commit)

## Join last commits into the previous one

1. select the commits: `git rebase -i HEAD~<number>`
2. mark the commits: change `pick` to `fixup`

Source: [Git scm: Squashing commits](http://git-scm.com/book/en/Git-Tools-Rewriting-History#Squashing-Commits)

## Reorder commits

1. select the commits: `git rebase -i HEAD~<number>`
2. change the order of the lines

Source: [Git scm: Reordering commits](http://git-scm.com/book/en/Git-Tools-Rewriting-History#Reordering-Commits)

## Modify commit

:warning: These tips work for un-propagated commits

### Change the last commit message

``git commit --amend``

### Modify specific commit

1. start rebase: `git rebase -i <number>^`
2. mark the commit: change `pick` to `edit`
3. edit your files
4. add them `git add <filepattern>`
5. `git commit --amend`
6. And return to the head with `git rebase --continue`

## Commit part of a file

1. select the file: `git add -p <file>`
2. choose what to do:
   * accept the chunk: hit `y`
   * reject the chunk: hit `n`
   * select part of the chunk: hit `e`, or `s` if available
3. save and quit the editor
4. commit: `git commit -m <message>`

Source: [Git scm: Staging patches](http://git-scm.com/book/en/Git-Tools-Interactive-Staging#Staging-Patches)

## Reset a local branch from remote branch

1. start fetch: `git fetch <remote>`
2. reset from the chosen remote: `git reset --hard <remote>/<branch>`
3. check you have no differences between the two branches: `git diff <branch>...<remote>/<branch>` should respond nothing.

Source: [Git scm: Reset history](http://git-scm.com/docs/git-reset)
