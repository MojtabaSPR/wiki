# Permissions

## Get permissions

```bash
~: ls -l
-rwxr-xr-x 1 mojtaba mojtaba       237 May 11  2023 Dockerfile
drwxr-xr-x 2 mojtaba mojtaba      4096 Jul 19 12:13 build
drwxr-xr-x 3 mojtaba mojtaba      4096 May 11  2023 test.py
```

The below table shows some more information regarding the first part of the ls -l command:

| Position | Meaning |
| --- | --- |
| 1 | What this entry is. Dash (-) is for ordinary files, 'l' is for links & 'd' is for directory
| 2,3,4 | read, write and execute access for the owner
| 5,6,7 | read, write and execute access for the group members
| 8,9,10 | read, write and execute access for other users
| 11 | Indicated if any other access methods (such as SELinux) are applies to this file - not part of the LPIC 101 exam

## Change permissions

```bash
~: chmod permissions filename
```

There are 2 ways to use the command:

1. Absolute mode
2. Symbolic mode

### *Absolute(Numeric) Mode in Linux:*

| Number | Permission Type | Symbol |
| --- | --- | --- |
| 0 | No Permission | - |
| 1 | Execute | -x |
| 2 | Write | -w- |
| 3 | Execute + Write | -wx |
| 4 | Read | r-- |
| 5 | Read + Execute | r-x |
| 6 | Read +Write | rw- |
| 7 | Read + Write +Execute | rwx

```bash
~: chmod 755  file1.txt
~: chmod 640  file2.txt
~: chmod -R 644 directory1
```

### *Symbolic Mode in Linux:*

In the Absolute mode, you change permissions for all 3 owners. In the symbolic mode, you can modify permissions of a specific owner. It makes use of mathematical symbols to modify the Unix file permissions.

| Operator | Description |
| --- | --- |
| + | Adds a permission to a file or directory
| – | Removes the permission
| = | Sets the permission and overrides the permissions set earlier

The various owners are represented as:

| User | Denotations |
| --- | --- |
| u | user/owner
| g | group
| o | other
| a | all

```bash
# Append execute permission to all (user,group,other)
~: chmod +x myfile

# Append execute,read to user and other
~: chmod uo+xr myfile

# Append read,write,execute to user ang group
~: $ chmod ug+rwx example.txt

# Remove execute permission from other
~: chmod o-x 1.txt

# Different permisssions can be seperated by comma
# This will append
~: chmod u+rwx,g+r,o-x 1.txt
# This will change
~: chmod u=rw,g=r,o   1.txt
```

## Change owner

```bash
# Change user owner of file
~: chown mojtaba  1.txt

# Change both user and group owner of file
~: chown mojtaba:mojtaba 1.txt

# In case you want to change group-owner only, use the command
~: chgrp groupname 1.txt
```

## Special file permissions

### SUID

SUID is the special permission for the user access level and always executes as the user who owns the file, no matter who is passing the command.

A file with SUID always executes as the user who owns the file, regardless of the user passing the command. If the file owner doesn't have execute permissions, then use an uppercase S here

Now, to see this in a practical light, let's look at the /usr/bin/passwd command. This command, by default, has the SUID permission set:

```bash
~: ls -l /usr/bin/passwd 
-rwsr-xr-x 1 root root 33544 Dec 13  2019 /usr/bin/passwd
```

Note the **s** where **x** would usually indicate execute permissions for the user.
The **s** that indicates SUID is in position 4.

### SGID

- If set on a file, it allows the file to be executed as the group that owns the file (similar to SUID)
- If set on a directory, any files created in the directory will have their group ownership set to that of the directory owner

```bash
~: ls -l
total 0
drwxrws--- 2 mojtaba mojtaba  69 Apr  7 11:31 my_articles
```

This permission set is noted by a lowercase **s** where the **x** would normally indicate execute privileges for the group.

It is also especially useful for directories that are often used in collaborative efforts between members of a group.
By setting the SGID permission on a shared directory, all files and directories created within it will automatically have the same group ownership.
So any member of the group can access any new file. This applies to the execution of files, as well. SGID is very powerful when utilized properly.

As noted previously for SUID, if the owning group does not have execute permissions, then an uppercase S is used.

The **s** that indicates SUID is in position 4.
The **s** that indicates SGID is in position 7.

### Sticky bit

This permission does not affect individual files. However, at the directory level, it restricts file deletion. Only the owner (and root) of a file can remove the file within that directory.

```bash
~: ls -l /tmp
drwxrwxrwt 15 root root 4096 Sep 22 15:28 /tmp/
```

The permission set is noted by the lowercase **t**, where the x would normally indicate the execute privilege.

## Setting special permissions

To set special permissions on a file or directory, you can utilize either of the two methods outlined for standard permissions:

Symbolic or numerical.

- numerical

```bash
~: chmod X### file or directory
```

Where **X** is the special permissions digit.like regular permission but by adding a digit in front of permissions

| Number | Permission |
| --- | --- |
| 0 | Remove special permissions
| 1 | Sticky
| 2 | SGID
| 4 | SUID
| 6 | SGID,SUID
| 5 | Sticky,SUID
| 3 | Sticky,SGID
| 7 | Sticky,SUID,SGID

```bash
# Give special sticky permission AND 644 of regular permission
~: chmod 1644 1.txt

# Give special SGID permission AND 744 of regular permission
~: chmod 2744 tmp/

# Give special SGID+SUID permission AND 740 of regular permission
~: chmod 6740 1.sh

# Remove all special permisiions AND 644 of regular permission
~: chmod 0644 1.txt
~: chmod 644 1.txt
```

## umask

Another tricky question: what is the permissions of a newly created file? What happens if you touch a non-existing file? What will be its permissions? This is set by the umask. This command tells the system what permissions (in addition to execute) should not be given to new files. In other words, imagine that all new files will have 666 octal permission (write + read for user, group & others), so a umask of 0002 (yes! 4 digits) will remove the 2 (write) for others (666-0002=664):

```bash
~: umask
0002
```

If we need to change umask, it can be done with the same command:

```bash
~: umask
0002
~: touch newfile
~: ls -ltrh newfile
-rw-rw-r-- 1 mojtaba mojtaba 0 Feb  8 21:38 newfile
~: mkdir newdir
~: ls -ltrhd newdir
drwxrwxr-x 2 mojtaba mojtaba 4.0K Feb  8 21:38 newdir
~: umask u=rw,g=,o=
~: touch newerfile
~: ls -l newerfile
-rw------- 1 mojtaba mojtaba 0 Feb  8 21:41 newerfile
~: umask
0177
```

Note how I used u=rw,g=,o= to tell umask or chmod what I exactly wanted from it.
