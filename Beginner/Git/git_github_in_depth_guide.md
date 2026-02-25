# 🧠 Git & GitHub — In-Depth Knowledge Guide with Cross Questions

---

## 1. 🐙 GitHub

### Overview
GitHub is a cloud-based platform built on top of Git that provides hosting for repositories along with collaboration tools like Pull Requests, Issues, Actions (CI/CD), Projects, and Wikis.

### Key Concepts

#### Repositories
- **Public vs Private**: Public repos are visible to everyone; private repos are restricted.
- **Fork**: A personal copy of someone else's repository on your GitHub account.
- **Clone**: A local copy of a remote repository.

#### Pull Requests (PRs)
- A PR is a request to merge changes from one branch into another.
- Supports code reviews, inline comments, and required approvals.
- Can be linked to Issues using keywords: `Closes #42`, `Fixes #10`.
- **Draft PRs**: Work-in-progress PRs not ready for review.

#### GitHub Actions (CI/CD)
- Automate workflows using YAML files in `.github/workflows/`.
- Triggers: `push`, `pull_request`, `schedule`, `workflow_dispatch`.
- Jobs run on **runners** (GitHub-hosted or self-hosted).

```yaml
name: CI
on: [push]
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - run: echo "Hello, World!"
```

#### GitHub Features
| Feature | Purpose |
|---|---|
| Issues | Bug tracking & feature requests |
| Projects | Kanban-style project management |
| Wikis | Documentation |
| Packages | Host packages (npm, Docker, etc.) |
| Releases | Tag versions with changelogs |
| Gists | Share code snippets |

### ❓ Cross Questions
1. What is the difference between a **fork** and a **clone**?
2. How do you link a Pull Request to an Issue?
3. What is a **GitHub Action** and how does it differ from a traditional CI tool like Jenkins?
4. What happens when you delete the source branch after merging a PR?
5. How do you protect a branch on GitHub and what rules can you enforce?
6. What is `CODEOWNERS` and how does it work?
7. Explain the difference between **Squash and Merge**, **Rebase and Merge**, and **Create a Merge Commit** in GitHub PRs.
8. How do GitHub Actions secrets work, and where are they stored?

---

## 2. 🖥️ Git Graphical Tools

### Overview
Git GUIs provide visual interfaces to interact with repositories, making complex operations like branching, merging, and history visualization easier.

### Popular Tools

#### GitKraken
- Cross-platform Git GUI with a visual commit graph.
- Supports GitHub, GitLab, Bitbucket integration.
- Built-in merge conflict editor.

#### Sourcetree (Atlassian)
- Free tool for Windows & Mac.
- Great for visualizing branches and history.
- Supports Git Flow out of the box.

#### GitHub Desktop
- Simplified interface specifically for GitHub workflows.
- Best for beginners; limited advanced features.

#### VS Code Git Integration
- Built-in source control panel.
- Extensions: **GitLens** (blame annotations, history), **Git Graph** (visual branch graph).

#### `gitk` and `git gui`
- Built-in Git GUI tools installed with Git.
- `gitk`: Repository browser and history viewer.
- `git gui`: Commit tool.

```bash
gitk --all          # View all branches visually
git gui             # Open the commit GUI
```

### ❓ Cross Questions
1. What advantage does a Git GUI offer over the CLI?
2. In what scenarios would you prefer the CLI over a GUI?
3. How does **GitLens** enhance Git usage inside VS Code?
4. Can you perform an **interactive rebase** in a GUI tool? Which ones support it?
5. What is the visual representation of a "detached HEAD" in GitKraken?

---

## 3. ⚙️ Git Internals

### Overview
Git is a **content-addressable filesystem**. Everything is stored as objects identified by SHA-1 (now transitioning to SHA-256) hashes.

### The Four Object Types

#### 1. Blob
- Stores raw file content (not the filename).
- SHA-1 hash of: `blob <size>\0<content>`

#### 2. Tree
- Represents a directory.
- Contains pointers (SHA-1s) to blobs and other trees, along with filenames and permissions.

