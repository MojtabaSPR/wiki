# User & Group

## Group Commands

List all groups:

```bash
    ~: cat /etc/group
```

List a group members:

```bash
    ~: getent group g1

    # /etc/group also shows group members
    ~: cat /etc/group
    ~: cat /etc/group | grep group1 
```

Create group:

```bash
~: groupadd groupname
```

Remove group:

```bash
~: groupdel groupname
```

## Users Commands

Print current user name:

```bash
~: users
~: whoami
~: id -un
```

List all users info:

```bash
~: cat /etc/passwd
~: getent passwd
```

List all group membership of a user:

```bash
    ~: groups user1
    ~: id user1
```

List users logged in:

```bash
~: who
~: who -a
~: who -u
```

Modify user:

```bash
# Add a User to a Group
~: usermod -a -G GROUP USER
~: usermod -aG docker mojtaba

# Remove a user from a specific group
~: gpasswd -d username groupname
~: deluser username groupname

# Change User Primary Group
~: usermod -g GROUP USER

# Change user password file comment field
~: usermod -c "GECOS Comment" USER

# Changing a User Home Directory
~: usermod -d DIR USER

# Changing a User Name
~: usermod -l NEW_USER USER

# Locking and Unlocking a User Account
~: usermod -L USER
~: usermod -U USER
```

Create user:

```bash
# When executed without any option, useradd creates a new user account using the default settings specified in the /etc/default/useradd file.
# The command adds an entry to the /etc/passwd, /etc/shadow, /etc/group and /etc/gshadow files.
~: useradd user2
~: passwd user2

# create user with home in /home directory
~: useradd -m user2

# create user with home in custom directory
~: useradd -m -d /opt/user2 user2

# create with specific user id
~: useradd -u 1500 user2

# create user and assign multiple group
~: useradd -g users -G wheel,developers user2
```

Remove user:

```bash
~: deluser user2
~: deluser --remove-home user2
```

## Files related to users and groups

### /etc/passwd

This file contains all the user information on your system:

```bash
~: tail /etc/passwd
scard:x:491:489:Smart Card Reader:/var/run/pcscd:/usr/sbin/nologin
sshd:x:493:491:SSH daemon:/var/lib/sshd:/bin/false
statd:x:488:65534:NFS statd daemon:/var/lib/nfs:/sbin/nologin
tftp:x:496:493:TFTP account:/srv/tftpboot:/bin/false
lightdm:x:10:14:Light Display Manager:/var/lib/lightdm:/bin/false
wwwrun:x:30:8:WWW daemon apache:/var/lib/wwwrun:/bin/false
mojtaba:x:1000:100:mojtaba:/home/mojtaba:/bin/bash
svn:x:485:482:user for Apache Subversion svnserve:/srv/svn:/sbin/nologin
privoxy:x:484:480:Daemon user for privoxy:/var/lib/privoxy:/bin/false
```

### /etc/shadow

This file contains password (hashed passwords) of the users. See how the /etc/passwd is readable for all but /etc/shadow is only readable for root and members of the shadow group:

```bash
~: ls -ltrh /etc/passwd /etc/shadow
-rw-r--r-- 1 root root   1.9K Oct 28 15:47 /etc/passwd
-rw-r----- 1 root shadow  851 Oct 29 19:06 /etc/shadow
```

```bash
~: tail /etc/shadow
scard:!:16369::::::
sshd:!:16369::::::
statd:!:16369::::::
tftp:!:16369::::::
uucp:*:16369::::::
lightdm:*:16369::::::
mojtaba:$6$enk5I3bv$uSQrRpen7m9xDapYLgwgh3P/71OLZUgj31n8AwzgIM2lA5Hc/BmRVAMC0eswdBGkseuXSvmaz0lsYFtduvuqUo:16737:0:99999:7:::
svn:!:16736::::::
privoxy:!:19473::::::
```

| field | meaning |
| ----- | ------- |
| 19473 | When was the last time this password changes (total days since 1970) |
| 0 | User wont be able to change the password 0 days after each change |
| 99999 | After this many days, the user HAVE to change his password |
| 7 | ...and the user will be informed 7 days before the expiration to change his password |

### /etc/group

This file contains the groups and their IDs.

```bash
~: tail /etc/group
avahi:x:486:
kdm:!:485:
mysql:x:484:
winbind:x:483:
at:x:25:
svn:x:482:
vboxusers:x:481:
input:x:1000:mojtaba,joe
privoxy:x:480:
```

Note: See that x there? Theoretically groups can have passwords but it is never used in any distro! The file is /etc/gshadow

### /etc/skel/

skel is derived from the skeleton because it contains basic structure of home directory.

The /etc/skel directory contains files and directories that are automatically copied over to a new user’s when it is created from useradd command.

This will ensure that all the users gets same intial settings and environment.

```bash
~: ls -la /etc/skel/
total 24
drwxr-xr-x.  2 root root   62 Apr 11  2018 .
drwxr-xr-x. 77 root root 2880 Mar 28 03:38 ..
-rw-r--r--.  1 root root   18 May 30 17:07 .bash_logout
-rw-r--r--.  1 root root  193 May 30 17:07 .bash_profile
-rw-r--r--.  1 root root  231 May 30 17:07 .bashrc
```

The location of /etc/skel can be changed by editing the line that begins with SKEL= in the configuration file /etc/default/useradd. By default this line says SKEL=/etc/skel.

```bash
~: cat /etc/default/useradd
# useradd defaults file
GROUP=100
HOME=/home
INACTIVE=-1
EXPIRE=
SHELL=/bin/bash
SKEL=/etc/skel
CREATE_MAIL_SPOOL=yes
```
