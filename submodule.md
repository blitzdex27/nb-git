# Git Submodule Guide

A comprehensive reference for managing Git submodules — adding, updating, removing, and troubleshooting.

---

## Table of Contents

1. [Adding a Submodule](#1-adding-a-submodule)
2. [Cloning a Repository with Submodules](#2-cloning-a-repository-with-submodules)
3. [Updating Submodules](#3-updating-submodules)
4. [Tracking a Specific Branch](#4-tracking-a-specific-branch)
5. [Locking a Submodule to a Specific Commit](#5-locking-a-submodule-to-a-specific-commit)
6. [Changing Submodule URLs (HTTPS → SSH)](#6-changing-submodule-urls-https--ssh)
7. [Renaming Branch from master to main](#7-renaming-branch-from-master-to-main)
8. [Removing Submodules](#8-removing-submodules)
9. [Useful Commands Reference](#9-useful-commands-reference)
10. [Troubleshooting](#10-troubleshooting)
11. [Tips and Best Practices](#11-tips-and-best-practices)

---

## 1. Adding a Submodule

```bash
git submodule add <repository-url> <path>
```

**Example:**
```bash
git submodule add git@github.com:user/library.git libs/library
```

This will:
- Clone the submodule repository into `libs/library`
- Create or update the `.gitmodules` file
- Stage both changes automatically

**Commit after adding:**
```bash
git add .gitmodules libs/library
git commit -m "Add library submodule"
git push
```

---

## 2. Cloning a Repository with Submodules

**Option A: Clone with submodules in one command (recommended)**
```bash
git clone --recurse-submodules <repository-url>
```

**Option B: If you already cloned without submodules**
```bash
git submodule init
git submodule update
```

Or combine both in one command:
```bash
git submodule update --init --recursive
```

> **Tip:** Always use `--recurse-submodules` when cloning to avoid forgetting the init/update step.

---

## 3. Updating Submodules

### Update a Single Submodule

```bash
git submodule update --remote <submodule-path>
```

**Example:**
```bash
git submodule update --remote libs/library
```

### Update All Submodules

```bash
git submodule update --remote --merge
```

Or use rebase for a cleaner history:
```bash
git submodule update --remote --rebase
```

### Commit After Updating

After updating, you may see:
```
modified:   nb-git (new commits)
modified:   nb-objc (new commits)
```

This means the submodules have moved to newer commits. Stage and commit them:
```bash
git add nb-git nb-objc
git commit -m "Update submodules to latest commits"
git push
```

> **Tip:** You can also use `git add .` to stage all submodule changes at once if you have multiple submodules.

---

## 4. Tracking a Specific Branch

By default, submodules are in a "detached HEAD" state. To configure a submodule to track a specific branch:

```bash
git config -f .gitmodules submodule.<submodule-path>.branch <branch-name>
```

**Example:**
```bash
git config -f .gitmodules submodule.libs/library.branch develop
```

**Commit the change:**
```bash
git add .gitmodules
git commit -m "Configure library submodule to track develop branch"
git push
```

After this, `git submodule update --remote` will pull the latest from the configured branch.

---

## 5. Locking a Submodule to a Specific Commit

Submodules are naturally "locked" to the commit that the parent repository records. To lock a submodule to a specific commit or tag:

```bash
cd <submodule-path>
git checkout <commit-sha-or-tag>
cd ..
git add <submodule-path>
git commit -m "Lock library submodule to v1.2.0"
git push
```

**Example:**
```bash
cd libs/library
git checkout v1.2.0
cd ..
git add libs/library
git commit -m "Lock library submodule to v1.2.0"
git push
```

> **Tip:** Locking to tags is preferred over commit SHAs because tags are human-readable and easier to maintain.

---

## 6. Changing Submodule URLs (HTTPS → SSH)

### Option A: Edit `.gitmodules` Manually

Open `.gitmodules` and change the URL:

```ini
# Before
[submodule "libs/library"]
    path = libs/library
    url = https://github.com/user/library.git

# After
[submodule "libs/library"]
    path = libs/library
    url = git@github.com:user/library.git
```

### Option B: Use `sed` (Recommended for Multiple Submodules)

```bash
# Linux
sed -i 's|https://github.com/|git@github.com:|g' .gitmodules

# macOS
sed -i '' 's|https://github.com/|git@github.com:|g' .gitmodules
```

### Option C: Use `git config`

```bash
git config -f .gitmodules submodule.libs/library.url git@github.com:user/library.git
```

### After Changing URLs

```bash
# Sync submodules with new URLs
git submodule sync --recursive

# Update submodules
git submodule update --init --recursive

# Verify the change
git submodule foreach 'git remote -v'

# Commit
git add .gitmodules
git commit -m "Change submodule URLs from HTTPS to SSH"
git push
```

### URL Format Reference

| Type  | Format                                      |
|-------|---------------------------------------------|
| HTTPS | `https://github.com/username/repository.git`|
| SSH   | `git@github.com:username/repository.git`    |

> **Note:** SSH uses a colon (`:`) after the host, not a slash (`/`).

---

## 7. Renaming Branch from `master` to `main`

### For the Main Repository

```bash
# Rename local branch
git branch -m master main

# Push new branch to remote
git push -u origin main

# (Go to GitHub/GitLab → Settings → Branches → Change default branch to main)

# Delete old master branch from remote
git push origin --delete master

# Delete old master branch locally (if it still exists)
git branch -D master
```

### For Submodules

```bash
# Rename master to main in all submodules
git submodule foreach 'git branch -m master main && git push -u origin main'

# (Go to GitHub/GitLab and change default branch to main for each submodule)

# Delete old master branch in all submodules
git submodule foreach 'git push origin --delete master'
git submodule foreach 'git branch -D master'
```

### Update `.gitmodules` Branch Tracking

```bash
# Replace all master references with main
sed -i 's/branch = master/branch = main/g' .gitmodules

# Commit
git add .gitmodules
git commit -m "Update submodule branch tracking from master to main"
git push
```

### For Team Members

Other team members who cloned the repository need to run:
```bash
git fetch origin
git checkout main
git branch -u origin/main
git branch -d master
git submodule update --remote --merge
```

---

## 8. Removing Submodules

### Remove a Single Submodule

```bash
# Step 1: Deinitialize
git submodule deinit -f <submodule-path>

# Step 2: Remove from working tree
git rm -f <submodule-path>

# Step 3: Remove cached Git data
rm -rf .git/modules/<submodule-path>

# Step 4: Commit
git commit -m "Remove <submodule-name> submodule"
git push
```

**Example:**
```bash
git submodule deinit -f libs/library
git rm -f libs/library
rm -rf .git/modules/libs/library
git commit -m "Remove library submodule"
git push
```

### Remove All Submodules

```bash
# Deinitialize all
git submodule foreach --recursive 'git submodule deinit -f .'

# Remove all submodule directories
git rm -rf .

# Remove cached Git data
rm -rf .git/modules/*

# Commit
git commit -m "Remove all submodules"
git push
```

### Remove Submodule but Keep the Files

If you want to convert a submodule into a regular directory:

```bash
git submodule deinit -f <submodule-path>
git rm --cached <submodule-path>
rm -rf .git/modules/<submodule-path>
git add <submodule-path>
git commit -m "Convert submodule to regular directory"
git push
```

### Clean Up `.gitmodules`

After removing all submodules, if `.gitmodules` is empty:
```bash
git rm .gitmodules
git commit -m "Remove empty .gitmodules file"
git push
```

---

## 9. Useful Commands Reference

| Command | Description |
|---------|-------------|
| `git submodule status` | Show all submodules and their current commit |
| `git submodule foreach '<cmd>'` | Run a command inside each submodule |
| `git submodule sync --recursive` | Sync submodule URLs from `.gitmodules` |
| `git submodule update --init --recursive` | Initialize and update all submodules |
| `git submodule update --remote --merge` | Update all submodules to latest on tracked branch |
| `git submodule update --remote --rebase` | Same as above but uses rebase |
| `cat .gitmodules` | View submodule configuration |
| `cat .git/config` | View local submodule config |

### Common One-Liners

**Pull latest changes and update submodules:**
```bash
git pull && git submodule update --init --recursive
```

**Checkout a specific branch in all submodules:**
```bash
git submodule foreach 'git checkout main'
```

**Check status of all submodules:**
```bash
git submodule foreach 'echo "=== $sm_path ===" && git status'
```

**Push all submodules:**
```bash
git submodule foreach 'git push'
```

---

## 10. Troubleshooting

### Push Rejected: "fetch first"

```
! [rejected]        main -> main (fetch first)
error: failed to push some refs
```

The remote has commits you don't have locally.

**Solution A: Pull and merge**
```bash
git pull origin main --allow-unrelated-histories
git push -u origin main
```

**Solution B: Force push (only if remote changes are not needed)**
```bash
# Safer option — fails if someone else pushed in the meantime
git push -u origin main --force-with-lease

# Nuclear option — use only when you're absolutely sure
git push -u origin main --force
```

### Submodule Shows "modified (new commits)" After URL Change

```
modified:   nb-git (new commits)
modified:   nb-objc (new commits)
```

This is expected after syncing URLs. Just stage and commit:
```bash
git add nb-git nb-objc
git commit -m "Update submodules after URL change"
git push
```

### "No submodule mapping found"

The submodule may be partially removed. Clean it up:
```bash
git rm -f <submodule-path>
rm -rf .git/modules/<submodule-path>
```

### Submodule Directory Still Exists After Removal

Manually delete it:
```bash
rm -rf <submodule-path>
git status
```

### Permission Denied When Deleting Remote Branch

The branch might be protected. Go to GitHub/GitLab:
1. **Settings → Branches**
2. Remove branch protection for the branch
3. Try deleting again:
   ```bash
   git push origin --delete master
   ```

### SSH Key Not Set Up

Before using SSH URLs, make sure your SSH key is configured:
```bash
# Check for existing keys
ls -la ~/.ssh

# Generate a new key (if needed)
ssh-keygen -t ed25519 -C "your_email@example.com"

# Start SSH agent and add key
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519

# Test connection
ssh -T git@github.com
```

Then add the public key (`~/.ssh/id_ed25519.pub`) to your GitHub/GitLab account.

---

## 11. Tips and Best Practices

1. **Always use SSH over HTTPS** — SSH keys are more secure and avoid repeated password prompts.

2. **Use `--recurse-submodules` when cloning** — This prevents the common mistake of forgetting to initialize submodules.

3. **Lock submodules to tags, not SHAs** — Tags like `v1.2.0` are readable and meaningful. Raw commit SHAs are not.

4. **Always configure branch tracking** — Use `git config -f .gitmodules submodule.<path>.branch <branch>` so that `--remote` updates pull from the correct branch.

5. **Commit submodule updates explicitly** — After running `git submodule update --remote`, always stage and commit the changes in the parent repo. Otherwise, the parent still points to the old commit.

6. **Use `--force-with-lease` instead of `--force`** — `--force-with-lease` is a safer alternative that won't overwrite someone else's push.

7. **Run `git submodule sync --recursive` after URL changes** — This ensures the local `.git/config` matches the updated `.gitmodules`.

8. **Use `git submodule foreach` for bulk operations** — Instead of manually entering each submodule directory, run commands across all submodules at once.

9. **Change the default branch on GitHub/GitLab before deleting `master`** — If you delete `master` before changing the default branch, the repository may have issues.

10. **Back up `.gitmodules` before bulk changes** — Before running `sed` or other bulk edits, copy the file first:
    ```bash
    cp .gitmodules .gitmodules.backup
    ```