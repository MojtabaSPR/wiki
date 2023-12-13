# Symbolic links

A symbolic link, also known as a symlink or soft link, is a special type of file that points to another file or directory.

## Links Types

There are two types of links in Linux/UNIX systems:

### Hard links

You can think a hard link as an additional name for an existing file. Hard links are associating two or more file names with the same inode.

You can create one or more hard links for a single file. Hard links cannot be created for directories and files on a different filesystem or partition.

### Soft links

A soft link is something like a shortcut in Windows. It is an indirect pointer to a file or directory. Unlike a hard link, a symbolic link can point to a file or a directory on a different filesystem or partition.

```bash
~: ln [OPTIONS] FILE LINK
```

```bash
# Create hard link To a File 
~: ln -s my_file.txt my_link.txt

# Create Symlink To a File
~: ln -s my_file.txt my_link.txt

# Create Symlink To a directory
~: ln -s /mnt/my_drive/movies ~/my_movies

# Overwrite symlink
~: ln -sf my_file2.txt my_link.txt

# Remove symlink
~: unlink symlink_to_remove
~: rm symlink_to_remove
```

```bash
~: ln script.sh hlink_to_script.sh
~: ln -s script.sh slink_to_script.sh
~: ls -l
total 8
-rw-rw-r-- 2 mojtaba mojtaba 40 Dec 10 19:02 hlink_to_script.sh
-rw-rw-r-- 2 mojtaba mojtaba 40 Dec 10 19:02 script.sh
lrwxrwxrwx 1 mojtaba mojtaba  9 Dec 10 19:01 slink_to_script.sh -> script.sh
```

- Notes:

1. In hard link if we remove original file the link still works because the data on disk wont delete
2. In symlink if we remove original file the link will be broken
