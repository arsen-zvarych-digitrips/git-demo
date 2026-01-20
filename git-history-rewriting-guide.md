# Git History Rewriting: Protected vs Unprotected Branches

## Concept Overview

**History Rewriting** changes commit history by modifying, combining, or reordering commits. This is powerful but dangerous if done incorrectly.

### The Golden Rule
```
┌────────────────────────────────────────────────────┐
│  NEVER REWRITE HISTORY ON SHARED/PROTECTED BRANCHES │
│                                                      │
│  ✅ Safe: Rewrite on personal feature branches      │
│  ❌ Dangerous: Rewrite on main/master/develop       │
└────────────────────────────────────────────────────┘
```

---

## Why Protect Main Branch?

```
SCENARIO: What happens if you rewrite main?

Original State:
Developer 1 (main):  A---B---C---D
Developer 2 (main):  A---B---C---D
Developer 3 (main):  A---B---C---D
     Everyone has same history ✅

After Dev 1 rewrites main (force push):
Developer 1 (main):  A---B---C'---D'  (rewritten)
Developer 2 (main):  A---B---C----D   (old)
Developer 3 (main):  A---B---C----D   (old)
     Histories diverged! ❌

When Dev 2 tries to pull:
Error: "Your branch and 'origin/main' have diverged"
     Now everyone has problems! 💥
```

---

## Demo Scenario: Feature Branch Workflow

### Setup

```bash
# Create the demo environment
mkdir git-history-demo && cd git-history-demo
git init -b main

# Create initial commits on main
echo "# Project" > README.md
git add README.md
git commit -m "Initial commit"

echo "v1.0" > version.txt
git add version.txt
git commit -m "Add version file"

echo "Production ready code" > app.py
git add app.py
git commit -m "Add production app"

# Simulate remote (protected main)
cd ..
git clone --bare git-history-demo git-history-remote.git
cd git-history-demo
git remote add origin ../git-history-remote.git
git push -u origin main

# Set up branch protection (conceptually - normally done on GitHub/GitLab)
echo "⚠️  Main branch is now 'protected' - no force pushes allowed"
```

**Initial State:**
```
main (protected):  A---B---C
                   ↑       ↑
              Initial   Production
```

---

## Part 1: Safe History Rewriting on Feature Branch

### Create Feature Branch with Messy History

```bash
# Create feature branch
git checkout -b feature/user-auth

# Make several messy commits
echo "def login():" > auth.py
git add auth.py
git commit -m "WIP auth"

echo "    pass" >> auth.py
git add auth.py
git commit -m "oops forgot body"

echo "def logout():" >> auth.py
git add auth.py
git commit -m "add logout"

echo "    # TODO: implement" >> auth.py
git add auth.py
git commit -m "fix typo"

echo "    return True" >> auth.py
git add auth.py
git commit -m "actually implement logout"
```

**Current State:**
```
main (protected):     A---B---C
                           \
feature/user-auth:          D---E---F---G---H
                            ↑   ↑   ↑   ↑   ↑
                          WIP oops add fix impl
                          
Messy feature branch history ❌
```

### Clean Up with Interactive Rebase

```bash
# View the messy history
git log --oneline

# Interactive rebase to clean it up
git rebase -i main
```

**In the editor:**
```
pick D WIP auth
squash E oops forgot body
pick F add logout
squash G fix typo
squash H actually implement logout

# Becomes:
pick D Add login function
pick F Add logout function
```

**After Interactive Rebase:**
```
Before:                              After:
main:     A---B---C                  main:     A---B---C
               \                                     \
feature:        D-E-F-G-H            feature:        D'-F'
                                                     ↑   ↑
                                                  login logout
                                                  
Clean, professional history ✅
```

---

## Part 2: Attempting to Rewrite Protected Main (DON'T DO THIS!)

### Scenario: Someone Tries to Rewrite Main

```bash
# Switch to main
git checkout main

# Try to amend a commit (rewrites history)
git commit --amend -m "Modified production commit"

# Try to force push
git push --force origin main
```

**What Happens:**
```
! [remote rejected] main -> main (protected branch hook declined)
error: failed to push some refs to 'origin'

Protected branch prevents the force push ✅
```

**Visualization of What Would Happen Without Protection:**
```
Before force push:           After force push:
Remote:  A---B---C           Remote:  A---B---C'
Team:    A---B---C           Team:    A---B---C (now orphaned!)
                             
Everyone else's work becomes incompatible! 💥

Team member tries to pull:
A---B---C (their local)
     \
      C' (remote)
      
Git doesn't know how to merge - CONFLICT!
```