#### 3. Commit
- Points to a tree (root snapshot), parent commit(s), author, committer, and message.

```
commit <hash>
  tree <hash>
  parent <hash>
  author ...
  committer ...
  message
```

#### 4. Tag Object
- Annotated tags point to a commit object with extra metadata.

### The .git Directory
```
.git/
├── HEAD           → Points to current branch (ref or SHA)
├── config         → Repo-level config
├── objects/       → All blob/tree/commit/tag objects
│   ├── pack/      → Packed objects for efficiency
│   └── info/
├── refs/
│   ├── heads/     → Local branches
│   ├── remotes/   → Remote-tracking branches
│   └── tags/      → Tags
├── index          → Staging area (binary file)
├── COMMIT_EDITMSG → Last commit message
├── FETCH_HEAD     → Last fetched SHA
└── ORIG_HEAD      → Previous HEAD before risky operations
```

### How Git Stores Data
- Git is **snapshot-based**, not delta-based (unlike SVN).
- Each commit stores a full snapshot (via tree objects), but deduplicates blobs.
- **Pack files**: Git periodically runs `git gc` (garbage collection) to compress objects into pack files using delta compression.

### Plumbing vs Porcelain Commands
| Type | Examples |
|---|---|
| Porcelain (high-level) | `git commit`, `git add`, `git push` |
| Plumbing (low-level) | `git hash-object`, `git cat-file`, `git ls-files` |

```bash
# Explore internals
git cat-file -t <sha>        # Type of object
git cat-file -p <sha>        # Pretty-print object
git ls-tree HEAD             # List tree contents
git hash-object file.txt     # Compute SHA of a file
git count-objects -v         # Count loose objects
```

### ❓ Cross Questions
1. What are the four object types in Git and what does each store?
2. What is the difference between a **blob** and a **tree**?
3. Where is the staging area stored in `.git/`?
4. What does `HEAD` contain, and what is a **detached HEAD**?
5. How does Git achieve deduplication of unchanged files across commits?
6. What is the difference between **plumbing** and **porcelain** commands?
7. What is a **pack file** and when is it created?
8. How would you use `git cat-file` to inspect a commit object?
9. Git is described as a "content-addressable filesystem." What does this mean?
10. What happens in `.git/` when you run `git commit`?

---

## 4. ↩️ Undoing Changes

### Overview
Git provides multiple mechanisms to undo changes depending on whether they're unstaged, staged, committed, or pushed.

### Unstaged Changes
```bash
git restore <file>           # Discard working directory changes (Git 2.23+)
git checkout -- <file>       # Older equivalent
git clean -fd                # Remove untracked files and directories
```

### Staged Changes
```bash
git restore --staged <file>  # Unstage a file (keep changes in working dir)
git reset HEAD <file>        # Older equivalent
```

### Committed Changes (Local)

#### `git revert`
- Creates a **new commit** that undoes the changes of a previous commit.
- Safe for shared history.
```bash
git revert <commit-sha>
git revert HEAD~2..HEAD      # Revert last 2 commits
```

#### `git reset`
- Moves the branch pointer to a previous commit.
- **Three modes:**

| Mode | Index (Staging) | Working Dir | Use Case |
|---|---|---|---|
| `--soft` | Unchanged | Unchanged | Re-commit with different message |
| `--mixed` (default) | Reset | Unchanged | Unstage changes |
| `--hard` | Reset | Reset | Completely discard commits |

```bash
git reset --soft HEAD~1      # Undo last commit, keep staged
git reset --mixed HEAD~1     # Undo last commit, keep unstaged
git reset --hard HEAD~1      # Undo last commit, discard changes
```

#### `git commit --amend`
- Modifies the most recent commit (message or content).
- Rewrites history — don't use on pushed commits.
```bash
git add forgotten-file.txt
git commit --amend --no-edit
```

### Recovering Lost Commits
```bash
git reflog                   # View history of HEAD movements
git checkout <sha>           # Recover a specific state
git branch recover-branch <sha>  # Create branch from lost commit
```

