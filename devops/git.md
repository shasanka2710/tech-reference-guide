# 🌿 Git Quick Reference

## Setup

```bash
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
git config --global core.editor "code --wait"
git config --global init.defaultBranch main
git config --list
```

## Repository Basics

```bash
git init                         # new repo
git clone <url>                  # clone remote
git clone <url> <directory>      # clone into directory
```

## Staging & Committing

```bash
git status
git diff                         # unstaged changes
git diff --staged                # staged changes

git add file.txt                 # stage file
git add .                        # stage all changes
git add -p                       # interactively stage hunks

git commit -m "message"
git commit -am "message"         # stage tracked + commit
git commit --amend               # amend last commit
git commit --amend --no-edit     # amend without editing message
```

## Branches

```bash
git branch                       # list local branches
git branch -a                    # list all (incl. remote)
git branch feature/login         # create branch
git checkout feature/login       # switch to branch
git checkout -b feature/login    # create + switch

# Modern syntax (Git 2.23+)
git switch main
git switch -c feature/login

git branch -d feature/login      # delete merged branch
git branch -D feature/login      # force delete
git branch -m old-name new-name  # rename
```

## Merging & Rebasing

```bash
# Merge
git checkout main
git merge feature/login
git merge --no-ff feature/login  # always create merge commit
git merge --squash feature/login # squash all commits

# Rebase
git checkout feature/login
git rebase main                  # replay commits on top of main
git rebase -i HEAD~3             # interactive rebase last 3 commits

# Rebase options (interactive)
# pick   = keep commit
# reword = change commit message
# edit   = amend commit
# squash = merge into previous commit
# fixup  = like squash, discard message
# drop   = remove commit

# Abort in-progress merge/rebase
git merge --abort
git rebase --abort
git rebase --continue            # after resolving conflicts
```

## Remote Operations

```bash
git remote -v
git remote add origin <url>
git remote set-url origin <url>

git fetch origin                 # download without merge
git pull                         # fetch + merge
git pull --rebase                # fetch + rebase
git push origin main
git push -u origin feature/login # set upstream
git push --force-with-lease      # safe force push
git push origin --delete feature/login  # delete remote branch
```

## Stash

```bash
git stash                        # stash uncommitted changes
git stash push -m "WIP: login"   # named stash
git stash list
git stash pop                    # apply latest + drop
git stash apply stash@{1}        # apply specific
git stash drop stash@{0}
git stash clear
```

## Log & History

```bash
git log
git log --oneline
git log --oneline --graph --all
git log -n 5                     # last 5 commits
git log --author="Alice"
git log --since="2024-01-01"
git log --grep="bugfix"
git log -- path/to/file          # file history
git log -p                       # show diffs

git show <commit>                # show commit details
git blame file.txt               # who changed each line
git shortlog -sn                 # commits per author
```

## Undoing Changes

```bash
# Discard working directory changes
git checkout -- file.txt         # (old syntax)
git restore file.txt             # (Git 2.23+)
git restore .                    # all files

# Unstage
git reset HEAD file.txt          # (old syntax)
git restore --staged file.txt    # (Git 2.23+)

# Undo commits (keeps changes staged)
git reset --soft HEAD~1

# Undo commits (keeps changes unstaged)
git reset --mixed HEAD~1

# Undo commits + discard changes (DESTRUCTIVE)
git reset --hard HEAD~1

# Safe undo (creates new commit)
git revert HEAD
git revert <commit>

# Recover deleted commit/branch
git reflog                       # find lost commit SHA
git checkout -b recovered <sha>
```

## Tags

```bash
git tag                          # list tags
git tag v1.0.0                   # lightweight tag
git tag -a v1.0.0 -m "Release 1.0" # annotated tag
git tag -a v1.0.0 <commit>       # tag specific commit
git push origin v1.0.0
git push origin --tags
git tag -d v1.0.0                # delete local tag
git push origin --delete v1.0.0  # delete remote tag
```

## Cherry-pick

```bash
git cherry-pick <commit>         # apply single commit
git cherry-pick A..B             # range of commits
git cherry-pick --no-commit <commit>  # stage without committing
```

## .gitignore Patterns

```gitignore
# Files
*.log
*.tmp
.env
.env.local

# Directories
node_modules/
dist/
build/
.cache/
__pycache__/
*.egg-info/

# OS files
.DS_Store
Thumbs.db

# IDEs
.idea/
.vscode/
*.swp
```

## Useful Aliases

```bash
git config --global alias.st status
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.lg "log --oneline --graph --all --decorate"
git config --global alias.undo "reset --soft HEAD~1"
```

## Git Flow Summary

```
main       -- production releases (tagged)
develop    -- integration branch
feature/*  -- new features (branch from develop)
release/*  -- release prep (branch from develop)
hotfix/*   -- urgent fixes (branch from main)
```
