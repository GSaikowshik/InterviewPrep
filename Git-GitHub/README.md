# Git & GitHub Master Reference & Version Control Engineering Guide

---

## 1. Git Internal Architecture: The Three Trees & Object Storage

Git is a distributed content-addressable version control system that stores snapshots of an entire project, not file deltas.

```mermaid
graph LR
    subgraph WorkingDirectory["1. Working Directory<br>(Sandbox: actual files on disk)"]
        WFiles["app.py, main.go"]
    end
    subgraph StagingArea["2. Staging Area / Index<br>(Pre-commit snapshot / manifest)"]
        Index["Index / Staged Changes"]
    end
    subgraph Repository["3. Git Directory (.git Repository)<br>(Object database & commit DAG)"]
        HEAD["HEAD -> refs/heads/main"]
        Commits["Commit DAG (Blobs, Trees, Commits)"]
    end

    WFiles --"git add"--> Index
    Index --"git commit"--> Commits
    Commits --"git checkout / restore"--> WFiles
    Index --"git restore --staged"--> WFiles
```

### Git Object Types
Every object in `.git/objects/` is compressed with zlib and addressed by its 40-character SHA-1 (or SHA-256) hash:
1. **Blob:** Stores raw file data without metadata or filename (two identical files share one blob).
2. **Tree:** Represents a directory; points to blobs and sub-trees with file permissions and filenames.
3. **Commit:** Contains a pointer to the top-level tree snapshot, author, committer, timestamp, commit message, and parent commit hash(es).
4. **Annotated Tag:** A permanent reference pointing to a specific commit, containing tagger metadata and a message.

---

## 2. Core Branching & Merging Mechanics

```mermaid
graph TD
    subgraph FastForward["Fast-Forward Merge (No Divergence)"]
        C1["C1"] --> C2["C2"] --> C3["C3 (main, feature)"]
    end
    subgraph ThreeWayMerge["3-Way Merge (Diverged History)"]
        M1["C1"] --> M2["C2"] --> M4["C4 (main)"]
        M2 --> M3["C3 (feature)"]
        M4 --> M5["C5 (Merge Commit with 2 Parents)"]
        M3 --> M5
    end
```

### Git Merge vs. Git Rebase

| Dimension | `git merge` | `git rebase` |
|---|---|---|
| **Mechanism** | Creates a new **Merge Commit** tying diverged branches together | Replays branch commits on top of base branch, creating new hashes |
| **History Shape** | Non-linear graph; preserves true chronological development context | Clean, linear timeline; looks like work was completed sequentially |
| **Commit Hashes** | Preserves all original commit SHAs | Generates brand-new commit hashes for all rebased commits |
| **Conflict Resolution**| Resolve conflicts once during the merge commit creation | May require resolving conflicts step-by-step for each replayed commit |
| **The Golden Rule**| Safe for shared public branches | **Never rebase a public branch shared with other developers** |

```bash
# Feature branch update via rebase (Clean linear history)
git checkout feature/dayzero-api
git fetch origin
git rebase origin/main

# Abort or continue during rebase conflict
git status
# (fix conflict markers in editor)
git add resolved_file.py
git rebase --continue
# or cancel:
git rebase --abort
```

---

## 3. Undoing Changes: `restore` vs. `reset` vs. `revert`

```mermaid
graph TD
    Undo["Undoing Operations"]
    Undo --> Working["Unstaged Local Changes"]
    Undo --> Staged["Staged Index Changes"]
    Undo --> CommittedLocal["Committed Local (Private)"]
    Undo --> CommittedPublic["Pushed to Public Remote"]

    Working -->|"git restore <file>"| Discard["Discard working tree edits"]
    Staged -->|"git restore --staged <file>"| Unstage["Unstage back to working tree"]
    CommittedLocal -->|"git reset (--soft / --mixed / --hard)"| Rewrite["Rewrite local commit history"]
    CommittedPublic -->|"git revert <commit_hash>"| SafeInverse["Create new inverse commit"]
```

### Git Reset Modes Comparison

| Command Flag | Working Directory | Staging Area (Index) | Commit History (HEAD) |
|---|---|---|---|
| `git reset --soft HEAD~1` | **Preserved** (Untouched) | **Preserved** (Changes stay staged) | Moved back 1 commit |
| `git reset --mixed HEAD~1` (Default)| **Preserved** (Untouched) | **Reset** (Changes become unstaged) | Moved back 1 commit |
| `git reset --hard HEAD~1` | **Wiped Out** (Data destroyed!) | **Wiped Out** (Data destroyed!) | Moved back 1 commit |

> **Best Practice:** When undoing commits already pushed to team remotes (`main`), **always use `git revert <hash>`**. It records an inverse patch without rewriting history, avoiding team synchronization chaos.

