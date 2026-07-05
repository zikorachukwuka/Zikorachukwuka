# Gitbook - My Personal Git Reference

*Scan for your situation. Copy the command. Move on.*

---

## 🚀 Quick Jump

- [One-Time Setup](#one-time-setup)
- [Starting a Brand New Project](#starting-a-brand-new-project)
- [Daily Workflow (the 90% case)](#daily-workflow-the-90-case)
- [Branching](#branching)
- [Undoing Mistakes](#undoing-mistakes)
- [Stashing (save work without committing)](#stashing)
- [Working with Remotes](#working-with-remotes)
- [Viewing History](#viewing-history)
- [.gitignore & Secrets](#gitignore--secrets)
- [Deploying (Render / Railway / GitHub)](#deploying-render--railway--github)
- [🔥 "Oh No" Emergency Fixes](#-oh-no-emergency-fixes)
- [Glossary](#glossary)

---

## One-Time Setup

Do this once per machine, not per project.

```bash
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
git config --global init.defaultBranch main
git config --global core.editor "code --wait"   # makes VS Code your git editor
```

**Connect to GitHub with SSH** (do once, avoids typing a password every push):

```bash
ssh-keygen -t ed25519 -C "you@example.com"   # press Enter through prompts
cat ~/.ssh/id_ed25519.pub                    # copy this output
```

Paste the copied key into GitHub → Settings → SSH and GPG keys → New SSH key.
Test it:

```bash
ssh -T git@github.com
```

You should see "Hi username! You've successfully authenticated."

---

## Starting a Brand New Project

**Path A — You already have code on your computer, GitHub doesn't know about it yet:**

```bash
mkdir my-project && cd my-project
code .                          # opens the folder in VS Code
git init                        # turns this folder into a git repo
```

Create a `.gitignore` (see [.gitignore & Secrets](#gitignore--secrets)) and your first files, then:

```bash
git add .
git commit -m "Initial commit"
```

Now create an **empty** repo on GitHub (no README, no .gitignore — avoids merge conflicts), then:

```bash
git remote add origin git@github.com:username/my-project.git
git branch -M main
git push -u origin main
```

`-u` links your local `main` to GitHub's `main` so future pushes just need `git push`.

**Path B — The repo already exists on GitHub, you want it locally:**

```bash
git clone git@github.com:username/my-project.git
cd my-project
code .
```

---

## Daily Workflow (the 90% case)

This is the loop you'll run constantly:

```bash
git status                      # what changed? (run this often — it's free)
git add <file>                  # stage a specific file
git add .                       # stage everything changed
git commit -m "clear message"   # save a snapshot
git push                        # send it to GitHub
```

**Before you start working each session**, pull first to avoid conflicts:

```bash
git pull
```

**Good commit message habits:**
- `feat: add login form` — new feature
- `fix: correct null check on signup` — bug fix
- `refactor: simplify auth middleware` — no behavior change
- `chore: update dependencies` — housekeeping

---

## Branching

Branches let you work on something without touching your stable `main` code. Use them even solo — it's cheap insurance.

```bash
git branch                          # list local branches, * = current
git checkout -b feature/login       # create AND switch to new branch
git switch feature/login            # switch to an existing branch (modern alt to checkout)
git switch -c feature/login         # create + switch (modern alt)
```

**Bring your branch's work into main:**

```bash
git switch main
git pull                            # make sure main is current
git merge feature/login             # merge feature into main
git push
```

**Clean up after merging:**

```bash
git branch -d feature/login         # delete local branch (safe, checks it's merged)
git push origin --delete feature/login   # delete it on GitHub too
```

**Rename a branch:**

```bash
git branch -m old-name new-name
```

---

## Undoing Mistakes

Git has different "undo" tools depending on *how far* the mistake got.

| Situation | Command |
|---|---|
| Unstage a file (keep changes) | `git restore --staged <file>` |
| Discard changes in a file (not staged, not committed) | `git restore <file>` |
| Change your last commit message | `git commit --amend -m "new message"` |
| Add a forgotten file to your last commit | `git add <file>` → `git commit --amend --no-edit` |
| Undo last commit, **keep** changes staged | `git reset --soft HEAD~1` |
| Undo last commit, **keep** changes unstaged | `git reset --mixed HEAD~1` (this is the default for `git reset HEAD~1`) |
| Undo last commit, **delete** changes completely | `git reset --hard HEAD~1` ⚠️ destructive |
| Undo a commit that's already pushed/shared | `git revert <commit-hash>` (makes a new "undo" commit — safe for shared history) |

**Rule of thumb:** use `reset` on commits only you have (not pushed, or pushed but you're 100% sure no one else pulled). Use `revert` once something is shared — it doesn't rewrite history.

---

## Stashing

For when you're mid-work, need to switch branches, but aren't ready to commit.

```bash
git stash                     # shelve current changes, working directory becomes clean
git stash list                # see all stashes
git stash pop                 # reapply the most recent stash AND remove it from the list
git stash apply                # reapply without removing (if you want to apply it elsewhere too)
git stash drop                # delete a stash you don't need
```

---

## Working with Remotes

```bash
git remote -v                       # see what remote(s) you're connected to
git fetch                           # download changes but don't merge them yet
git pull                            # fetch + merge in one step (most common)
git push                            # send your commits to GitHub
git push -u origin <branch-name>    # first push of a new branch (sets upstream)
```

**Multiple remotes** (rare solo, but useful, e.g. deploying to Render via git):

```bash
git remote add render <render-git-url>
git push render main
```

---

## Viewing History

```bash
git log                             # full history
git log --oneline --graph --all     # compact visual view (bookmark this one)
git diff                            # unstaged changes vs last commit
git diff --staged                   # staged changes vs last commit
git show <commit-hash>              # full details of one commit
git blame <file>                    # who/what commit last touched each line
```

---

## .gitignore & Secrets

Create a `.gitignore` file **before your first commit** so secrets and junk never enter history.

Minimal Node/full-stack starter:

```
node_modules/
.env
.env.local
dist/
build/
.DS_Store
*.log
```

**If you already committed a `.env` or secret by accident:**

```bash
git rm --cached .env                # untrack it (keeps the local file)
echo ".env" >> .gitignore
git commit -m "chore: stop tracking .env"
git push
```

⚠️ The secret still exists in old commit history. If it was a real credential (API key, password), **rotate/revoke it** — removing the file doesn't erase it from history. For a solo project, that's usually enough; for anything public/shared, you'd need `git filter-repo` to scrub history (ask when you hit this).

---

## Deploying (Render / Railway / GitHub)

- Both platforms watch a GitHub branch (usually `main`) and auto-deploy on push. Your workflow doesn't change — just `git push` and check their dashboard.
- **Never commit `.env` files.** Set environment variables directly in the Render/Railway dashboard instead.
- Good practice: keep a `.env.example` file (committed) listing variable *names* with no real values, so future-you knows what's needed:

```
DATABASE_URL=
API_KEY=
```

- Consider a `staging` or `dev` branch that deploys to a preview environment, merging to `main` only when it's confirmed working — this is the "right hygiene" habit worth building now even solo.

---

## 🔥 "Oh No" Emergency Fixes

**Merge conflict appeared:**
```bash
# Git will mark conflicts in the file like:
# <<<<<<< HEAD
# your version
# =======
# their version
# >>>>>>> branch-name
```
Open the file in VS Code (it highlights conflicts with clickable "Accept Current/Incoming" buttons), pick the right content, delete the `<<<<<<<`/`=======`/`>>>>>>>` markers, then:
```bash
git add <file>
git commit
```

**Committed to the wrong branch:**
```bash
git branch new-correct-branch   # save current state to a new branch
git reset --hard HEAD~1         # remove commit from wrong branch (if not pushed)
git switch new-correct-branch   # continue work here
```

**"detached HEAD" message:**
You checked out a commit instead of a branch — you're not "on" any branch.
```bash
git switch -c temp-branch     # save your spot by creating a branch here
```

**Need to completely discard all local changes and match GitHub exactly:**
```bash
git fetch origin
git reset --hard origin/main   ⚠️ destroys all local uncommitted/unpushed work
```

**Pushed but need to force-update remote (only if you're SURE no one else pulled it):**
```bash
git push --force-with-lease   # safer than --force, fails if remote has new commits you don't have
```

**Accidentally deleted a branch:**
```bash
git reflog                    # shows recent HEAD movements, find the last commit hash of that branch
git branch recovered-branch <commit-hash>
```

---

## Glossary

| Term | Meaning |
|---|---|
| **Repo** | A project folder tracked by git |
| **Commit** | A saved snapshot of your code with a message |
| **Staging area** | The "waiting room" for changes before they become a commit (`git add` puts them here) |
| **Branch** | A parallel, independent line of work |
| **Remote** | A version of your repo hosted elsewhere (e.g. GitHub) — usually named `origin` |
| **Origin** | The default nickname for your main remote |
| **HEAD** | Pointer to your current commit/branch |
| **Merge** | Combining changes from one branch into another |
| **Clone** | Copying a remote repo to your machine |
| **Fork** | Your own GitHub copy of someone else's repo |
| **Upstream** | The branch your local branch is linked to for push/pull |

---

*Keep this file at the root of a personal notes repo, or pin it in VS Code, so it's always one click away.*
