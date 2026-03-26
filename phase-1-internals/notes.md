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

## What git is actually storing

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

## Key commands

```bash
# See which zone each file is in
git status

# Stage a file
git add <filename>

# Stage everything changed
git add .

# Commit what is staged
git commit -m "your message here"

# See commit history
git log --oneline
```

---

## Reading git diff output

`git diff` compares two zones. There are two versions you will use constantly:

```bash
git diff            # working tree vs staging area
                    # "what have I changed but not staged yet?"

git diff --staged   # staging area vs last commit
                    # "what am I about to commit?"
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

### Reading the hunk header `@@ -0,0 +1 @@`

Format: `@@ -[old_start],[old_lines] +[new_start],[new_lines] @@`

- `-0,0` means: in the old file, starting at line 0, there were 0 lines (file was empty)
- `+1` means: in the new file, starting at line 1, one line was added

A more complex example: `@@ -14,7 +14,9 @@` means the old file had 7 lines starting at line 14, and the new file has 9 lines starting at line 14 (2 lines were added in that section).

---

## Useful Linux commands for practicing

```bash
echo "some text" > file.md    # write (overwrites the file completely)
echo "more text" >> file.md   # append (adds to the end, safe)
nano file.md                  # open file in terminal editor
                              # Ctrl+O to save, Ctrl+X to exit
```

> Warning: `>` silently overwrites everything in the file. Use `>>` when you want to add to an existing file.

---

## Still to cover in Phase 1

- [ ] `git log` — reading history in detail
- [ ] `git restore` and `git checkout` — undoing changes in each zone
- [ ] `.gitignore` — telling Git which files to never track
- [ ] `git show` — inspecting a specific commit in detail

