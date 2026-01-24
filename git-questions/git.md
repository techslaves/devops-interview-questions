# Git Interview Questions and Answers

This document contains common interview questions related to Git and their answers.

### 1. What is Git and why is it used?
**Answer:**
Git is a **Distributed Version Control System (DVCS)** used to track changes in source code during software development.
*   **Distributed:** Every developer has a full copy of the codebase and history.
*   **Performance:** Most operations (commit, diff, log) are local and fast.
*   **Branching:** Lightweight branching allows for easy feature isolation and context switching.
*   **Collaboration:** Facilitates multiple developers working on the same project simultaneously.

### 2. Difference between Git and GitHub?
**Answer:**

| Feature | Git | GitHub |
| :--- | :--- | :--- |
| **Type** | Version Control System (Software). | Hosting Service (Web Platform). |
| **Installation** | Installed locally on your machine. | Accessed via web browser. |
| **Function** | Tracks file changes and history. | Hosts repositories, provides UI, PRs, and CI/CD actions. |
| **Dependency** | Works without GitHub. | Needs Git to push/pull code. |

### 3. What is the difference between `git pull` and `git fetch`?
**Answer:**

| Feature | `git fetch` | `git pull` |
| :--- | :--- | :--- |
| **Action** | Downloads changes from remote to local origin branches. | Downloads changes AND merges them into current branch. |
| **Working Directory** | Does NOT touch your working files. | Updates your working files. |
| **Safety** | Safe (allows review before merging). | Can lead to unexpected conflicts. |
| **Formula** | `fetch` only. | `git fetch` + `git merge`. |

### 4. What is the staging area (Index)?
**Answer:**
The **Staging Area** (or Index) is an intermediate layer between your working directory and the repository handling the history.
*   It allows you to group related changes before committing.
*   You use `git add` to move files from the working directory to the staging area.
*   It enables partial commits (committing only modified files, not all changes).

### 5. What is the difference between `git merge` and `git rebase`?
**Answer:**

| Feature | `git merge` | `git rebase` |
| :--- | :--- | :--- |
| **History** | Non-destructive. Preserves complete history. Adds a "merge commit". | Destructive. Rewrites history to be linear. |
| **Graph** | Messy (many branches/merge bubbles). | Clean (straight line). |
| **Conflict Handling** | Resolve once per merge. | May need to resolve conflicts for *each* commit being replayed. |
| **Use Case** | Merging feature branches to main/master. | Keeping a feature branch up-to-date with main before merging. |

### 6. What is a "detached HEAD" state?
**Answer:**
A **Detached HEAD** means `HEAD` (the pointer to the current branch/commit) is pointing directly to a specific commit, not a branch.
*   **Cause:** Checking out a specific commit hash or tag (`git checkout <commit_hash>`).
*   **Risk:** New commits made in this state are orphan (belong to no branch) and will be lost if you switch away.
*   **Fix:** Create a branch from this point to save changes: `git switch -c new-branch`.

### 7. What is `.gitignore` and what are common patterns?
**Answer:**
A text file that tells Git which files or folders to ignore (not track).
*   **Purpose:** Prevent sensitive data (API keys), build artifacts (`target/`, `node_modules/`), and editor settings (`.vscode/`) from being committed.
*   **Common Patterns:**
    *   `*.log` (Ignore all .log files)
    *   `node_modules/` (Ignore directory)
    *   `.env` (Ignore config file)

### 8. Explain the difference between `git reset` styles (Soft, Mixed, Hard).
**Answer:**
`git reset` moves the HEAD pointer to a previous commit.

| Mode | HEAD Ref | Staging Index | Working Directory | Use Case |
| :--- | :--- | :--- | :--- | :--- |
| **--soft** | Moves to <commit> | Preserves changes (Staged) | Preserves changes | Undo commit but keep work ready to commit again. |
| **--mixed** (Default) | Moves to <commit> | Resets to match HEAD (Unstaged) | Preserves changes | Undo commit & unstage files. |
| **--hard** | Moves to <commit> | Resets to match HEAD | **DELETES CHANGES** | Discard all local work and reset to a clean state. |