### ❓ Cross Questions
1. What is the difference between `git revert` and `git reset`?
2. When should you use `--soft`, `--mixed`, and `--hard` reset?
3. How do you undo a `git reset --hard` that deleted commits?
4. What is `git reflog` and how does it differ from `git log`?
5. Can you undo a `git revert`? How?
6. What is the danger of using `git reset` on already-pushed commits?
7. How does `git restore` differ from `git checkout -- <file>`?
8. How would you revert a merge commit? What is the `--mainline` flag?
9. What happens to the index and working tree after `git reset --mixed`?

---

## 5. 🌿 Branching and Merge

### Overview
Branching allows parallel lines of development. A branch in Git is simply a lightweight pointer (a file containing a SHA-1) to a commit.

### Branch Operations
```bash
git branch                      # List local branches
git branch -a                   # List all branches (local + remote)
git branch feature-x            # Create branch
git switch feature-x            # Switch to branch (Git 2.23+)
git checkout -b feature-x       # Create + switch (classic)
git branch -d feature-x         # Delete branch (safe)
git branch -D feature-x         # Force delete
git branch -m old-name new-name # Rename branch
```

### Merging

#### Fast-Forward Merge
- Happens when the target branch has no new commits since the branch diverged.
- Simply moves the pointer forward. No merge commit created.
```bash
git merge feature-x             # Fast-forward if possible
git merge --no-ff feature-x     # Force a merge commit
```

#### Three-Way Merge
- When both branches have diverged, Git finds the **common ancestor** and creates a **merge commit** with two parents.

#### Merge Conflicts
- Occur when the same lines are modified in both branches.
```
<<<<<<< HEAD
your changes
=======
incoming changes
>>>>>>> feature-x
```
- Resolve manually → `git add <file>` → `git merge --continue`

### Rebasing
- Re-applies commits on top of another base commit.
- Creates a **linear history**.
```bash
git rebase main                  # Rebase current branch onto main
git rebase --interactive HEAD~3  # Interactive rebase (squash, edit, reorder)
```

#### Interactive Rebase Actions
| Action | Meaning |
|---|---|
| `pick` | Use commit as-is |
| `reword` | Change commit message |
| `edit` | Pause to amend commit |
| `squash` | Combine with previous commit |
| `fixup` | Like squash, discard message |
| `drop` | Remove commit |

### `git cherry-pick`
- Apply a specific commit from one branch to another.
```bash
git cherry-pick <sha>
git cherry-pick A..B            # Apply range of commits
```

### ❓ Cross Questions
1. What is the difference between **fast-forward** and **three-way merge**?
2. When would you prefer `rebase` over `merge`?
3. What is the "Golden Rule of Rebasing"? Why shouldn't you rebase public branches?
4. How do you abort a merge in progress?
5. What is `git cherry-pick` used for? Give a practical example.
6. How does `--no-ff` affect the commit history?
7. What does `MERGE_HEAD` in `.git/` indicate?
8. How do you squash the last 3 commits using interactive rebase?
9. Explain `git rerere` and when it's useful.
10. What is an **octopus merge** in Git?

---

## 6. 🏷️ Tags

### Overview
Tags mark specific points in history (typically releases). Unlike branches, tags don't move.

### Lightweight Tags
- Just a pointer (ref) to a commit — no extra metadata.
```bash
git tag v1.0                  # Create lightweight tag
git tag                       # List tags
git show v1.0                 # Show tag info
```

### Annotated Tags
- Stored as a full Git object with tagger name, email, date, and message.
- Preferred for releases.
```bash
git tag -a v1.0 -m "Release version 1.0"
git tag -a v1.0 <commit-sha>   # Tag a specific commit
```

### Pushing Tags
- By default, `git push` does **not** push tags.
```bash
git push origin v1.0           # Push specific tag
git push origin --tags         # Push all tags
git push origin --follow-tags  # Push annotated tags only
```

### Deleting Tags
```bash
git tag -d v1.0                # Delete locally
git push origin --delete v1.0  # Delete remotely
```

