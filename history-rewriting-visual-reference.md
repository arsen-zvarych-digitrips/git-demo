# History Rewriting Visual Quick Reference

## 1. The Core Concept

```
SAFE ZONE                          DANGER ZONE
(Your feature branch)              (Shared/protected branch)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

✅ Rewrite freely                  ❌ Never rewrite
✅ Force push OK                   ❌ No force push
✅ Squash commits                  ❌ No squashing
✅ Rebase often                    ❌ No rebasing
✅ Amend commits                   ❌ No amending

feature/your-work                  main / master / develop
```

---

## 2. History Rewriting Operations

### Interactive Rebase (Squash)
```
Before:                          After:
A---B---C---D---E               A---F
    │   │   │   │                   │
   WIP Fix Oops Done            Clean commit
   
Command: git rebase -i HEAD~4
Action:  Combine 4 commits into 1
Result:  Professional history
```

### Commit Amend
```
Before:                          After:
A---B---C                       A---B---C'
        │                               │
   Incomplete                      Complete
   
Command: git commit --amend
Action:  Modify last commit
Result:  Fixed commit
```

### Rebase onto Another Branch
```
Before:                          After:
      D---E (feature)            A---B---C---D'---E' (feature)
     /                                       │
A---B---C (main)                         main
   
Command: git rebase main
Action:  Move feature commits to tip of main
Result:  Linear history
```

---

## 3. Force Push Comparison

### ❌ --force (Dangerous)
```
Local:   A---B---C'              Remote:  A---B---C---D
                                          
git push --force
                                 ↓
Result:  A---B---C'              Remote:  A---B---C'
                                          
Teammate's commit D is LOST! 💥
```

### ✅ --force-with-lease (Safe)
```
Local:   A---B---C'              Remote:  A---B---C---D
                                          
git push --force-with-lease
                                 ↓
Result:  Push REJECTED! ✅       Remote:  A---B---C---D
                                          
Protects teammate's commit D
```

---

## 4. Protected Main Branch Workflow

```
┌─────────────────────────────────────────────────────┐
│                     MAIN (Protected)                 │
│         A───────B───────C──────────I─────────M      │
│                          \         /\       /        │
│                           \       /  \     /         │
│                            \     /    \   /          │
│      Feature 1              D───E      \ /           │
│      (rewrite OK)                       X            │
│                                        / \           │
│      Feature 2                        /   G──H       │
│      (rewrite OK)                    /               │
└─────────────────────────────────────────────────────┘

Key Points:
• Main only grows forward (never rewritten)
• Features rebase/squash before merging
• Merge commits (M, I) preserve history
• No force pushes to main
```

---

## 5. Team Conflict Scenario

### What Happens with Force Push to Main

```
⏰ 9:00 AM - Everyone in sync
Alice:  A───B───C (main)
Bob:    A───B───C (main)
Remote: A───B───C (main)

⏰ 10:00 AM - Alice rewrites and force pushes
Alice:  A───B───C'──D' (main)
Remote: A───B───C'──D' (main)  ⚠️ History rewritten!
Bob:    A───B───C (main)       ⚠️ Now outdated!

⏰ 10:30 AM - Bob tries to push his work
Bob:    A───B───C───E (main)
Remote: A───B───C'──D' (main)

Git error:
! [rejected] main -> main (non-fast-forward)

        C───E (Bob's work)
       /
  A───B
       \
        C'──D' (Remote)
        
Histories diverged - conflict! 💥
```

---

## 6. The Right Way: Feature Branch Workflow

```
Day 1: Create feature
main:     A───B───C
               \
feature:        D──E──F──G──H
                WIP commits OK

Day 2: Clean up feature
main:     A───B───C
               \
feature:        D'──F'
                Clean commits

Day 3: Main gets updates
main:     A───B───C───I
               \
feature:        D'──F'
                Out of date!

Day 3: Rebase feature
main:     A───B───C───I
                   \
feature:            D''──F''
                    Up to date!

Day 4: Merge to main
main:     A───B───C───I─────M
                   \       /
feature:            D''──F''
                    
✅ No force push to main needed!
```

---

## 7. Command Decision Tree

```
Are you on main/master/develop?
│
├─ YES → ❌ Use only safe commands:
│         • git merge feature
│         • git pull
│         • git push
│         • git revert
│
└─ NO → Is it your personal feature branch?
         │
         ├─ YES → ✅ Rewrite freely:
         │         • git rebase -i
         │         • git commit --amend
         │         • git push --force-with-lease
         │
         └─ NO → ⚠️  It's shared:
                   • Coordinate with team
                   • Avoid force push
                   • Or make your own branch
```

