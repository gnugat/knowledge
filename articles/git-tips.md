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
