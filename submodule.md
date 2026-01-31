# Git Submodules

## Adding a Submodule

```bash
git submodule add <repository-url> <path>
```

Or remove the `<path>` to use the repository name as path

```bash
git submodule add <repository-url>
```

**Note:** Creates `.gitmodules` file

Commit

```bash
git add .
git commit -m "Add my submodule"
```

## Cloning a Repository with Submodules

When cloning a repository that contains submodules, you have two options.

Clone and initialize submodules in one command:

```bash
git clone --recurse-submodules <repository-url>
```

Or if you've already cloned the repository

```bash
git submodule init
git submodule update
```

## Updating a Submodule

To update a specific submodule to the latest commit on its tracked branch:

```bash
cd <submodule-path>
git fetch
git merge origin/<branch-name>
cd ..
git add <submodule-path>
git commit -m "Update submodule"
```

Or from the parent repository

```bash
git submodule update --remote <submodule-path>
```

## Updating All Submodules

To update all submodules to their latest commits on their tracked branches:

```bash
git submodule update --remote --merge
```

Or to rebase intead of merge:

```bash
git submodule update --remote --rebase
```
