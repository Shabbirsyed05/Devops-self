# 🌿 Git Interview Guide — Complete Q&A

> 📚 Source: [iam-veeramalla/devops-cloud-interview-guide](https://github.com/iam-veeramalla/devops-cloud-interview-guide)
> 🎯 Purpose: Master every Git interview question with full explanations and scripts

---

## 📋 Table of Contents

| # | Topic |
|---|-------|
| [1](#-1-git-fork-vs-git-clone) | Git Fork vs Git Clone |
| [2](#-2-fork-instead-of-clone--scenario) | Fork Instead of Clone — Scenario |
| [3](#-3-git-fetch-vs-git-pull) | Git Fetch vs Git Pull |
| [4](#-4-which-do-you-use-more-fetch-or-pull) | Fetch vs Pull — Most Used |
| [5](#-5-git-merge-vs-rebase) | Git Merge vs Rebase |
| [6](#-6-git-branching-strategy) | Git Branching Strategy |
| [7](#-7-three-git-challenges) | Three Challenges with Git |
| [8](#-8-recent-git-challenge) | Recent Git Challenge |
| [9](#-9-git-merge-conflicts) | Git Merge Conflicts |
| [10](#-10-git-strategies--ours-and-theirs) | Ours and Theirs Strategies |
| [11](#-11-git-tags) | Git Tags |
| [12](#-12-combine-multiple-commits-squash) | Squash / Combine Commits |
| [13](#-13-10-most-used-git-commands) | 10 Most Used Git Commands |
| [14](#-14-gitignore--assume-unchanged) | Gitignore & assume-unchanged |
| [15](#-15-the-git-folder) | The .git Folder |
| [16](#-16-restore-deleted-git-folder) | Restore Deleted .git Folder |
| [17](#-17-remove-secrets-from-git-history) | Remove Secrets from Git |
| [Master Cheatsheet](#-master-cheatsheet) | Master Cheatsheet |

---

---

# 🔀 1. Git Fork vs Git Clone

**Q: What is the difference between `git fork` and `git clone`?**

| | Fork | Clone |
|---|---|---|
| **Level** | GitHub / GitLab (server-side) | Local machine |
| **Creates** | Copy of repo under your GitHub account | Local copy of any repo |
| **When used** | Contributing to repos you don't have write access to | Starting work on any repo |

```
Fork  → GitHub action → your-username/repo (on GitHub)
Clone → Terminal action → copy on your laptop
```

**The typical open-source workflow:**
```bash
# Step 1: Fork on GitHub (click the Fork button)
# Step 2: Clone YOUR fork locally
git clone https://github.com/your-username/the-repo.git
# Step 3: Make changes, push to your fork
git push origin bugfix
# Step 4: Submit Pull Request from your fork → original repo
```

### 🎙️ Answer
> *"Fork is a GitHub-level action that creates a copy of a repo under my GitHub account. Clone is a local action that downloads any repo to my machine. When I want to contribute to a repo I don't own, I fork it first (getting my own write-access copy), then clone my fork locally. Changes go to my fork, and then I submit a pull request to the original."*

---

---

# 🔀 2. Fork Instead of Clone — Scenario

**Q: Explain a scenario where you used fork instead of clone.**

**Scenario:** Contributing a bug fix to an organization's Helm chart repo without write access.

```bash
# ❌ What didn't work:
git clone https://github.com/org/devops-helm-project.git
# Push fails → no write access

# ✅ What worked:
# 1. Fork on GitHub → my-username/devops-helm-project
# 2. Clone my fork
git clone https://github.com/my-username/devops-helm-project.git
# 3. Create branch + fix
git checkout -b bugfix-helm-values
# ... make changes ...
# 4. Push to my fork
git push origin bugfix-helm-values
# 5. Submit PR: my-username/devops-helm-project → org/devops-helm-project
```

### 🎙️ Answer
> *"When fixing a bug in an organization's Helm chart, I didn't have write access to push directly. Forking gave me my own copy with full write access. I cloned my fork, created a branch, fixed the bug, pushed to my fork, and then submitted a pull request to the original org repo. Without forking, I couldn't have contributed at all."*

---

---

# 📥 3. Git Fetch vs Git Pull

**Q: What is the difference between `git fetch` and `git pull`?**

```
git pull = git fetch + git merge (in one step)
```

| | `git fetch` | `git pull` |
|---|---|---|
| **Downloads changes** | ✅ Yes | ✅ Yes |
| **Merges into branch** | ❌ No — you review first | ✅ Yes — automatic |
| **Safe?** | ✅ Very safe | ⚠️ Can cause surprise merges |
| **Use when** | Want control + review before merging | Ready to sync immediately |

```bash
# Fetch — see what changed without touching your branch
git fetch origin
git log origin/main --oneline   # review what's new
git merge origin/main           # merge only when ready

# Pull — one step (fetch + merge)
git pull origin main
```

### 🎙️ Answer
> *"`git fetch` downloads changes from the remote but doesn't touch my working branch — I can review before merging. `git pull` is shorthand for fetch + merge in one step. I use fetch when I want to inspect upstream changes before integrating, and pull for fast day-to-day syncing when I trust there won't be conflicts."*

---

---

# 🔄 4. Which Do You Use More: Fetch or Pull?

**Q: Which do you use more often and why?**

> **Answer: `git pull`** — for daily work. **`git fetch`** — before major integrations.

```bash
# Day-to-day:
git pull origin main       # fast, keep in sync

# Before merging a big feature:
git fetch origin
git diff origin/main       # inspect first
git merge origin/main      # then merge consciously
```

**Safety habit before pulling:**
```bash
git stash          # save local changes
git pull
git stash pop      # restore local changes
```

---

---

# 🌿 5. Git Merge vs Rebase

**Q: What is the difference between `git merge` and `git rebase`?**

```
Starting state:
A---B---C  (main)
     \
      D---E  (feature)
```

**After `git merge main` into feature:**
```
A---B---C  (main)
     \       \
      D---E---M  (feature, M = merge commit)
```
- Preserves all history ✅
- Creates a merge commit ← can clutter history

**After `git rebase main` on feature:**
```
A---B---C  (main)
             \
              D'---E'  (feature, commits rewritten on top of C)
```
- Clean, linear history ✅
- Rewrites commits (D → D', E → E') ← risky on shared branches

---

| Use `merge` when... | Use `rebase` when... |
|---|---|
| Collaborating on shared branches | Working alone / before raising a PR |
| Want to preserve full commit context | Want clean linear history |
| History safety is important | Cleaning up local commits |
| Long-lived branches | Short-lived feature branches |

```bash
# Merge
git checkout feature
git merge main

# Rebase
git checkout feature
git rebase main

# ⚠️ Never rebase branches others are working on — rewrites history
```

### 🎙️ Answer
> *"Merge creates a merge commit and preserves the complete history of both branches — safer for collaboration. Rebase re-applies commits on top of the target branch, creating a clean linear history — better for local cleanup before a PR. I rebase feature branches before raising a PR, and use merge for integrating into shared long-lived branches."*

---

---

# 🌲 6. Git Branching Strategy

**Q: Explain the branching strategy you used. Align with Kubernetes's approach.**

**4-branch model (mirrors Kubernetes):**

| Branch | Purpose | Who uses it |
|---|---|---|
| `main` | Stable, always deployable — no direct commits | Everyone via PRs |
| `feature/*` | New features — short-lived, squash before merge | Developers |
| `release/*` | Stabilization — bug fixes + docs only, no new features | QA + Release team |
| `hotfix/*` | Emergency production patches | On-call engineer |

```
                    feature/login
                   /             \
main ─────────────────────────────── main (merged via PR)
         \                      /
          release/1.4 ─────────
                \
                 hotfix/critical-auth-bug → merged to main + release/1.4
```

**Key rules:**
- Direct commits to `main` are **blocked** (branch protection rules)
- All merges via Pull Requests with mandatory code review
- `release/*` branches have code freeze — only bug fixes allowed
- Hotfixes merge back to **both** `main` AND the active `release/*` branch

### 🎙️ Answer
> *"We followed a four-branch strategy mirroring Kubernetes. `main` is always deployable — all changes go through PRs. Feature branches are short-lived — squashed and merged after review. Release branches are cut from main and only allow bug fixes, aligning with Kubernetes's code freeze policy. Hotfixes merge into both main and the active release branch so the fix reaches current and future releases."*

---

---

# ⚠️ 7. Three Git Challenges

**Q: Explain three challenges you faced with Git.**

### Challenge 1: Merge Conflicts from Blind `git pull`

```
Relied on git pull without checking upstream changes
→ Surprise merge conflicts in fast-moving branches
```
**Fix:** Switched to `git fetch` + review + manual merge for critical branches.

---

### Challenge 2: Messy History from Frequent Merges

```
Long-lived feature branch + frequent git merges
→ Dozens of merge commits cluttering git log
```
**Fix:** Started using `git rebase` locally before pushing, squashing related commits before PRs.

---

### Challenge 3: Fork vs Clone Confusion in Open Source

```
Cloned upstream repo directly → git push failed (no write access)
→ Realized needed to fork first, then clone the fork
```
**Fix:** Learned the fork → clone → PR workflow; now second nature.

---

---

# 🏢 8. Recent Git Challenge

**Q: Explain a recent challenge and how you solved it.**

**Challenge:** Standardizing branching across 100+ repos with inconsistent conventions — breaking CI/CD pipelines.

```
State before:
  Team A: main + dev
  Team B: master + feature
  Team C: no long-lived branches
  → CI fails, releases chaotic, no common PR rules
```

**Solution in 4 steps:**

```
Step 1: Audit → GitHub/GitLab API to scan all repos, document current state

Step 2: Design → trunk-based model:
  main = production-ready always
  release/x.y = stabilization
  feature/* = short-lived, rebase before merge

Step 3: Rollout → starter repo templates + branch protection rules
                  + GitHub Actions enforcement + team handbook

Step 4: Iterate → feedback loop, temporary exceptions during migration
```

**Result:** 95%+ repos aligned in 2 months. CI/CD reliability improved significantly.

---

---

# 🔧 9. Git Merge Conflicts

**Q: How do you handle merge conflicts in Git?**

**When conflicts occur:**
- Two branches modify the same lines in a file
- One branch deletes a file the other branch modifies
- A rebase applies commits that clash with existing changes

**Resolution workflow:**

```bash
# Step 1: Conflict identified
git merge feature-branch
# CONFLICT (content): Merge conflict in app.py

# Step 2: Open the file — Git marks the conflict
<<<<<<< HEAD
print("Hello from main")
=======
print("Hello from feature-branch")
>>>>>>> feature-branch

# Step 3: Edit the file to the correct final state (delete markers)
print("Hello from main")   # or combine both, or pick one

# Step 4: Mark resolved + complete
git add app.py
git commit              # if merging
# git rebase --continue  # if rebasing

# If you want to abort and start over:
git merge --abort
# or
git rebase --abort
```

**Tools for visual resolution:** VS Code conflict editor, GitKraken, IntelliJ.

### 🎙️ Answer
> *"When I hit a conflict, I identify the files with `git status`, open them to see the `<<<<<<<`/`=======`/`>>>>>>>` markers, edit to the correct final state, then `git add` the resolved files and commit or `git rebase --continue`. Merge conflicts aren't errors — they're Git asking me to make a decision when both branches changed the same thing."*

---

---

# ⚔️ 10. Git Strategies — Ours and Theirs

**Q: What are `ours` and `theirs` strategies in Git?**

| Strategy | Behavior | Command |
|---|---|---|
| **`ours`** (merge strategy) | Keeps current branch's content entirely — ignores incoming | `git merge -s ours feature-branch` |
| **`theirs`** (conflict resolution) | Accepts incoming branch's version of a conflicted file | `git checkout --theirs conflicted_file.txt` |

```bash
# ours — merge strategy (keeps your branch, discards incoming)
git merge -s ours old-dead-branch
# Result: merge recorded in history, but your branch content unchanged
# Use case: closing a dead branch without taking any of its changes

# theirs — per-file conflict resolution (accept incoming changes)
git checkout --theirs app.py
git add app.py
git rebase --continue
# Use case: their version is definitely correct, don't want to manually edit
```

> ⚠️ There is **no `-s theirs`** merge strategy — `theirs` only works at the file level during conflict resolution.

### 🎙️ Answer
> *"The `ours` merge strategy keeps your current branch's content when merging — useful for formally closing a dead branch without taking any of its changes. `theirs` isn't a merge strategy, but during conflict resolution I can run `git checkout --theirs <file>` to accept the incoming branch's version of a specific conflicted file. Both help make conflict resolution more precise and automated."*

---

---

# 🏷️ 11. Git Tags

**Q: Have you used Git tags? Why?**

> Tags mark specific commits as important milestones — most commonly production releases.

```bash
# Annotated tag (recommended — includes message + author + date)
git tag -a v1.0.3 -m "Release v1.0.3 — critical auth bug fix"
git push origin v1.0.3           # push specific tag
git push origin --tags           # push all tags

# Lightweight tag (just a pointer, no metadata)
git tag v1.0.3

# List all tags
git tag

# Check out a specific version
git checkout v1.0.3

# Delete a tag
git tag -d v1.0.3
git push origin --delete v1.0.3
```

**Why tags matter in DevOps:**
- CI/CD pipelines trigger builds/deployments on tag events
- Rollback: `git checkout v1.0.2` returns to a known-good state
- `git log v1.0.2..v1.0.3` shows exact changes between releases

### 🎙️ Answer
> *"I use annotated Git tags to mark every production release — like `v1.0.3`. Jenkins pipelines trigger Docker image builds and Kubernetes deployments when a new tag is pushed. Tags make it easy to roll back — I just check out the previous tag. They're also used for changelog generation between releases: `git log v1.0.2..v1.0.3 --oneline`."*

---

---

# 🗜️ 12. Combine Multiple Commits (Squash)

**Q: How do you combine multiple commits into a single commit?**

> Use **interactive rebase** to squash commits before pushing or raising a PR.

```bash
# View recent commits
git log --oneline
# abc123 Fix typo
# def456 Add input validation
# ghi789 Update error message
# jkl012 Initial work on login form

# Squash last 4 commits into 1
git rebase -i HEAD~4
```

**In the editor that opens:**
```
pick jkl012 Initial work on login form
squash ghi789 Update error message      ← change pick → squash (or 's')
squash def456 Add input validation
squash abc123 Fix typo
```

```bash
# Save → editor opens for new commit message
# Write: "Add login form with validation and error handling"
# Save → done

# If already pushed, force push is required
git push origin feature-branch --force
```

> ⚠️ Only force-push on **your own feature branches** — never on shared branches.

### 🎙️ Answer
> *"I use `git rebase -i HEAD~N` to squash N commits. In the editor, I change all but the first `pick` to `squash`, then write a single clean commit message. This is my standard practice before raising a PR — reviewers see one meaningful commit instead of a dozen 'fix typo' commits. It keeps the main branch history clean and bisectable."*

---

---

# 🛠️ 13. 10 Most Used Git Commands

**Q: Explain 10 Git commands you use daily.**

```bash
# 1. git clone — download a repo to your machine
git clone https://github.com/org/repo.git

# 2. git status — see what's staged, unstaged, untracked
git status

# 3. git add — stage specific files or everything
git add file.py
git add .

# 4. git commit — save a snapshot with a message
git commit -m "Fix login bug"

# 5. git push — upload local commits to remote
git push origin feature-branch

# 6. git pull — fetch + merge remote changes (stay in sync)
git pull origin main

# 7. git fetch — download changes without merging (review first)
git fetch origin

# 8. git branch — list or create branches
git branch                        # list all branches
git branch feature/payment        # create new branch

# 9. git checkout — switch branches or create + switch
git checkout main
git checkout -b hotfix/critical   # create and switch in one step

# 10. git rebase — re-apply commits on top of another branch
git rebase main                   # clean history before PR
```

---

---

# 🙈 14. Gitignore & assume-unchanged

**Q: How do you ignore pushing changes to a specific file?**

**Two scenarios — two different solutions:**

### Scenario A: File not yet tracked → use `.gitignore`

```bash
# Add to .gitignore BEFORE first commit
echo ".env" >> .gitignore
echo "*.log" >> .gitignore

# If already committed once, untrack it first:
git rm --cached .env
echo ".env" >> .gitignore
git commit -m "Stop tracking .env"
```

### Scenario B: File already tracked → use `assume-unchanged`

```bash
# Tell Git: "pretend this file never changes"
git update-index --assume-unchanged config/local.json

# To resume tracking:
git update-index --no-assume-unchanged config/local.json
```

> ⚠️ `assume-unchanged` is **local only** — won't prevent teammates from seeing/pushing the file. It's a personal convenience flag, not a security mechanism.

**Use cases for `assume-unchanged`:**
- `.env` files with local credentials
- `settings.json` with IDE or environment-specific config
- Scripts temporarily tweaked for local testing

### 🎙️ Answer
> *"For files that should never be tracked, I add them to `.gitignore` before first commit — or untrack + add to gitignore if already committed. For files that are tracked but I don't want to accidentally include in my commits (like local config), I use `git update-index --assume-unchanged`. It's a local flag that tells Git to ignore changes to that specific file on my machine only."*

---

---

# 📂 15. The .git Folder

**Q: What is the purpose of the `.git` folder?**

> The `.git` directory is the **entire database of your Git repository**. Without it, the directory is just a regular folder.

```bash
ls .git/
# HEAD     ← current branch reference ("where you are")
# config   ← repo settings (remote URLs, user info, aliases)
# objects/ ← all commits, file contents (blobs), trees — compressed
# refs/    ← branch + tag pointers (refs/heads/main, refs/tags/v1.0)
# logs/    ← history of ref updates (used by git reflog)
# index    ← the staging area (files after git add, before git commit)
```

```bash
cat .git/HEAD
# ref: refs/heads/main     ← tells Git you're on the main branch

cat .git/config
# [remote "origin"]
#   url = https://github.com/user/repo.git
```

**What happens if you delete `.git`:**
```bash
rm -rf .git
git status
# fatal: not a git repository ← project becomes a plain directory
# ALL version history is gone
```

### 🎙️ Answer
> *"The .git folder is the repository's internal database. It contains HEAD (what branch you're on), the objects/ directory (every commit, file version, and tree structure), refs/ (branch and tag pointers), index (the staging area), and config (remote URLs, user info). Without .git, the project is just a folder with no version history. Accidentally deleting it wipes all your git history."*

---

---

# 🔁 16. Restore Deleted .git Folder

**Q: Can you restore a deleted `.git` folder?**

> **Short answer:** Only if you have a remote backup or an explicit backup of the folder.

### Option 1: Remote exists (most common — always push!)

```bash
# Move current code to safety
mkdir ../backup && cp -r * ../backup/

# Clone fresh from remote (restores full history)
git clone https://github.com/user/repo.git

# Move your uncommitted files back in
cp -r ../backup/* repo/

# Recommit your uncommitted work
git add . && git commit -m "Restore uncommitted work"
```

### Option 2: You have a .git backup

```bash
mv .git.bak .git
git status   # full history restored ✅
```

### Option 3: File recovery tools (last resort, rare success)

`extundelete`, `recuva` — only works if disk sectors not yet overwritten. Unreliable.

**Prevention best practices:**
- Push frequently — remote = backup
- Use IDE/Git GUI (reduces accidental `rm -rf` risk)
- Enable automatic snapshots (Time Machine, AWS EBS snapshots)

### 🎙️ Answer
> *"Once .git is deleted with no remote and no backup, the history is gone. The safest recovery is to clone from the remote — which is why I push frequently. If there's a remote, I clone fresh and manually reapply any uncommitted local changes. This is also why I never use raw `rm -rf` without double-checking the path first."*

---

---

# 🔐 17. Remove Secrets from Git History

**Q: A teammate committed a Kubernetes Secret (base64 encoded) to Git. What do you do?**

> **Base64 is NOT encryption.** Anyone can run `echo <value> | base64 --decode` and read the secret.

### Step-by-step response:

### Step 1: Rotate the compromised secret IMMEDIATELY

```bash
# Assume it's compromised — generate a new value
kubectl create secret generic my-secret \
  --from-literal=password=NEW_SECURE_PASSWORD \
  --dry-run=client -o yaml | kubectl apply -f -
```

### Step 2: Remove from Git history (not just latest commit)

```bash
# If only in the latest commit:
git reset HEAD~1
git restore --staged secret.yaml
rm secret.yaml
git commit -m "Remove accidentally committed secret"

# If in multiple commits (full history purge):
# Option A: git filter-repo (recommended)
git filter-repo --path secret.yaml --invert-paths
git push --force

# Option B: BFG Repo-Cleaner
bfg --delete-files secret.yaml
git push --force
```

> Everyone who cloned the repo needs to re-clone after a force push.

### Step 3: Prevent recurrence

```bash
# Add to .gitignore
echo "secrets.yaml" >> .gitignore
echo "*.key" >> .gitignore

# Use pre-commit hooks to scan for secrets
pip install pre-commit
# Add git-secrets, detect-secrets, or truffleHog to pre-commit config
```

**Better secret management alternatives:**
- Kubernetes Sealed Secrets (encrypted at rest)
- AWS Secrets Manager / HashiCorp Vault + External Secrets Operator
- Never store raw K8s Secrets in Git — they're only base64 encoded, not encrypted

### 🎙️ Answer
> *"First, rotate the secret immediately — assume it's compromised. Then remove it from the entire Git history using `git filter-repo --path secret.yaml --invert-paths` and force-push. Add the file to `.gitignore` and set up pre-commit hooks with tools like `detect-secrets` or `git-secrets` to prevent it from happening again. Long-term, I'd recommend using Sealed Secrets or an External Secrets Operator that pulls from Vault or AWS Secrets Manager — nothing sensitive should ever live in a Git repo, even encrypted."*

---

---

# 📌 Master Cheatsheet

```
╔══════════════════════════════════════════════════════════════════════╗
║               GIT INTERVIEW MASTER CHEATSHEET                        ║
╠══════════════════════════════════════════════════════════════════════╣
║                                                                      ║
║  FORK vs CLONE:                                                      ║
║  Fork = GitHub-level copy under your account (for contributing)     ║
║  Clone = local download of any repo                                  ║
║  Open source flow: Fork → Clone fork → PR back to original          ║
║                                                                      ║
║  FETCH vs PULL:                                                      ║
║  fetch = download only (review before merging)                       ║
║  pull = fetch + merge (fast, but can surprise you)                   ║
║                                                                      ║
║  MERGE vs REBASE:                                                    ║
║  merge = creates merge commit, preserves full history               ║
║  rebase = rewrites commits on top of target (clean linear history)  ║
║  Use merge on shared branches | Use rebase before PR                 ║
║  ⚠️ Never rebase branches others are working on                     ║
║                                                                      ║
║  BRANCHING STRATEGY:                                                 ║
║  main = always deployable (protected, PRs only)                     ║
║  feature/* = short-lived, squash before merge                       ║
║  release/* = stabilization (bug fixes only, code freeze)            ║
║  hotfix/* = emergency patch → merge to main + release/*             ║
║                                                                      ║
║  CONFLICT RESOLUTION:                                                ║
║  git merge --abort | git rebase --abort (start over)                ║
║  git checkout --ours file (keep your version)                       ║
║  git checkout --theirs file (accept incoming version)               ║
║  git merge -s ours branch (keep current, discard incoming)          ║
║                                                                      ║
║  TAGS:                                                               ║
║  git tag -a v1.0.3 -m "msg" | git push origin v1.0.3              ║
║  CI/CD triggers on tags | git checkout v1.0.2 for rollback         ║
║                                                                      ║
║  SQUASH COMMITS:                                                     ║
║  git rebase -i HEAD~N → change 'pick' to 'squash'                  ║
║  git push --force (only on your own branches)                       ║
║                                                                      ║
║  GITIGNORE:                                                          ║
║  .gitignore = for untracked files                                   ║
║  git update-index --assume-unchanged file = local ignore only       ║
║  git rm --cached file = untrack already-committed file              ║
║                                                                      ║
║  .GIT FOLDER:                                                        ║
║  HEAD=current branch | objects/=all commits | refs/=branch pointers║
║  index=staging area | Deleting .git = losing ALL history            ║
║  Recovery: only possible if remote exists                            ║
║                                                                      ║
║  SECRETS IN GIT:                                                     ║
║  Base64 ≠ encryption (anyone can decode it)                         ║
║  Rotate secret → filter-repo/BFG to purge history → force push     ║
║  Prevention: .gitignore + pre-commit hooks + Vault/Sealed Secrets   ║
╚══════════════════════════════════════════════════════════════════════╝
```

---

## 🗣️ Universal Interview Delivery Tips

| ✅ DO | ❌ DON'T |
|---|---|
| Start with the **Problem → Solution** structure | Jump straight to commands |
| Explain **why** rebase risks on shared branches | Just say "rebase rewrites history" |
| Say "rotate first" for secrets question | Say "just delete the commit" |
| Mention **pre-commit hooks** for secret prevention | Stop at "add to .gitignore" |
| Use the **fork → clone → PR** sequence clearly | Conflate fork and clone |

---

## 📚 Resources

- 🔗 [Git Docs](https://git-scm.com/docs)
- 🔗 [iam-veeramalla/devops-cloud-interview-guide](https://github.com/iam-veeramalla/devops-cloud-interview-guide)
- 🔗 [git filter-repo](https://github.com/newren/git-filter-repo)
- 🔗 [BFG Repo-Cleaner](https://rtyley.github.io/bfg-repo-cleaner/)
- 🔗 [pre-commit framework](https://pre-commit.com/)

---

> ⭐ **Star this repo** if it helped you prepare for your DevOps interview!
> 🔔 Paste the next topic's notes — I'll overwrite with only those!