---

## 4. Advanced Git Utilities

### A. The Emergency Safety Net: `git reflog`
Git tracks every movement of `HEAD` in local reference logs. Even after an accidental `git reset --hard` or deleted branch, lost commits remain recoverable for ~30–90 days:
```bash
git reflog
# 7c4a102 HEAD@{0}: reset: moving to HEAD~1
# 9f21b88 HEAD@{1}: commit: Add critical DayZero AI backend router
git reset --hard 9f21b88  # Recovered!
```

### B. Context Switching: `git stash`
```bash
git stash save "WIP: mid-flight fastapi route"
git stash list
git stash apply stash@{0}  # Reapplies changes but keeps them in stash
git stash pop              # Reapplies changes and drops them from stash
git stash drop stash@{0}
```

### C. Surgical Commits: `git cherry-pick`
Applies the exact changes from an arbitrary commit from another branch onto your current `HEAD`:
```bash
git checkout main
git cherry-pick 4a7bc91
```

### D. Automated Debugging: `git bisect`
Performs binary search across commit history to pinpoint the exact commit that introduced a bug:
```bash
git bisect start
git bisect bad                 # Current commit is broken
git bisect good v1.0.0         # Commit v1.0.0 was known good
# Git checks out midpoint commits automatically:
pytest tests/
git bisect good  # or git bisect bad
# Bisect identifies: 3a28c11 is the first bad commit
git bisect reset
```

---

## 5. Master Git CLI Cheatsheet

```bash
# -----------------------------
# Configuration & Setup
# -----------------------------
git config --global user.name "Gandikota Sai Kowshik"
git config --global user.email "kowshik@example.com"
git init                                       # Initialize new repository
git clone <url>                                # Clone remote repository

# -----------------------------
# Everyday Workflow
# -----------------------------
git status -s                                  # Short concise status
git add .                                      # Stage all modified and new files
git add -p                                     # Stage interactively by hunk
git commit -m "feat(api): add auth endpoint"   # Commit staged changes
git commit --amend --no-edit                   # Append changes to previous commit
git log --oneline --graph --decorate --all     # ASCII graphical commit log
git diff                                       # Diff working tree vs staging area
git diff --staged                              # Diff staging area vs HEAD

# -----------------------------
# Branching & Remote Sync
# -----------------------------
git branch -M main                             # Rename active branch
git switch -c feature/new-flow                 # Create and switch to new branch
git switch main                                # Switch branches (modern syntax)
git merge <branch>                             # Merge branch into active branch
git remote add origin <url>                    # Link remote repository
git fetch origin                               # Download remote metadata without merging
git pull origin main --rebase                  # Pull and rebase local commits
git push -u origin main                        # Push and set upstream tracking

# -----------------------------
# Cleanup & Maintenance
# -----------------------------
git clean -fd                                  # Delete untracked files & directories
git branch -d feature/done                     # Safe delete merged branch
git branch -D feature/abandoned                # Force delete unmerged branch
```

## 6. Industry Branching Strategies: GitFlow vs. GitHub Flow vs. Trunk-Based

```mermaid
graph TD
    Strategies["Git Branching Models"]
    Strategies --> GitFlow["GitFlow<br>- Long-lived branches: main, develop, release/*, hotfix/*, feature/*<br>- Strict release gating; heavy merging overhead"]
    Strategies --> GitHubFlow["GitHub Flow<br>- main branch is always deployable<br>- Short-lived feature branches merged via Pull Requests (PRs)<br>- Ideal for web apps with continuous deployment"]
    Strategies --> TrunkBased["Trunk-Based Development<br>- Developers commit daily to 'trunk' (main)<br>- Uses Feature Flags / Toggles to hide incomplete features<br>- Highest developer velocity, zero merge drift"]
```

---

## 7. GitHub Actions CI/CD Pipeline Automation

```yaml
# .github/workflows/backend-ci.yml
name: Backend CI/CD Pipeline

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

jobs:
  test:
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:16-alpine
        env:
          POSTGRES_PASSWORD: secret
          POSTGRES_DB: testdb
        ports:
          - 5432:5432
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5

    steps:
      - name: Check out repository
        uses: actions/checkout@v4

      - name: Set up Python 3.12
        uses: actions/setup-python@v5
        with:
          python-version: "3.12"
          cache: "pip"

      - name: Install dependencies
        run: pip install -r requirements.txt pytest httpx flake8

      - name: Lint with Flake8
        run: flake8 . --count --select=E9,F63,F7,F82 --show-source --statistics

      - name: Run Pytest Test Suite
        env:
          DATABASE_URL: postgresql://postgres:secret@localhost:5432/testdb
        run: pytest --maxfail=1 --disable-warnings -v
```
