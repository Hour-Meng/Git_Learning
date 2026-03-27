# Phase 1 — Git internals

## The three zones

Every file you work with lives in one of three places at any time.

| Zone | Also called | Where it lives | What it means |
|------|-------------|----------------|---------------|
| Working tree | Working directory | Your actual files on disk | Files you can open and edit freely |
| Staging area | The index | `.git/index` | A draft of your next commit |
| Local repository | The repo | `.git/` folder | The permanent, full history of all commits |

The flow is always in one direction:

```
Working tree  -->  Staging area  -->  Local repository
              git add             git commit
```

And you can move back the other way with:

```
Local repository  -->  Working tree
                  git restore / git checkout
```

---

## What Git is actually storing

Git does not store diffs (lists of changes). It stores **complete snapshots** of every tracked file at the time of each commit.

Each commit object contains:
- A pointer to the full snapshot (called a "tree")
- A pointer to the previous commit (the "parent")
- Author name, email, and timestamp
- The commit message

This is why Git is fast — jumping to any point in history means switching to a stored snapshot, not replaying changes.

---

## HEAD

`HEAD` is a pointer to the commit you are currently on. Most of the time it points to a branch name (like `main`), which itself points to the latest commit on that branch.

```
HEAD → main → commit abc123 → commit def456 → ...
```

If you ever see `detached HEAD state`, it means HEAD is pointing directly at a commit instead of a branch. It is not dangerous — it just means you are not on any named branch right now.

---

## The staging area is a point-in-time snapshot

When you run `git add`, Git takes a snapshot of the file **at that exact moment**. If you edit the file again after staging, the staging area still holds the old version.

```
1. Edit file       → working tree has v2, staging area is empty
2. git add         → staging area now holds v2
3. Edit file again → working tree has v3, staging area still holds v2
4. git commit      → commits v2, NOT v3
```

After this, `git status` will still show the file as modified because v3 in the working tree does not match v2 that was committed.

---

## git status — checking which zone your files are in

`git status` is the command you run constantly. It tells you exactly which zone each changed file is in.

```bash
git status
```

### Reading the output

```
On branch main

Changes to be committed:        ← these are in the STAGING AREA
  (use "git restore --staged <file>..." to unstage)
        modified:   notes.md

Changes not staged for commit:  ← these are in the WORKING TREE
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes)
        modified:   feature-log.md

Untracked files:                ← Git has never seen these files before
  (use "git add <file>..." to include in what will be committed)
        secrets.txt
```

Three sections, three states:
- **Changes to be committed** — staged, will go into your next commit
- **Changes not staged for commit** — modified in working tree, not yet staged
- **Untracked files** — brand new files Git has never tracked

### Key commands shown inside git status output

Git is helpful — it literally tells you which command to use next inside the output. Read it carefully instead of ignoring it.

---

## git add — staging files

```bash
git add <filename>         # stage one specific file
git add .                  # stage everything changed in current folder
git add -p                 # stage interactively, chunk by chunk
                           # useful when one file has multiple unrelated changes
                           # and you want to split them into separate commits
```

### The -p flag (patch mode)

`git add -p` breaks the file into hunks and asks you what to do with each one:

```
Stage this hunk [y,n,q,a,d,s,?]?
```

| Key | What it does |
|-----|-------------|
| `y` | Stage this hunk |
| `n` | Skip this hunk |
| `s` | Split into smaller hunks |
| `q` | Quit, stop staging |
| `a` | Stage this hunk and all remaining hunks in the file |
| `?` | Show help |

---

## git diff — comparing zones

```bash
git diff                   # working tree vs staging area
                           # "what have I changed but not staged yet?"

git diff --staged          # staging area vs last commit
                           # "what am I about to commit?"

git diff HEAD              # working tree vs last commit
                           # "everything changed since my last commit"

git diff <hash1> <hash2>   # compare any two commits directly
```

### Anatomy of a diff

```diff
diff --git a/notes.md b/notes.md     ← a/ = before, b/ = after
index e69de29..8a2b2bd 100644        ← internal object hashes (e69de29 = empty file)
--- a/notes.md                       ← --- represents the old version
+++ b/notes.md                       ← +++ represents the new version
@@ -0,0 +1 @@                       ← hunk header: where in the file the change is
+# Phase 1 Notes                     ← + = line added, - = line removed, no symbol = unchanged
```

### Reading the hunk header

Format: `@@ -[old_start],[old_lines] +[new_start],[new_lines] @@`

```
@@ -0,0 +1 @@
   old: starting at line 0, 0 lines existed (file was empty)
   new: starting at line 1, 1 line added

@@ -14,7 +14,9 @@
   old: starting at line 14, there were 7 lines
   new: starting at line 14, there are now 9 lines (2 lines added)
```

### Line symbols

| Symbol | Meaning |
|--------|---------|
| `+` | Line was added in the new version |
| `-` | Line was removed from the old version |
| (none) | Unchanged context line, shown so you know where you are |

---

## git commit — saving a snapshot

```bash
git commit -m "your message"        # commit with an inline message
git commit                          # opens your editor to write a longer message
git commit --amend                  # edit the last commit message (before pushing)
                                    # also adds any currently staged changes into it
```

### Writing good commit messages

A good message completes the sentence: *"If applied, this commit will..."*

```
# Good — describes what the commit does
feat: add user login form
fix: correct typo in README
phase-1: document git diff anatomy

# Bad — too vague to be useful later
update
fix stuff
wip
```

---

## git log — reading history

