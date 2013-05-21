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

1. listing existing branches: `git branch -a`;
2. creating a new branch and switch to it: `git checkout -b <branch-name>`;
3. renaming the current branch: `git branch -m <current-branch-name> <new-branch-name>`;
4. pushing the branch and tracking it: `git push -u origin <branch-name>`;
5. creating a branch from a remote one and track it: `git checkout -t origin/<branch-name>`;
6. removing the remote branch: `git push origin :<branch-name>` (note the `:` before the branch name);
7. removing a local branch: `git branch -d <branch-name>` (won't remove unmerged branches);
8. removing branches that doesn't exist anymore on the repository: `git fetch -p`.

## Managing your tags

1. listing existing tags: `git tag`;
2. creating a new tag: `git tag -a <tag-name> -m '<message>'`;
3. pushing the tag: `git push origin <tag-name>`;
4. creating an alias: `git tag <existing-tag-name> <new-tag-name>`;
5. removing a remote tag: `git push origin :<tag-name>` (note the `:` before the tag name);
6. removing a local tag: `git tag -d <tag-name>`;
7. renaming a tag: create an alias and remove the old tag.