---

## Part 3: The Right Way - Rebase Feature, Merge to Main

### Update Feature Branch from Main

```bash
# Simulate main branch receiving new commits
git checkout main
echo "Security update" > security.patch
git add security.patch
git commit -m "Apply security patch"
git push origin main
```

**Current State:**
```
main:     A---B---C---I (new commit on main)
               \
feature:        D'---F' (our clean feature)
```

### Rebase Feature onto Latest Main

```bash
# Update feature branch
git checkout feature/user-auth
git rebase main
```

**Rebase Process:**
```
Before rebase:                After rebase:
main:     A---B---C---I        main:     A---B---C---I
               \                                     \
feature:        D'---F'        feature:               D''---F''

Feature commits moved on top of latest main ✅
```

```bash
# View the result
git log --oneline --graph --all
```

### Push Feature Branch (First Time)

```bash
# Push feature branch to remote
git push -u origin feature/user-auth
```

**State:**
```
Local:   main:    A---B---C---I
              \
         feature:              D''---F''

Remote:  main:    A---B---C---I
              \
         feature:              D''---F'' (same)
```

### Updating After More Changes and Rebase

```bash
# Make more changes
echo "def validate():" >> auth.py
git add auth.py
git commit -m "Add validation"

# Need to rebase again
git rebase main
```

**Problem: Already Pushed!**
```
Local feature:   A---B---C---I---D''---F''---G
Remote feature:  A---B---C---I---D''---F''

They diverged because of rebase!
```

```bash
# Regular push fails
git push origin feature/user-auth
# ! [rejected] feature/user-auth -> feature/user-auth (non-fast-forward)

# Force push is OK on feature branches (you own it)
git push --force-with-lease origin feature/user-auth
```

**Why `--force-with-lease` is safer than `--force`:**
```
--force:              Overwrites remote no matter what
--force-with-lease:   Only overwrites if nobody else pushed

Protects against:
Your local:    A---B---C---D
Remote:        A---B---C---D---E (teammate pushed E!)
               
--force:       Deletes E (teammate's work lost!) ❌
--force-with-lease: Rejects push (conflict detected) ✅
```

---

## Part 4: Final Merge to Main (No Force Push Needed!)

### Create Pull Request / Merge Request

```bash
# Feature branch is clean and ready
git checkout main
git merge feature/user-auth
```

**Merge Process:**
```
Before merge:                    After merge:
main:     A---B---C---I           main:     A---B---C---I---M
                   \                                    \   /
feature:            D''---F''---G  feature:              D''---F''---G

Merge commit M brings feature into main ✅
```

```bash
# Push to remote (regular push, no force!)
git push origin main
```

**Key Point:**
```
┌─────────────────────────────────────────────┐
│ Main branch NEVER needed force push!        │
│ Only feature branch used force push         │
│ Merge to main was clean, fast-forward       │
└─────────────────────────────────────────────┘
```

---

## Complete Workflow Visualization

```
TIME →

1. Initial State
   main:  A---B---C

2. Create Feature Branch
   main:  A---B---C
               \
   feature:     D---E---F---G---H (messy)

3. Interactive Rebase (Clean History)
   main:  A---B---C
               \
   feature:     D'---F' (clean)

4. Main Gets Update
   main:  A---B---C---I
               \
   feature:     D'---F'

5. Rebase Feature onto Main
   main:  A---B---C---I
                   \
   feature:         D''---F''

6. Push Feature (force-with-lease OK)
   origin/feature:  D''---F''

7. Make More Changes
   feature:  D''---F''---G

8. Final Rebase & Force Push
   origin/feature:  D''---F''---G

9. Merge to Main (NO FORCE!)
   main:  A---B---C---I---M
                   \     /
                    D''---F''---G

✅ Main history: Linear, clean, no force pushes
✅ Feature history: Rewritten freely, clean result
```

---

## Commands Summary

### ✅ Safe on Feature Branches

```bash
# Clean up commits
git rebase -i main
git rebase main

# Amend last commit
git commit --amend

# Force push with safety check
git push --force-with-lease origin feature/branch

# Squash commits
git rebase -i HEAD~3
```

### ❌ Never on Main/Protected Branches

```bash
# DON'T DO THESE ON MAIN:
git commit --amend           # Rewrites history
git rebase                   # Rewrites history
git reset --hard HEAD~1      # Deletes commits
git push --force            # Overwrites remote

# These cause:
# - Team member conflicts
# - Lost work
# - Broken CI/CD
# - Repository corruption
```

### ✅ Safe on Main Branch

