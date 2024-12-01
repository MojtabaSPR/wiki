# Git History

## Git log command

### Show git history

```bash
~/dev/git (master ✗) git log

~/dev/git (master ✗) git log --oneline
d473240 (HEAD -> master) update gitignore
94a4bd2 init

# show git history with file changes stats
~/dev/git (master ✗) git log --oneline --stat

# show git log with file content changes
~/dev/git (master ✗) git log --oneline --patch

# show git log of a specific file with content changes
# it shows all commits that main.go has been modified in them
~/dev/git (master ✗) git log --oneline --patch main.go

# show git log of a specific commit id with files content changes
~/dev/git (master ✗) git log --oneline --patch 2dfe5b3
```

### Filter out result

```bash

~/dev/git (master ✗) git log --oneline --author="Mojtaba Safaei"

~/dev/git (master ✗) git log --before="2020-08-17"
~/dev/git (master ✗) git log --after="one week ago"

~/dev/git (master ✗) git log --grep="GUI" # Commits with "GUI" in their message
~/dev/git (master ✗) git log --oneline --grep="Fix" # Commits with "Fix" in their message

~/dev/git (master ✗) git log -S"GUI" # Commits with "GUI" in their patches

~/dev/git (master ✗) git log hash1..hash2 # Range of commits

~/dev/git (master ✗) git log file.txt # Commits that touched file.txt


```

### Git show

```bash
# show the content of a file in specific commit id
~/dev/git (master ✗) git show d473240:.gitignore

# show the content of file in master
~/dev/git (master ✗) git show d473240:.gitignore

# show list of files changed in specific commit
~/dev/git (master ✗) git show 2dfe5b3 --name-only

# show list of files changed in specific commit with type of change (A = Added, M = Modified, D = Deleted)
~/dev/git (master ✗) git show 94a4bd2 --name-status
```