### 9. What is `git cherry-pick`?
**Answer:**
It allows you to apply the changes introduced by some existing commits into your current branch.
*   **Command:** `git cherry-pick <commit_hash>`
*   **Scenario:** You fixed a bug in `release-1.0` branch and want to apply *only* that specific fix to `main` without merging the whole branch.

### 10. How to resolve merge conflicts?
**Answer:**
1.  Git marks the file as "conflicted".
2.   Open the file and look for conflict markers:
     ```text
     <<<<<<< HEAD
     Changes in current branch
     =======
     Changes in incoming branch
     >>>>>>> branch-name
     ```
3.  Edit the file to pick the correct code or combine both.
4.  `git add <file>` to mark it resolved.
5.  `git commit` to finish the merge.

### 11. What is `git stash`?
**Answer:**
Temporarily shelves (saves) changes in your working directory so you can work on something else, and then come back and re-apply them later.
*   **Save:** `git stash`
*   **List:** `git stash list`
*   **Apply:** `git stash pop` (applies and removes from list) or `git stash apply` (applies but keeps in list).
*   **Scenario:** You are working on a feature but need to switch branches to fix a critical bug immediately.

### 12. What is `git reflog`?
**Answer:**
**Reference Logs** record when the tips of branches and other references were updated in the local repository.
*   **Usage:** It is a safety net. It allows you to find "lost" commits (e.g., after a bad `git reset --hard` or deleting a branch).
*   **Command:** `git reflog`
*   **Recover:** Find the HEAD@{n} index and reset to it: `git reset --hard HEAD@{n}`.

### 13. What is `git bisect`?
**Answer:**
A tool to use binary search to find the commit that introduced a bug.
*   **Process:**
    1.  `git bisect start`
    2.  `git bisect bad` (Current broken version)
    3.  `git bisect good <commit_hash>` (Last known working version)
    4.  Git jumps to the middle commit. You test and say `git bisect good` or `git bisect bad`.
    5.  Repeat until the culprit commit is found.

### 14. Explain `git push --force` vs `git push --force-with-lease`.
**Answer:**
*   **`--force`:** Unconditionally overwrites the remote branch with your local history. If someone else pushed code in between, their work is **erased**.
*   **`--force-with-lease`:** Safer. It checks if the remote branch is in the state you expect (i.e., you have the latest changes). If someone updated the remote, the push fails, preventing accidental overwrites.

### 15. What are Git Hooks?
**Answer:**
Scripts that run automatically before or after specific Git events.
*   **Client-side:**
    *   `pre-commit`: Check code style (linting), run tests before commit.
    *   `commit-msg`: Enforce commit message format.
*   **Server-side:**
    *   `pre-receive`: Enforce access controls or reject force pushes.

### 16. How do you handle large binary files in Git?
**Answer:**
Standard Git is not optimized for large binaries (videos, datasets) because every user downloads every version of every file.
*   **Solution:** Use **Git LFS (Large File Storage)**.
*   It stores pointers (text files) in the Git repository, while the actual large files are stored on a separate server.
*   Files are downloaded lazily (only the version you checkout).

### 17. Difference between `git rebase` and `git merge --squash`?
**Answer:**
*   **Rebase:** Replays commits one by one. Keeps history linear but preserves individual commit granularity (unless interactive rebase is used to squash).
*   **Merge Squash:** Takes all commits from a feature branch and squashes them into a **single** new commit on the target branch. The feature branch history is lost/collapsed.

### 18. What is the difference between Fork and Branch?
**Answer:**

