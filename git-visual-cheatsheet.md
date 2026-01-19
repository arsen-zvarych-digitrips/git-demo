# Git Visual Cheat Sheet

## 1. Local vs Remote Architecture

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

## 2. git fetch vs git pull

### git fetch
```
Before:                      After:
Local:  A---B---C (main)     Local:  A---B---C (main)
Remote: A---B---C---D                      D (origin/main)
                                           
Downloads D but doesn't change your working directory
```

### git pull (fetch + merge)
```
Before:                      After:
A---B---C (main)             A---B---C-------M (main)
         \                            \     /
          D (origin/main)              D---
                                       
Creates merge commit M
```

### git pull --rebase (fetch + rebase)
```
Before:                      After:
A---B---C (main)             A---D---C' (main)
         \                            
          D (origin/main)              
                                       
Linear history, no merge commit
```

---

## 3. git commit --amend

```
Before:                      After:
A---B---C (forgot file)      A---B---C' (includes everything)

Rewrites last commit instead of creating new one
```

---

## 4. git restore

```
Working Directory            Staging Area           Repository
┌──────────────┐            ┌──────────────┐        ┌──────────┐
│ file.txt     │            │              │        │ Commit C │
│ (modified)   │ git add    │ file.txt     │        │          │
│              │─────────►  │ (staged)     │        │          │
│              │            │              │        │          │
└──────────────┘            └──────────────┘        └──────────┘
       ▲                           │
       │ git restore               │ git commit
       │ (discard)                 ▼
       │                    ┌──────────────┐
       │git restore         │              │
       │--staged            │  Commit D    │
       └────────────────────┤              │
                            └──────────────┘
```

---

## 5. git rebase

### Simple Rebase
```
Before:                      After:
      C (feature)            A---B---D---C' (feature)
     /                                   (main)
A---B---D (main)             

Moves feature commits on top of main
```

### Interactive Rebase (squash)
```
Before:                      After:
A---B---C---D---E            A---B---F
    │   │   │   │                │
   WIP Fix Oops Done         Clean commit

Combines multiple commits into one
```

---

## 6. git merge

### Fast-Forward Merge
```
Before:                      After:
A---B---C (main)             A---B---C---D (main, feature)
         \
          D (feature)
          
main pointer just moves forward
```

### 3-Way Merge
```
Before:                      After:
      C (feature)            A---B-------M (main)
     /                            \     /
A---B---D (main)                   C---D (feature)

Creates new merge commit M with two parents
```

### Merge Conflict
```
A---B---C (main)
     \
      D (feature)
      
Both modified same file differently

<<<<<<< HEAD
Your changes from main
=======
Their changes from feature
>>>>>>> feature
```

---

## 7. Complete Git Workflow

```
                    ┌─────────────────┐
                    │  Remote (origin)│
                    │                 │
                    │   origin/main   │
                    └─────────────────┘
                       ▲           │
              git push │           │ git fetch
                       │           ▼
┌───────────────────────────────────────────────┐
│              LOCAL REPOSITORY                 │
│                                               │
│  Working Dir    Staging Area    Local Commits│
│  ┌─────────┐    ┌─────────┐    ┌─────────┐  │
│  │ file.txt│    │         │    │ Commit C│  │
│  │(modified│───→│ file.txt│───→│         │  │
│  │         │add │ (staged)│commit         │  │
│  └─────────┘    └─────────┘    └─────────┘  │
│       ▲                                       │
│       │ restore                               │
│       │ (discard)                             │
│  ┌─────────┐                                  │
│  │  Stash  │ stash / pop                      │
│  └─────────┘                                  │
└───────────────────────────────────────────────┘
```

---

## 8. Branch Visualization

```
main branch:      A---B---C---D---E (main)
                       \
feature branch:         F---G---H (feature)
                             \
sub-feature:                  I---J (sub-feature)

After merging feature to main:
A---B---C---D---E-------M (main)
         \             /
          F---G---H---

After rebasing feature onto main:
A---B---C---D---E (main)
                 \
                  F'---G'---H' (feature)
```

---

## 9. Typical Workflow Patterns

### Pattern 1: Feature Branch Workflow
```
1. Create branch:     main---A
                           \
                            B (feature)

2. Work on feature:   main---A
                           \
                            B---C---D (feature)

3. Update from main:  main---A---E---F
                           \
                            B---C---D (feature)
                            
4. Rebase:            main---A---E---F
                                     \
                                      B'--C'--D' (feature)

5. Merge:             main---A---E---F---M
                                     \   /
                                      B'-C'-D'
```

### Pattern 2: Pull Request / Code Review
```
Local:  A---B---C---D (feature)
                ↓ git push
Remote: A---B---C---D (origin/feature)
                ↓ Create PR
Review: [Code Review on GitHub/GitLab]
                ↓ Approved
Remote: A---B---C---D---M (origin/main)
                ↓ git pull
Local:  A---B---C---D---M (main)
```

---

## Command Quick Reference with Visuals

```
┌────────────────────┬──────────────────────────────┐
│ Command            │ Visual Effect                │
├────────────────────┼──────────────────────────────┤
│ git commit         │ A---B---C → A---B---C---D    │
│ git commit --amend │ A---B---C → A---B---C'       │
│ git rebase main    │ A-B-C → A-B-D-C'            │
│                    │    \ /                       │
│ git merge feature  │ A-B-C → A-B-C-M             │
│                    │    \     /                   │
│ git fetch          │ Updates origin/main ref      │
│ git pull           │ fetch + merge                │
│ git pull --rebase  │ fetch + rebase               │
│ git restore file   │ Discard changes              │
│ git stash          │ Save & clean working dir     │
└────────────────────┴──────────────────────────────┘
```

---

## Commit History Representations

### Linear (after rebase)
```
A---B---C---D---E---F (main)
Clean, easy to read
```

### Non-linear (after merges)
```
A---B---C-------E-------G (main)
     \         /       /
      D-------         F (features)
Shows actual development flow
```

---

## Remember

- **Solid lines** (---) = commits
- **Circles** (A, B, C) = commit hashes
- **Labels** (main, feature) = branch pointers
- **origin/main** = remote tracking branch
- **Vertical lines** = time progression (down = newer)
