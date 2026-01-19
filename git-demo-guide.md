# Git Demo: Common Scenarios & Commands

## Setup Instructions

```bash
# Create demo repository
mkdir git-demo && cd git-demo
git init
echo "# Git Demo Project" > README.md
git add README.md
git commit -m "Initial commit"

# Create a 'remote' repository (simulated locally)
cd ..
git clone --bare git-demo git-demo-remote.git
cd git-demo
git remote add origin ../git-demo-remote.git
git push -u origin main
```

---

## Scenario 1: Local vs Remote Concepts

**Explanation:**
- **Local**: Your machine's repository (working directory + staging area + local commits)
- **Remote (origin)**: The shared repository on a server (GitHub, GitLab, etc.)
- **origin/main**: Reference to the last known state of main branch on remote

```
LOCAL REPOSITORY                    REMOTE REPOSITORY (origin)
┌─────────────────────┐            ┌──────────────────┐
│ Working Directory   │            │                  │
│ (modified files)    │            │   origin/main    │
│         ↓           │            │   (remote refs)  │
│ Staging Area        │            │                  │
│ (git add)           │            └──────────────────┘
│         ↓           │                     ↑
│ Local Commits       │                     │
│ (git commit)        │            git push │ git fetch
│                     │                     │
└─────────────────────┘                     ↓
```

---

## Scenario 2: Making Changes - git status, git add, git commit

```bash
# Make some changes
echo "Let's add some content" >> README.md
echo "print('Hello World')" > app.py

# Check status
git status
```

**Output shows:**
- Untracked files (app.py)
- Modified files (README.md)

```bash
# Stage specific file
git add README.md
git status  # README.md in staging, app.py untracked

# Stage all changes
git add .
git status  # Both files staged

# Commit the changes
git commit -m "Add app.py and update README"
```

---

## Scenario 3: git stash - Temporarily Save Work

```bash
# Start working on a feature
echo "def calculate():" >> app.py
echo "    return 42" >> app.py

git status  # Shows modified app.py

# Suddenly need to switch branches or pull changes
# Save work temporarily
git stash

git status  # Working directory clean
```

**Visualization:**
```
Before stash:                After stash:
Working Dir: [modified]      Working Dir: [clean]
Stash: []                    Stash: [saved changes]
```

```bash
# Later, restore your work
git stash pop  # Applies and removes from stash
# OR
git stash apply  # Applies but keeps in stash

# List all stashes
git stash list

# Clear stash
git stash drop
```

---

## Scenario 4: git commit --amend - Fix Last Commit

```bash
# Made a commit but forgot something
git commit -m "Add calculation function"

# Oops! Forgot to add a file
echo "# Math utilities" > utils.py
git add utils.py

# Amend the previous commit (don't create new commit)
git commit --amend -m "Add calculation function and utils"
```

**Visualization:**
```
Before --amend:              After --amend:
A---B---C (forgot file)      A---B---C' (includes file)
```

**Note:** Only use `--amend` on commits that haven't been pushed!

---

## Scenario 5: git restore - Undo Changes

```bash
# Modify a file
echo "Some bad changes" >> app.py
git status

# Restore from staging (unstage)
git add app.py
git restore --staged app.py  # Unstages but keeps changes

# Restore from last commit (discard changes)
git restore app.py  # Discards all changes
```

---

## Scenario 6: git fetch vs git pull

### Setup: Simulate remote changes
```bash
# In another terminal/tab, simulate teammate's changes
cd ../git-demo-remote.git
cd ..
git clone git-demo-remote.git git-demo-teammate
cd git-demo-teammate
echo "Teammate's work" > feature.txt
git add feature.txt
git commit -m "Add feature by teammate"
git push origin main
cd ../git-demo
```

### git fetch - Download but Don't Merge

```bash
git fetch origin
```

**Visualization:**
```
Before fetch:                After fetch:
Local:  A---B---C (main)     Local:  A---B---C (main)
Remote: A---B---C---D                      D (origin/main)
                                           
Your local main is behind, but working directory unchanged
```

```bash
# Check what's new
git log origin/main --oneline
git diff main origin/main

# Decide when to integrate
git merge origin/main  # or git rebase origin/main
```

### git pull - Fetch + Merge

```bash
# Pull = fetch + merge
git pull origin main
```

**Visualization:**
```
Before pull:                 After pull (merge):
A---B---C (main)             A---B---C-------M (main)
         \                            \     /
          D (origin/main)              D---
                                       
Creates merge commit M
```

### git pull --rebase - Fetch + Rebase

```bash
# Cleaner alternative
git pull --rebase origin main
```

**Visualization:**
```
Before pull:                 After pull --rebase:
A---B---C (main)             A---D---C' (main)
         \                            
          D (origin/main)              
                                       
No merge commit, linear history
```

---

## Scenario 7: git rebase - Rewrite History