| Feature | Branch | Fork |
| :--- | :--- | :--- |
| **Scope** | Within the *same* repository. | A *copy* of the entire repository associated with your account. |
| **Collaboration** | Direct access (if permissions allow). | Bridge using "Pull Requests" across repos. |
| **Usage** | Team members working on features. | Open Source contributions (where you don't have write access to the original repo). |

### 19. How to recover a deleted branch?
**Answer:**
If the branch was deleted but not garbage collected:
1.  Run `git reflog` to find the HEAD pointer of the tip of the deleted branch.
2.  Recreate the branch at that commit: `git checkout -b <branch_name> <commit_hash>`.

### 20. Why is `git pull --rebase` often preferred?
**Answer:**
By default, `git pull` does a merge, which creates an unnecessary "Merge branch 'origin/main' into 'main'" commit if your local history diverged.
*   `git pull --rebase` replays your local unpushed commits *on top* of the incoming remote changes.
*   **Result:** A clean, linear history without "merge bubbles".

### 21. How does Git ensure data integrity?
**Answer:**
Git uses the **SHA-1** hashing algorithm.
*   Every file, directory structure (tree), and commit is identified by a checksum (hash) of its contents.
*   If a single bit changes in a file, the file's hash changes, causing the tree hash to change, and the commit hash to lose validity. This makes it impossible to alter history without detection.

### 22. What is a Bare Repository?
**Answer:**
A repository that acts as a central storage point for sharing.
*   **Contents:** Contains only the `.git` folder contents (history, config).
*   **No Working Tree:** You cannot edit files or run code in a bare repo.
*   **Usage:** Remote servers (GitHub, GitLab, internal servers). Created with `git init --bare`.

### 23. What is the difference between `revert` and `reset`?
**Answer:**

| Command | Action | History Effect | Shared Repo Safe? |
| :--- | :--- | :--- | :--- |
| **Reset** | Moves HEAD back to past. | Removes commits. | **NO**. Breaks history for others. |
| **Revert** | Creates a *new* commit that is the inverse of the target. | Adds a commit. | **YES**. Safe for public branches. |

### 24. How do you squash multiple commits into one?
**Answer:**
Using interactive rebase:
1.  `git rebase -i HEAD~n` (where n is number of commits).
2.  In the editor, change `pick` to `squash` (or `s`) for the commits you want to combine into the previous one.
3.  Save and close. Git will prompt to merge commit messages.

### 25. Explain Gitflow workflow.
**Answer:**
A strict branching model designed for project releases.
*   **master/main:** Production-ready code.
*   **develop:** Integration branch for features.
*   **feature/*:** New features branched off develop.
*   **release/*:** Pre-production polish.
*   **hotfix/*:** Urgent fixes branched off master, merged to master and develop.

### 26. How do you remove a file from Git but keep it local?
**Answer:**
If you accidentally committed a config file but want to keep it on your disk:
```bash
git rm --cached <file>
```
Then add it to `.gitignore` to prevent future tracking.

### 27. What is `HEAD`, `HEAD~`, and `HEAD^`?
**Answer:**
*   **HEAD:** Pointer to the current snapshot/branch.
*   **HEAD~1** (or HEAD^): The parent of the current commit.
*   **HEAD~2:** The grandparent of the current commit.
    (Tilde `~` navigates back usually by generation, Caret `^` is used for selecting specific parents in merge commits).

### 28. How to modify the last commit message?
**Answer:**
If the commit has not been pushed yet:
```bash
git commit --amend -m "New message"
```
*Warning:* This changes the commit hash. Do not do this if already pushed and shared.

### 29. What is a submodule?
**Answer:**
A Git repository embedded inside another Git repository.
*   It points to a specific commit of the external repo.
*   **Usage:** Including shared libraries or dependencies that are maintained separately.
*   **Complexity:** Requires explicit initialization (`git submodule update --init --recursive`) and careful management of pointers.

### 30. How to list files changed in a specific commit?
**Answer:**
```bash
git show --name-only <commit_hash>
# OR
git diff-tree --no-commit-id --name-only -r <commit_hash>
# OR for stats
git show --stat <commit_hash>
```

### 31. What is Trunk-Based Development?
**Answer:**
A strategy where developers work in short-lived branches or directly on the "trunk" (main branch).
*   **Key:** Frequent merges (multiple times a day).
*   **Benefit:** Avoids "merge hell" and encourages continuous integration.
*   **Contrast:** Opposite of long-lived feature branches (Gitflow).

### 32. How to clean untracked files?
**Answer:**
To remove files that are in the directory but not added to git:
*   `git clean -n`: **Dry run** (shows what will be deleted).
*   `git clean -f`: Force delete files.
*   `git clean -fd`: Delete files and directories.
