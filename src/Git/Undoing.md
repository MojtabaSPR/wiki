# Undoing Changes

## Different type of undoings

- Unstaged changes (changes made to the working tree since the last commit)

- Staged changes (changes made to the working tree and added to the staging area / index)

- Commits

The "restore" command helps to unstage or even discard uncommitted local changes.

### Undo Unstaged Changes

You made changes to a file, saved the file and decide that you don't want to keep these changes.

- if there is a staged version of file, this will restore the file to the last staged version
- if there is NOT a staged version of file, this will restore the file to the last commited version

```bash
~/dev/git (master ✗) git restore .gitignore
```

### Undo Staged Changes

Removes the file from the Staging Area, but leaves its actual modifications untouched.

```bash
# with this command the file will only be removed from the Staging Area
~/dev/git (master ✗) git restore --staged .gitignore
```

### Restores a specific revision of the file

```bash
# restore file to a specific commit id
~/dev/git (master ✗) git restore --source 7173808e index.html

# restore file to the 2 commits before master (master~2)
~/dev/git (master ✗) git restore --source master~2 index.html

```
