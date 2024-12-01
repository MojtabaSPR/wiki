# Git Snapshot

## Initialize new repository

### Add file to staging area

```bash
# add one or more files
~/dev/git (master ✗) git add main.go cmd.go

# add all modified files
~/dev/git (master ✗) git add -A

```

### List staged files

```bash
~/dev/git (master ✗) git ls-files
cmd.go
helper.go
main.go
```

### Commiting changes

```bash
~/dev/git (master ✗) git commit -m "some commit messages"
```

### Rename or move file in git

#### Traditional way

```shell
# first rename or move the file
~/dev/git (master ✗) mv helper.go helper.go.old

# then add two files (new one and old one. git sees them as two seperate files)
~/dev/git (master ✗) git add helper.go helper.go.old

```

#### Git way

```bash
# using git to rename or move a file and stage it at once
# no need to seperately rename annd then add two files (old and new one)
~/dev/git (master ✗) git mv cmd.go cmd.go.old

```

### Ignoring files

#### create ignore file

```bash
~/dev/git (master ✗) touch .gitignore
```

#### add new files

```bash
# add log directory to gitignore
~/dev/git (master ✗) echo logs/ > .gitignore
```

#### ignoring a file or dir that has been staged

To ignore a file that is in staging area first we should remove it from index or staging area
then add it to .gitignore

```bash
~/dev/git (master ✗) git ls-files
.gitignore
build/app.bin
cmd.go.old
helper.go.old
main.go

# --cached remove file or dir only from staging area
# -r is for recursive
~/dev/git (master ✗) git rm --cached -r build/
rm 'build/app.bin'


~/dev/git (master ✗) git ls-files
.gitignore
cmd.go.old
helper.go.old
main.go

~/dev/git (master ✗) echo build/ >> .gitignore

```

### Git Diff

git diff is a good command to view changes if files, it's good to check changes of files before staging or commiting

```bash
# to show changes of files that are in staging area
~/dev/git (master ✗) git diff --staged

# to show changes of files that are in not in staging area
~/dev/git (master ✗) git diff

```
