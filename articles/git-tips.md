# Git tips

This article will provide some tips for using Git, using the following sources:

* [genomewiki](http://genomewiki.ucsc.edu/index.php/Resolving_merge_conflicts_in_Git).

## Solving merge conflicts

When merging with Git, conflicts can happen. To resolve them, simply:

1. find the conflicted files with `git status`
   (they'll have the `unmerged` mention);
2. edit the conflicted files and make the necessary modifications;
3. `git add` the resolved files;
4. `git commit` to finish the merge.

## Managing your branches

1. create a new branch and switch to it: `git checkout -b <branch-name>`;
2. rename the current branch: `git branch -m <current-branch-name> <new-branch-name>`;
3. pushing the branch and tracking it: `git push -u origin <branch-name>`;
4. removing the remote branch: `git push origin :<branch-name>` (note the `:` before the branch name);
5. removing a local branch: `git branch -D <branch-name>`;
6. removing branches that doesn't exist anymore on the repository: `git remote prune origin`.
