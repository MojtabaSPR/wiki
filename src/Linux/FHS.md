# FHS

|   directory    |    usage                      |
|----------------|-------------------------------|
| /              |
| /boot          | Contains static files required to boot the system, such as the Linux kernel. These files are essential for the system to boot properly.
| /home          | Contains users' personal files. In addition, home directories contain personalized configuration files which are hidden files preceded by a dot. Such files include the .bashrc, .bash_logout, and .bash_profile to mention a few.|
| /dev           | The /dev directory contains special files that are representative of devices attached to the system. These include consoles, hard drives, or any other peripheral devices plugged into the system. A good example of a device file is /dev/sda which represents the first SATA hard drive attached to the Linux system. The /dev directory is also a storage location for pseudo devices or virtual devices that do not reference any hardware connected to the system. An example is the /dev/null file which discards any data sent to it.|
| /etc           | The /etc directory contains host-specific system-wide configuration files. It stores configuration files required by all programs as well as startup and shutdown shell scripts.|
| /lib,/lib64    | The /lib directory contains shared library images required in the /bin or /sbin directories. These are essential libraries required by the system to boot and run normally. These two directories are symlink to /usr/lib and /usr/lib64 |
| /media         | The /media directory contains temporary sub-directories on which removable media such as optical drives are automatically mounted. A good example of a sub-directory is /media/cdrom for optical drives.|
| /mnt           | The /mnt directory provides a temporary mount point on which removable media such as CDROMs can be mounted. It is most often used to mount storage devices or partitions manually, and is more of a relic of the past.|
| /opt           | The /opt directory contains add-on applications or software packages that are provided by a third-party vendor and are not installed through your operating system package manager. Each such application has its own subdirectory which contains all the essential files needed for it to run. When you install a software package from a third-party repository, or compile software binaries yourself, the files are stored in the /opt directory.|
| /run           | Most Linux distributions come with the /run filesystem. This is a directory that stores volatile runtime data since the system was started. Data stored in this directory does not persist upon a reboot.|
| /var           | Var stands for variable. As the name suggests the /var directory is a directory that contains files that are constantly changing in size such as log and spool files.|
| /proc          | Also referred to as the proc filesystem, The /proc directory is a virtual or pseudo filesystem that contains special files that provide information about running processes and the kernel’s current state. It is regarded as the information and control center of the Linux kernel. The proc directory is a peculiar directory in that it’s not a real filesystem and it ceases to exist once the system is powered off. It is mounted at the /proc mount point during the booting process.|
| /usr           | This is one of the most critical directories in the Linux system. The /usr directory is a directory that comprises libraries, binaries, and documentation for installed software applications. System files contained in this directory are shareable among other users.|
| /bin           | symlink to /usr/bin |
| /sbin          | symlink to /usr/sbin |
| /usr/bin       | This directory contains essential system binaries (i.e., programs) that are necessary for basic system functionality. These include cat, chown, chmod, ping, cp, mkdir, ls, cat, rm, and mv just to mention a few. |
| /usr/sbin      | This directory contains binary executables and command line tools that are preserved for the root user. These are privileged commands used for system administration tasks. Examples of such commands include fdisk, route, reboot, mkfs, init, and fsck to mention a few. |
| /usr/local     | The directory contains user programs installed from source or outside the distribution's provided software. For example, when you install the Go programming language from source, it goes under the /usr/local/go directory.|
| /usr/src       | This contains the Linux header files, kernel sources, and documentation.|
| /srv           | The term srv stands for service. The /srv directory contains site-specific data for your Linux distribution. It points to the location for data files for a specific service such as www, rsync, FTP and CVS.|
| /sys           | The /sys/ directory utilizes the new sysfs virtual file system specific to the 2.6 kernel. With the increased support for hot plug hardware devices in the 2.6 kernel, the /sys/ directory contains information similarly held in /proc/, but displays a hierarchical view of specific device information in regards to hot plug devices.|
| /run           | Most Linux distributions come with the /run filesystem. This is a directory that stores volatile runtime data since the system was started. Data stored in this directory does not persist upon a reboot.|
| /usr/share     | The /usr/share directory contains architecture-independent shareable text files. The contents of this directory can be shared by all machines, regardless of hardware architecture.|

## Locating files

### Which

Linux which command is used to identify the location of a given executable that is executed when you type the executable name (command) in the terminal prompt. The command searches for the executable specified as an argument in the directories listed in the PATH environment variable.

```bash
~: which ping
/usr/bin/ping

~: which touch
/usr/bin/touch

~: which -a touch
/usr/bin/touch
/bin/touch
```

### Type

The type command is a built-in bash shell command that can provide the type of a specified command.

What does it mean by the “type of a command”? It means that you can get information like whether a Linux command is built-in shell command, where its executable is located and whether it is aliased to some other command.

```bash
~: type echo
echo is a shell builtin

~: type ls
ls is aliased to 'ls  --color'

~: type for
for is a shell keyword

~: type ping
ping is /usr/bin/ping

~: type fdisk
fdisk is /usr/sbin/fdisk

~: type -a pwd
pwd is a shell builtin
pwd is /usr/bin/pwd
pwd is /bin/pwd

~: type -a fdisk
fdisk is /usr/sbin/fdisk
fdisk is /sbin/fdisk

~: type -a for
for is a shell keyword

~: type -a passwd
passwd is /usr/bin/passwd
passwd is /bin/passwd

~: type cut
cut is /usr/bin/cut

~: type -a cut
cut is /usr/bin/cut
cut is /bin/cut

~: type -t cut
file

~: type -t fdisk
file

~: type -t pwd
builtin
```

### Whereis

whereis is a command-line utility that allows you to find the location of the binary, source, and manual page files for a given command.

```bash
~: whereis bash
bash: /usr/bin/bash /usr/share/man/man1/bash.1.gz

~: whereis ping
ping: /usr/bin/ping /usr/share/man/man8/ping.8.gz

~: whereis type
type:

```

### Locate

The locate command is the quickest and simplest way to search for files and directories by their names.

The locate command searches for a given pattern through a database file that is generated by the updatedb command. The found results are displayed on the screen, one per line.

During the installation of the mlocate package, a cron job is created that runs the updatedb command every 24 hours. This ensures the database is regularly updated. For more information about the cron job check the /etc/cron.daily/mlocate file.

```bash
~: apt install mlocate
```