### Setup: Create divergent branches
```bash
git checkout -b feature-branch
echo "Feature work" > feature.py
git add feature.py
git commit -m "Implement feature"

# Meanwhile, main branch gets updated
git checkout main
echo "Hotfix" > hotfix.txt
git add hotfix.txt
git commit -m "Critical hotfix"
```

**Visualization:**
```
      C (feature-branch)
     /
A---B---D (main)

Branches have diverged
```

### Regular Rebase
```bash
git checkout feature-branch
git rebase main
```

**Visualization:**
```
Before rebase:               After rebase:
      C (feature)            A---B---D---C' (feature)
     /                                   (main)
A---B---D (main)             
                             
Feature commits moved to tip of main
```

### Interactive Rebase - Cleanup Commits
```bash
# Create messy history
echo "Change 1" >> feature.py
git commit -am "WIP"
echo "Change 2" >> feature.py
git commit -am "Fix typo"
echo "Change 3" >> feature.py
git commit -am "Actually fix it"

# Clean it up
git rebase -i HEAD~3
```

**In the editor that opens:**
```
pick abc1234 WIP
squash def5678 Fix typo
squash ghi9012 Actually fix it
```

**Visualization:**
```
Before interactive rebase:   After interactive rebase:
A---B---C---D---E            A---B---F
    ↑               ↑                ↑
   WIP      Fix typo...       Clean commit

Three messy commits become one clean commit
```

**Interactive rebase commands:**
- `pick` - keep commit as is
- `squash` - combine with previous commit
- `reword` - change commit message
- `edit` - stop to amend commit
- `drop` - remove commit

---

## Scenario 8: git merge - Combine Branches

```bash
# Ensure you're on the target branch
git checkout main

# Merge feature branch
git merge feature-branch
```

**Visualization (Fast-forward):**
```
Before merge:                After merge (fast-forward):
A---B---C (main)             A---B---C---D (main, feature)
         \
          D (feature)
          
If main hasn't moved, just move pointer forward
```

**Visualization (3-way merge):**
```
Before merge:                After merge:
      C (feature)            A---B-------M (main)
     /                            \     /
A---B---D (main)                   C---D (feature)
                             
Both branches had new commits, creates merge commit M
```

### Handling Merge Conflicts
```bash
# If conflicts occur
git merge feature-branch
# Auto-merging file.txt
# CONFLICT (content): Merge conflict in file.txt

# Fix conflicts in your editor
# Look for markers:
# <<<<<<< HEAD
# Your changes
# =======
# Their changes
# >>>>>>> feature-branch

# After fixing
git add file.txt
git commit  # Complete the merge
```

---

## Complete Demo Workflow

```bash
# 1. Start fresh
git status

# 2. Make local changes
echo "New feature" >> app.py
git status
git add app.py
git commit -m "Add new feature"

# 3. Oops, forgot something
echo "# TODO: Add tests" >> app.py
git add app.py
git commit --amend --no-edit

# 4. Check remote for updates
git fetch origin
git log origin/main..main  # What you have that remote doesn't
git log main..origin/main  # What remote has that you don't

# 5. Integrate remote changes
git pull --rebase origin main

# 6. Push your work
git push origin main

# 7. Create and work on feature branch
git checkout -b feature/new-stuff
echo "Experimental" > experiment.txt
git add experiment.txt
git commit -m "Experiment"

# 8. Clean up with interactive rebase
git rebase -i main

# 9. Merge back to main
git checkout main
git merge feature/new-stuff

# 10. Push everything
git push origin main
```

---

## Quick Command Reference

| Command | Purpose |
|---------|---------|
| `git status` | Check working directory state |
| `git add <file>` | Stage changes |
| `git commit -m "msg"` | Save staged changes |
| `git commit --amend` | Modify last commit |
| `git stash` | Temporarily save changes |
| `git stash pop` | Restore stashed changes |
| `git restore <file>` | Discard changes |
| `git restore --staged <file>` | Unstage changes |
| `git fetch` | Download remote changes |
| `git pull` | Fetch + merge |
| `git pull --rebase` | Fetch + rebase |
| `git rebase <branch>` | Move commits to new base |
| `git rebase -i <commit>` | Interactive rebase |
| `git merge <branch>` | Combine branches |
| `git push` | Upload local commits |

---

## Best Practices

1. **Use `git fetch` first** to see changes before integrating
2. **Use `git pull --rebase`** for cleaner history
3. **Never rebase public/shared commits** - only rebase local commits
4. **Use `--amend` carefully** - only for unpushed commits
5. **Commit often**, rebase/squash later with `-i`
6. **Check `git status`** frequently
7. **Stash before switching** branches with uncommitted work

---

## Tips for Your Demo

1. **Start with setup** - show local vs remote concept
2. **Do status/add/commit** first (basics)
3. **Show stash** when "interrupted" by urgent task
4. **Demonstrate fetch vs pull** by simulating remote changes
5. **Show rebase** with clear before/after visualization
6. **End with merge** to bring feature back
7. **Use `git log --oneline --graph --all`** to visualize branches
8. **Keep each scenario short** (2-3 minutes max)