```bash
# Always use these on main:
git merge feature/branch     # Creates merge commit
git revert <commit>          # Undoes commit without rewriting
git push origin main         # Regular push
git pull origin main         # Regular pull
```

---

## Real-World Scenario: Team Conflict

### The Problem

```
Initial state (everyone in sync):
Alice:   A---B---C
Bob:     A---B---C
Remote:  A---B---C

Alice rewrites history and force pushes:
Alice:   A---B---C'---D'
Remote:  A---B---C'---D'

Bob tries to push his work:
Bob:     A---B---C---E
Remote:  A---B---C'---D'
         
Bob's push rejected! His C doesn't match remote C'
```

**Bob's Terminal:**
```bash
git push
# ! [rejected] main -> main (non-fast-forward)

git pull
# Auto-merging failed; fix conflicts and commit the result

git log --graph
#     * (remote) C'---D'
#     * (local)  C----E
#    /
#   B
#  |
#  A
# 
# Diverged history - difficult to resolve!
```

### The Solution: Protect Main

```
Protected main workflow:

Alice (feature branch):  A---B---C
                              \
                               D---E (rebase freely)

Bob (feature branch):    A---B---C
                              \
                               F---G (rebase freely)

Main (protected):        A---B---C---M1---M2
                              \        \
                               D---E    F---G
                               
Both merge cleanly, no conflicts!
```

---

## Key Takeaways

### 1. Branch Protection Rules

```
┌─────────────────────┬──────────────────────────┐
│ Branch Type         │ History Rewriting        │
├─────────────────────┼──────────────────────────┤
│ main/master         │ ❌ NEVER                 │
│ develop/staging     │ ❌ NEVER                 │
│ release/*           │ ❌ NEVER                 │
│ feature/* (yours)   │ ✅ Yes, freely           │
│ feature/* (shared)  │ ⚠️  Coordinate with team │
└─────────────────────┴──────────────────────────┘
```

### 2. When History Rewriting is Good

```
Before rebase -i:
A---"WIP"---"fix typo"---"oops"---"actually works"---"final fix"

After rebase -i:
A---"Implement user authentication feature"

✅ Cleaner git log
✅ Easier code review
✅ Better git bisect
✅ Professional commits
```

### 3. When History Rewriting is Bad

```
Shared branch with force push:

Dev 1: A---B---C'  (force pushed)
Dev 2: A---B---C---D (has unrewritten C)
Dev 3: A---B---C---E (has unrewritten C)

Everyone needs to:
1. Detect the problem
2. Stash their work
3. Reset to match remote
4. Reapply their commits
5. Hope nothing was lost

❌ Wasted time
❌ Potential data loss
❌ Team frustration
```

---

## Practice Scenario

Try this sequence to see the difference:

```bash
# 1. Create protected main scenario
./setup-protected-branches.sh

# 2. Create messy feature branch
git checkout -b feature/test
for i in {1..5}; do
  echo "Change $i" >> file.txt
  git commit -am "WIP $i"
done

# 3. Clean it up
git rebase -i main
# Squash all into one commit

# 4. Try to rewrite main (will fail)
git checkout main
git commit --amend -m "Modified"
git push --force
# ❌ Rejected!

# 5. Merge feature properly
git merge feature/test
git push
# ✅ Success!
```

---

## Emergency: "I Force Pushed to Main!"

If you accidentally force push to a protected branch:

```bash
# 1. DON'T PANIC
# 2. Find the old commit hash
git reflog
# Find the commit before force push

# 3. Reset to old commit
git reset --hard <old-commit-hash>

# 4. Force push back
git push --force

# 5. Notify your team immediately!

# 6. If remote has protection: Contact admin to restore
```

**Visualization:**
```
Bad force push:
A---B---C---D (old, correct)
         \
          C'---D' (wrong, force pushed)

Recovery:
A---B---C---D (restored)

Reflog saved the day! ✅
```

---

## Conclusion

```
┌────────────────────────────────────────────────────┐
│              GOLDEN RULES                          │
├────────────────────────────────────────────────────┤
│ 1. Never rewrite public/shared branch history     │
│ 2. Always rebase/clean feature branches           │
│ 3. Use --force-with-lease, never --force          │
│ 4. Merge features to main (don't rebase main)     │
│ 5. When in doubt, ask the team first              │
└────────────────────────────────────────────────────┘
```

**Remember:**
- Protected branches = stable, reliable history
- Feature branches = your playground, rewrite freely
- Force push = dangerous unless on your own branch
- Good commit history = easier debugging and code review
