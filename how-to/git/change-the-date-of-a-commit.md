# HOW TO: change the date of a commit

The **author date** is when the commit was originally written.
The **committer date** is when the commit was last applied (e.g. after a rebase).

## Last commit

1. amend its dates, keeping author and committer dates the same:
    `d="<YYYY-MM-DD HH:MM:SS +0000>"; GIT_COMMITTER_DATE="$d" git commit --amend --date="$d" --no-edit`

## Specific commit

1. find the target hash:
    `git log --oneline`
2. read its dates:
    `git show -s --format="author: %aI%ncommitter: %cI" <hash>`
3. start an interactive rebase on it:
    `git rebase -i --committer-date-is-author-date <hash>^`
    (in the editor, change `pick <hash>` to `edit <hash>`)
4. amend its dates:
    `d="<YYYY-MM-DD HH:MM:SS +0000>"; GIT_COMMITTER_DATE="$d" git commit --amend --date="$d" --no-edit`
5. continue, preserving other commits' dates:
    `git rebase --continue`