### Checking Out Tags
```bash
git checkout v1.0              # Detached HEAD state
git checkout -b release-fix v1.0  # Create branch from tag
```

### ❓ Cross Questions
1. What is the difference between a **lightweight** and **annotated** tag?
2. Why does `git push` not push tags by default?
3. How do you tag a commit that happened 3 commits ago?
4. What state does Git enter when you checkout a tag? What are the risks?
5. How do you list all tags matching a pattern (e.g., `v1.*`)?
6. What Git object type is an annotated tag?
7. How would you sign a tag using GPG?

---

## 7. 📦 Stash

### Overview
`git stash` temporarily shelves changes so you can switch context without committing incomplete work.

### Basic Usage
```bash
git stash                      # Stash tracked modified + staged files
git stash push -m "WIP: login feature"  # With a message
git stash -u                   # Include untracked files
git stash -a                   # Include untracked + ignored files
```

### Managing Stashes
```bash
git stash list                 # List all stashes (stash@{0}, stash@{1}...)
git stash show                 # Show diff of latest stash
git stash show -p stash@{1}    # Detailed patch of specific stash
```

### Applying Stashes
```bash
git stash pop                  # Apply latest stash + remove from stash list
git stash apply stash@{2}      # Apply specific stash (keep in list)
git stash drop stash@{1}       # Remove specific stash
git stash clear                # Remove all stashes
```

### Advanced
```bash
git stash branch feature-x     # Create branch from stash (avoids conflicts)
git stash push -- path/to/file # Stash a specific file
```

### How Stash Works Internally
- Stash creates two (or three) commits: one for the index state and one for the working directory state.
- Stored under `.git/refs/stash`.

### ❓ Cross Questions
1. What is the difference between `git stash pop` and `git stash apply`?
2. Does `git stash` save untracked files by default? How do you include them?
3. How would you stash only specific files?
4. What happens if you `git stash pop` and there's a conflict?
5. How does Git store stashes internally?
6. What is the use of `git stash branch`?
7. Can you stash changes on a detached HEAD? What happens?

---

## 8. 🌐 Remotes

### Overview
Remotes are references to remote repositories. `origin` is the conventional default name for the remote a repo was cloned from.

### Managing Remotes
```bash
git remote -v                        # List remotes with URLs
git remote add origin <url>          # Add a remote
git remote rename origin upstream    # Rename remote
git remote remove origin             # Remove remote
git remote set-url origin <new-url>  # Change URL
```

### Fetching and Pulling
```bash
git fetch origin                     # Download changes, don't merge
git fetch --all                      # Fetch all remotes
git pull                             # fetch + merge (or rebase with config)
git pull --rebase                    # fetch + rebase instead of merge
```

### Pushing
```bash
git push origin main                 # Push branch to remote
git push -u origin feature-x        # Set upstream tracking
git push --force-with-lease          # Safe force push (checks remote state first)
git push --force                     # Dangerous! Overwrites remote history
```

### Tracking Branches
- Remote-tracking branches: `origin/main`, `origin/feature-x`
- Local tracking: `git branch -u origin/main` sets upstream for a local branch.

```bash
git branch -vv                       # Show tracking info for all branches
```

### Working with Multiple Remotes
```bash
git remote add upstream https://github.com/original/repo.git
git fetch upstream
git merge upstream/main
```

### ❓ Cross Questions
1. What is the difference between `git fetch` and `git pull`?
2. What does `origin` represent in Git?
3. What is a **remote-tracking branch** (e.g., `origin/main`)?
4. Why is `--force-with-lease` safer than `--force`?
5. How do you set the upstream tracking branch for a local branch?
6. You cloned a repo and someone else pushed new changes. How do you update your local repo?
7. What happens when you `git push` without specifying a branch?
8. What is the purpose of having an `upstream` remote in an open-source workflow?
9. How do you delete a remote branch?
10. Explain `FETCH_HEAD` and when it's useful.

---

## 9. 🔀 Branching Strategies