---

## 8. Rebase vs Merge on Main

### ❌ Rebasing Main (DON'T)
```
Before:
main:     A───B───C
feature:  A───B───D───E

git checkout main
git rebase feature  ← REWRITES main history!

After:
main:     A───B───D───E───C'  ← C is rewritten to C'
                      
Problem: C' is different from C
         Everyone with C is now broken! 💥
```

### ✅ Merging to Main (DO)
```
Before:
main:     A───B───C
feature:  A───B───D───E

git checkout main
git merge feature  ← Safe merge

After:
main:     A───B───C───────M
               \         /
feature:        D───E───
                      
C stays unchanged
Everyone can pull M cleanly! ✅
```

---

## 9. Recovery from Accidental Force Push

```
Accident:
main:     A───B───C───D (good)
              ↓ force push
main:     A───B───X───Y (bad)

Recovery using reflog:
$ git reflog
a1b2c3d HEAD@{0}: commit: Y
x4y5z6e HEAD@{1}: commit: X
d7e8f9g HEAD@{2}: commit: D  ← Find this!

$ git reset --hard d7e8f9g
main:     A───B───C───D (restored!)

$ git push --force origin main
          A───B───C───D (fixed!)

✅ Original history restored
```

---

## 10. Common Patterns

### Pattern 1: Clean Feature Before Merge
```
1. Work messily:     feature: W─I─P─F─i─x─D
2. Squash:           feature: CLEAN
3. Merge to main:    main: ────M
                              /
                     feature: CLEAN
```

### Pattern 2: Keep Feature Updated
```
1. Main updates:     main: A─B─C───D
                            \
                     feature: E─F

2. Rebase feature:   main: A─B─C─D
                                 \
                     feature:     E'─F'

3. Continue work:    main: A─B─C─D
                                 \
                     feature:     E'─F'─G─H

4. Rebase again:     main: A─B─C─D
                                 \
                     feature:     E''─F''─G'─H'

5. Final merge:      main: A─B─C─D───────M
                                 \       /
                     feature:     E''─F''─G'─H'
```

---

## 11. Danger Signs

```
⚠️  WARNING SIGNS - STOP AND THINK:

1. "! [rejected] main -> main (non-fast-forward)"
   → You're about to rewrite shared history!

2. "git push --force"
   → Are you 100% sure this is YOUR branch?

3. "Your branch and 'origin/main' have diverged"
   → History has been rewritten somewhere!

4. Multiple people working on same branch
   → Coordinate before any rewriting!

5. "This branch is protected"
   → Good! It's working as designed.
```

---

## 12. Quick Reference Table

```
┌──────────────────┬─────────┬─────────┬─────────────┐
│ Command          │ Main    │ Feature │ Shared      │
│                  │ Branch  │ (yours) │ Feature     │
├──────────────────┼─────────┼─────────┼─────────────┤
│ commit --amend   │    ❌   │   ✅    │     ⚠️      │
│ rebase           │    ❌   │   ✅    │     ⚠️      │
│ rebase -i        │    ❌   │   ✅    │     ⚠️      │
│ reset --hard     │    ❌   │   ✅    │     ❌      │
│ push --force     │    ❌   │   ⚠️    │     ❌      │
│ force-with-lease │    ❌   │   ✅    │     ⚠️      │
│ merge            │    ✅   │   ✅    │     ✅      │
│ pull             │    ✅   │   ✅    │     ✅      │
│ push             │    ✅   │   ✅    │     ✅      │
│ revert           │    ✅   │   ✅    │     ✅      │
└──────────────────┴─────────┴─────────┴─────────────┘

Legend: ✅ Safe  ⚠️ Coordinate  ❌ Never
```

---

## Remember

```
┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓
┃  IF YOU'RE ABOUT TO FORCE PUSH TO MAIN, STOP!  ┃
┃                                                  ┃
┃  Ask yourself:                                   ┃
┃  1. Is this really necessary?                    ┃
┃  2. Does anyone else have this branch?           ┃
┃  3. Have I communicated with the team?           ┃
┃  4. Is there a safer alternative?                ┃
┃                                                  ┃
┃  When in doubt, DON'T force push.               ┃
┗━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┛
```