```bash
git log                          # full log with author, date, message
git log --oneline                # short hash + message only (daily driver)
git log --oneline --graph        # visual branch tree (essential in Phase 2)
git log --oneline --all          # shows commits on ALL branches
git log -p                       # full diff introduced by each commit
git log --oneline -5             # last 5 commits only
git log --oneline -- <file>      # commits that touched a specific file
git show HEAD                    # inspect the most recent commit in full
git show <hash>                  # inspect any specific commit by hash
```

### The pager — less

When output is long, Git opens it inside a program called `less`. This is not a bug and you are not stuck. It is a pager — a program that shows long output one screen at a time.

**The key you need most: `q` to quit.**

| Key | Action |
|-----|--------|
| `q` | Quit and return to terminal |
| `Space` | Page down (one full screen) |
| `b` | Page back up |
| `Enter` or `j` | Scroll down one line |
| `k` | Scroll up one line |
| `g` | Jump to the very top |
| `G` | Jump to the very bottom |
| `/searchterm` | Search forward for text |
| `n` | Jump to next search result |
| `N` | Jump to previous search result |

### If you want to disable the pager permanently for a command

```bash
git config --global pager.log false     # disable pager for git log only
```

Not recommended to disable it globally — long diffs become unreadable without it.

---

## git restore — undoing mistakes

This is where most beginners get confused because there are multiple undo tools and each one targets a different zone.

### The rule: the right tool depends on which zone the mistake is in

```
ZONE 1 — Working tree (not staged yet)
  You edited a file but have not run git add.
  Command: git restore <file>
  Effect:  Permanently discards the change. Cannot be recovered.

ZONE 2 — Staging area (staged but not committed)
  You ran git add but have not committed yet.
  Command: git restore --staged <file>
  Effect:  Moves the file back to the working tree. Changes are NOT lost.

ZONE 3 — Already committed
  You ran git commit.
  Command: git reset or git revert (covered in Phase 2)
```

### What git restore --staged actually does

This is the one that causes confusion. It does exactly one thing:

```
Before git restore --staged:
  working tree  →  (your edit is here too, or not — doesn't matter)
  staging area  →  file with your change   ← this is what gets moved
  last commit   →  original file

After git restore --staged:
  working tree  →  file with your change   ← lands here instead
  staging area  →  (empty, nothing staged)
  last commit   →  original file           ← untouched
```

Your change is NOT gone. It moved from the staging area back to the working tree. You are back to the state before you ran `git add`.

### Common mistake — thinking git restore --staged undoes a commit

It does not touch commits at all. It only moves files between the staging area and working tree. If you already committed and want to undo that, you need `git reset`.

### Full undo reference table

| Situation | Command | Changes lost? |
|-----------|---------|---------------|
| Edited file, not staged | `git restore <file>` | Yes — permanent |
| Staged file, not committed | `git restore --staged <file>` | No — back to working tree |
| Committed, want to undo but keep changes | `git reset --soft HEAD~1` | No — back to staging area |
| Committed, want to wipe everything | `git reset --hard HEAD~1` | Yes — permanent |
| Committed and pushed, need safe undo | `git revert <hash>` | No — creates new undo commit |

> `HEAD~1` means "one commit before HEAD." `HEAD~2` means two commits back, and so on.

---

## .gitignore — telling Git what to never track

Create a file named `.gitignore` in the root of your repo. Each line is a pattern. Files matching any pattern are completely invisible to Git — they will not appear in `git status`, cannot be staged, and will never be committed.

```bash
nano .gitignore
```

### Pattern syntax

| Pattern | What it ignores |
|---------|----------------|
| `secrets.txt` | That exact filename |
| `*.log` | Any file ending in `.log` |
| `node_modules/` | The entire folder and everything inside it |
| `**/.env` | `.env` files anywhere in the project, at any depth |
| `!important.log` | Exception — do NOT ignore this specific file |
| `logs/*` | Everything inside logs/ but not the folder itself |

### Standard .gitignore for most projects

```
# OS files
.DS_Store
Thumbs.db

# Editor settings
.vscode/
.idea/

# Logs
*.log

# Environment variables — never commit passwords or API keys
.env

# Dependency folders
node_modules/
```

### The critical rule — already-tracked files

`.gitignore` only works on files Git has **never tracked before**. If you already committed a file and then add it to `.gitignore`, Git will keep tracking it. The ignore rule is silently ignored for already-tracked files.

To fix this:

```bash
# Stop tracking the file but keep it on disk
git rm --cached <filename>
git commit -m "remove <filename> from tracking"
```

After that commit, `.gitignore` will take effect going forward.

---

## Useful Linux commands for this workflow

```bash
echo "some text" > file.md    # write — WARNING: overwrites entire file silently
echo "more text" >> file.md   # append — adds to the end, safe
nano file.md                  # open file in terminal editor
```

### nano key reference

| Key | Action |
|-----|--------|
| `Ctrl+O` | Save file |
| `Ctrl+X` | Exit nano |
| `Ctrl+K` | Cut current line |
| `Ctrl+U` | Paste cut line |
| `Ctrl+W` | Search in file |

> Always use `>>` when you want to add to an existing file. A single `>` will silently delete everything already in the file.

---

## Phase 1 checklist

- [x] The three zones — working tree, staging area, local repo
- [x] What Git actually stores (snapshots, not diffs)
- [x] HEAD and what detached HEAD state means
- [x] The staging area is a point-in-time snapshot, not a live link
- [x] git status — reading which zone each file is in
- [x] git add — staging files including patch mode
- [x] Reading git diff output and hunk headers
- [x] git log variations and navigating the pager (less)
- [x] git restore — undoing changes, the difference between staged and working tree
- [x] .gitignore — patterns and the already-tracked file rule

