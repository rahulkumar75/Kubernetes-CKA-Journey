# Git Daily Commands Cheat Sheet

## STATUS / TRACKING

```bash
git status                    # current changes
git log --oneline             # compact logs
git log --graph --oneline     # graph view
git diff                      # unstaged changes
git diff --staged             # staged changes
git show <commit-id>          # show commit details
git blame file.txt            # who changed lines
```

---

## STAGING

```bash
git add .                     # add all changes
git add file.txt              # add specific file
```

---

## COMMIT

```bash
git commit -m "message"       # commit changes
git commit --amend            # edit last commit
```

---

## PULL / PUSH

```bash
git pull origin main          # pull latest code
git pull --rebase             # pull with rebase

git push origin main          # push code
git push -u origin branch     # push new branch
```

---

## CLONE

```bash
git clone <repo-url>          # clone repo
```

---

## BRANCH

```bash
git branch                    # list local branches
git branch -a                 # list all branches

git checkout main             # switch branch
git checkout -b feature       # create + switch branch
git checkout -                # previous branch

git switch main               # switch branch
git switch -c feature         # create + switch branch

git merge branch-name         # merge branch
git rebase main               # rebase current branch

git cherry-pick <commit-id>   # apply specific commit
```

---

## FETCH

```bash
git fetch --all               # fetch all updates
```

---

## STASH

```bash
git stash                     # save temp work
git stash pop                 # restore stash
git stash list                # list stash
```

---

## UNDO / RESET

```bash
git restore file.txt          # discard file changes

git reset --soft HEAD~1       # remove commit keep changes
git reset --hard HEAD~1       # remove commit + changes

git revert <commit-id>        # revert commit safely
```

---

## REMOTE

```bash
git remote -v                 # show remotes
git remote add origin <url>   # add remote
```

---

## FILE OPERATIONS

```bash
git rm file.txt               # remove file
git mv old new                # rename file
```

---

## CLEANUP

```bash
git clean -fd                 # remove untracked files
```

---

## TAGS

```bash
git tag                       # list tags
git tag v1.0                  # create tag
```

---

## CONFIG

```bash
git config --list             # show git config
```
