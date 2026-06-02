# Git Commands Memory Map (Purpose-Based Revision)

This format is much better for interview recall because it answers:

> **"I have a problem → Which Git command should I use?"**

---

```text
                           GIT COMMANDS
                                │
 ───────────────────────────────┼──────────────────────────────
                                │
                   WHAT DO I WANT TO DO?
                                │

 ┌─────────────────────────────────────────────────────────────┐
 │ 1. CREATE / GET REPOSITORY                                  │
 └─────────────────────────────────────────────────────────────┘

 Start new repo
 └── git init

 Download existing repo
 └── git clone <repo>

 Check remote repositories
 └── git remote -v

 Add remote repository
 └── git remote add origin <url>

──────────────────────────────────────────────────────────────

 ┌─────────────────────────────────────────────────────────────┐
 │ 2. SEE WHAT IS HAPPENING                                    │
 └─────────────────────────────────────────────────────────────┘

 Check current status
 └── git status

 View commit history
 └── git log

 Compact history
 └── git log --oneline

 Visual branch history
 └── git log --graph --oneline

 See changes
 └── git diff

 Show details of commit
 └── git show <commit>

──────────────────────────────────────────────────────────────

 ┌─────────────────────────────────────────────────────────────┐
 │ 3. SAVE MY CHANGES                                          │
 └─────────────────────────────────────────────────────────────┘

 Add one file
 └── git add file.txt

 Add everything
 └── git add .

 Save snapshot
 └── git commit -m "message"

 Modify last commit
 └── git commit --amend

──────────────────────────────────────────────────────────────

 ┌─────────────────────────────────────────────────────────────┐
 │ 4. SEND / RECEIVE CODE                                      │
 └─────────────────────────────────────────────────────────────┘

 Upload code
 └── git push

 Download changes
 └── git pull

 Download only (no merge)
 └── git fetch

 Compare local vs remote
 └── git diff origin/main

──────────────────────────────────────────────────────────────

 ┌─────────────────────────────────────────────────────────────┐
 │ 5. WORK WITH BRANCHES                                       │
 └─────────────────────────────────────────────────────────────┘

 List branches
 └── git branch

 Create branch
 └── git branch feature

 Create + switch
 └── git checkout -b feature
 └── git switch -c feature

 Switch branch
 └── git switch feature

 Delete branch
 └── git branch -d feature

──────────────────────────────────────────────────────────────

 ┌─────────────────────────────────────────────────────────────┐
 │ 6. COMBINE CHANGES                                          │
 └─────────────────────────────────────────────────────────────┘

 Merge branch
 └── git merge feature

 Linear history
 └── git rebase main

 Copy specific commit
 └── git cherry-pick <commit>

──────────────────────────────────────────────────────────────

 ┌─────────────────────────────────────────────────────────────┐
 │ 7. TEMPORARILY SAVE WORK                                    │
 └─────────────────────────────────────────────────────────────┘

 Save unfinished work
 └── git stash

 View stashes
 └── git stash list

 Restore stash
 └── git stash apply

 Restore + delete stash
 └── git stash pop

 Delete stash
 └── git stash drop

──────────────────────────────────────────────────────────────

 ┌─────────────────────────────────────────────────────────────┐
 │ 8. UNDO CHANGES                                             │
 └─────────────────────────────────────────────────────────────┘

 Undo file changes
 └── git restore file.txt

 Unstage file
 └── git restore --staged file.txt

 Undo last commit
 └── git reset HEAD~1

 Keep code, remove commit
 └── git reset --soft HEAD~1

 Delete everything
 └── git reset --hard HEAD~1

 Safely undo commit
 └── git revert <commit>

──────────────────────────────────────────────────────────────

 ┌─────────────────────────────────────────────────────────────┐
 │ 9. RELEASE MANAGEMENT                                       │
 └─────────────────────────────────────────────────────────────┘

 Create tag
 └── git tag v1.0

 Annotated tag
 └── git tag -a v1.0 -m "release"

 Push tag
 └── git push origin v1.0

 List tags
 └── git tag

──────────────────────────────────────────────────────────────

 ┌─────────────────────────────────────────────────────────────┐
 │ 10. INVESTIGATION / TROUBLESHOOTING                         │
 └─────────────────────────────────────────────────────────────┘

 Who changed code?
 └── git blame file.txt

 Find bad commit
 └── git bisect

 Find deleted commit
 └── git reflog

 Compare commits
 └── git diff commit1 commit2

──────────────────────────────────────────────────────────────

 ┌─────────────────────────────────────────────────────────────┐
 │ 11. CLEANUP                                                 │
 └─────────────────────────────────────────────────────────────┘

 Remove untracked files
 └── git clean -f

 Remove directories too
 └── git clean -fd

 Delete merged branch
 └── git branch -d branch

 Force delete branch
 └── git branch -D branch
```

# Production Scenario Cheat Sheet

| Situation                 | Command                 |
| ------------------------- | ----------------------- |
| See what changed          | `git status`            |
| Check commit history      | `git log --oneline`     |
| Save changes              | `git commit`            |
| Push code                 | `git push`              |
| Get latest code           | `git pull`              |
| Only download latest code | `git fetch`             |
| Create branch             | `git switch -c feature` |
| Switch branch             | `git switch branch`     |
| Merge feature             | `git merge feature`     |
| Linear history            | `git rebase`            |
| Temporary save work       | `git stash`             |
| Restore stash             | `git stash pop`         |
| Undo last commit          | `git reset`             |
| Safely undo commit        | `git revert`            |
| Recover lost commit       | `git reflog`            |
| Copy one commit           | `git cherry-pick`       |
| Create release            | `git tag`               |
| Find who changed code     | `git blame`             |

# Interview Recall Formula

Remember Git in **11 verbs**:

```text
CREATE    → init, clone
CHECK     → status, log, diff
SAVE      → add, commit
SYNC      → fetch, pull, push
BRANCH    → branch, switch
MERGE     → merge, rebase
PAUSE     → stash
UNDO      → restore, reset, revert
RELEASE   → tag
INVESTIGATE → blame, bisect, reflog
CLEANUP   → clean
```

If you can explain these **11 categories with the main command and when you'd use it in a real project**, you're already at a strong level for most 0–3 YOE DevOps/Developer Git interviews.