### Overview
Branching strategies define conventions for how teams use branches to manage releases, features, and hotfixes.

---

### 9.1 Git Flow (Vincent Driessen)

#### Branch Types
| Branch | Purpose |
|---|---|
| `main` | Production-ready code |
| `develop` | Integration branch for features |
| `feature/*` | New features |
| `release/*` | Release preparation |
| `hotfix/*` | Urgent production fixes |

#### Workflow
```
develop → feature/login → develop → release/1.0 → main + develop
```

#### Pros
- Clear separation of concerns
- Good for scheduled releases

#### Cons
- Complex for small teams
- Long-lived branches cause merge conflicts
- Not ideal for continuous delivery

---

### 9.2 GitHub Flow

#### Branches
- Just `main` + short-lived feature branches.

#### Workflow
1. Create branch from `main`
2. Add commits
3. Open Pull Request
4. Review + discuss
5. Deploy (optional)
6. Merge into `main`

#### Pros
- Simple, fast
- Ideal for continuous deployment

#### Cons
- No explicit versioning/release process
- Requires mature CI/CD pipeline

---

### 9.3 GitLab Flow

Combines GitHub Flow with **environment branches** (`production`, `staging`) or **release branches**.

```
feature → main → pre-production → production
```

- Supports both **continuous delivery** and **versioned releases**.

---

### 9.4 Trunk-Based Development (TBD)

- All developers commit to a single branch: `trunk` (or `main`).
- Feature branches live for < 1 day.
- Uses **feature flags** to hide incomplete work.

#### Pros
- Eliminates long-lived branch problems
- Encourages small, frequent commits
- Best for high-performing teams (used by Google, Meta)

#### Cons
- Requires strong CI, testing, and feature flag infrastructure
- High discipline needed

---

### 9.5 Forking Workflow (Open Source)

- Contributors fork the repo to their own account.
- Changes are submitted via Pull Requests from their fork.
- Maintainers review and merge.

---

### Comparison Table

| Strategy | Complexity | Release Style | Best For |
|---|---|---|---|
| Git Flow | High | Scheduled releases | Enterprise, versioned software |
| GitHub Flow | Low | Continuous Deployment | SaaS, small teams |
| GitLab Flow | Medium | Flexible | Mixed environments |
| Trunk-Based | Low–Medium | Continuous | High-scale, DevOps-mature teams |
| Forking | Medium | Varies | Open source |

### ❓ Cross Questions
1. What are the five branch types in **Git Flow**?
2. When is **GitHub Flow** more suitable than **Git Flow**?
3. What is **Trunk-Based Development** and what practices does it rely on?
4. What is a **feature flag** and why is it important for TBD?
5. How does **GitLab Flow** differ from GitHub Flow?
6. In Git Flow, where do hotfixes branch from, and where do they merge into?
7. What is the risk of **long-lived branches** in any strategy?
8. How does the **Forking Workflow** ensure the main repo isn't polluted with bad commits?
9. How would you choose a branching strategy for a team deploying 10 times a day?
10. What is **ship/show/ask** branching model and how does it differ from strict PR-based workflows?

---

## 🧩 Mega Cross Questions (Mixed Topics)

1. You accidentally pushed sensitive data to GitHub. What steps do you take?
2. Explain the end-to-end lifecycle of a commit from `git add` to storage in `.git/objects/`.
3. How does `git bisect` work? What Git internals does it leverage?
4. What is the difference between `git merge --squash` and `git rebase --interactive`?
5. You're in the middle of a rebase and a conflict occurs. How do you resolve it and continue?
6. What are **shallow clones** (`--depth`)? When would you use them?
7. How does `git gc` help maintain repository performance?
8. What is the difference between `git log --oneline`, `git log --graph`, and `git reflog`?
9. How would you mirror a repository from GitHub to another Git hosting service?
10. What is **submodule** vs **subtree**? When would you use each?

---

*This guide covers Git & GitHub from surface to internals — from day-to-day commands to architecture. Master each section with the cross questions to build interview-ready, production-grade Git knowledge.*
