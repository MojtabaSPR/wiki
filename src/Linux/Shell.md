# Shell

## What is Shell

A Shell provides you with an interface to the Unix system.

Shell is an environment in which we can run our commands, programs, and shell scripts.

### Some Shell

1. Bourne Shell (sh)
2. C Shell (csh)
3. TENEX C Shell (tcsh)
4. KornShell (ksh)
5. Debian Almquist Shell (dash)
6. Bourne Again Shell (bash)
7. Z Shell (zsh)
8. Friendly Interactive Shell (fish)

**Determine which shell we are using:**

```bash
~: echo $SHELL
/bin/bash

~: echo $SHELL
/usr/bin/zsh
```

## Differnet type of shell

### 1. login shell

A login shell is a shell given to a user upon login into their user account.

When you login into the shell, say after a ssh or when you sit behind a terminal and login into a Linux machine or switche to a user with the “su -” command. This is an interactive login session.

#### The following scripts are executed by the Login Shell

1. /etc/profile is run

2. A line in /etc/profile runs whatever is in /etc/profile.d/

3. One of the below items will run
 ~/.bash_profile
 ~/.bash_login
 ~/.profile

4. ~/.bashrc
5. ~/.bashrc invokes /etc/bashrc
Commonly you will add whatever personal config you want into the ~/.bashrc

### 2. non-login shell (sub shell)

When you open a new terminal (say from a GUI env). This is also interactive but not a login shell.

When start a shell within another shell it is a non-login shell.

#### The following scripts are executed by the non Login Shell

1. /etc/bash.bashrc (or /etc/bashrc in some systems)
2. ~/.bashrc

### 3. non-login shell non-interactive

When execute a program or command or scripts they open a new shell that is a non-login non-interactive shell. It means we can not interact with that shell. It just opens to do the command

### Checking whether a shell is a login or non-login shell

```bash
~: echo $0
```

Login shell output will be -bash or -su.

Non logins shell output will be bash or su.

```bash
# ssh to a server
~: echo $0
-bash

# if open a terminal from GUI
~: echo $0
bash

# ssh to a server then open a new shell in current shell
~: echo $0
-bash
~: bash
~: echo $0
bash
~: exit     # exit from child shell
~: echo $0
-bash
```

## Variable

### Show variables

```bash
# show all variables
~: env
~: export

# show specific variable
~: echo $VAR_NAME
~: echo $HOME
/home/mojtaba
~: echo $SHELL
/bin/bash
```

### Define new variable

#### 1. shell variable

#### 2. environment variable

The key difference between Linux shell variables and Linux environment variables is: shell variables are not shared with a shell’s child processes, environment variables are shared with a shell's child processes.

```bash
# define new shell variable
~: VAR_NAME=VALUE
~: Name=mojtaba
~: echo $Name
mojtaba

# define new environment variable
~: export Name=mojtaba
~: echo $Name
mojtaba
```

#### 3. Permanent environment variable

The first two variables(shell and environment) only last for the duration of your shell session. If you log out or reboot, you would need to recreate them if you want to use them.

We can define variable in these three files to make them permanent:

1- ~/.profile: Use .profile to Make Environment Variables Permanent for Login Shells

```bash
~: vim ~/.profile
export Name=Mojtaba
```

2- ~/.bashrc: Use .bashrc to Make Environment Variables Permanent for Non-login Interactive Shells

```bash
~: vim ~/.bashrc
export Name=Mojtaba
```

3- /etc/environment: Use /etc/environment to Make Environment Variables Permanent System-wide

```bash
~: vim /etc/environment
Name=Mojtaba
```

## Source

The source command reads and executes commands from the file specified as its argument in the current shell environment. It is useful to load functions, variables, and configuration files into shell scripts.

source is a shell built-in in Bash and other popular shells used in Linux and UNIX operating systems. Its behavior may be slightly different from shell to shell.

source and . (a period) are the same command.

If the FILENAME is not a full path to a file, the command will search for the file in the directories specified in the $PATH environmental variable . If the file is not found in the $PATH, the command will look for the file in the current directory.

If any ARGUMENTS are given, they will become positional parameters to the FILENAME.

If the FILENAME exists, the source command exit code is 0, otherwise, if the file is not found it will return 1.

```bash
~: source .venv/bin/activate
```
