# HOW TO: sneak a change into a commit, after the fact

1. find the target hash:
    `git log --oneline`
2. read its dates:
    `git show -s --format="author: %aI%ncommitter: %cI" <hash>`
3. stage your files:
    `git add <files>`
4. create a fixup commit:
    `git commit --fixup=<hash>`
5. fold it in, preserving both dates:
    `GIT_COMMITTER_DATE="<committer-date>" GIT_SEQUENCE_EDITOR=true git rebase -i --autosquash <hash>^`
