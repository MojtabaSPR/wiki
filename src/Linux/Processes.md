# Managing processes

## foreground and background jobs

Typically when you run a command in the terminal, you have to wait until the command finishes before you can enter another one. This is called running the command in the foreground or foreground process. When a process runs in the foreground, it occupies your shell, and you can interact with it using the input devices.

A background process is a process/command that is started from a terminal and runs in the background, without interaction from the user.

### Run a Linux Command in the Background

```bash
~: command &
~: cp -r tmp/ tmp2 &
[1] 15524
```

You can have multiple processes running in the background at the same time.

#### To see background jobs

```bash
~: cp -r tmp/ tmp2 &
[1] 15524

~: jobs
[1]+  Running   cp -r tmp/ tmp2 &

```

#### To bring a background process to the foreground

```bash
~: fg

# if we have multiple jobs in background we can bring specific one by its number
~: jobs
[1]-  Running                 xeyes &
[2]+  Running                 sleep 1000 &

~: fg %2
```

#### To move a running foreground process to background

```bash
~: xeyes
# press ctrl-z to pause the proccess
^Z
[1]+    Stopped   xeyes

~: jobs
[1]+    Stopped   xeyes

~: bg  # or bg %1
[1]+  xeyes &

~: jobs 
[1]+    Running   xeyes &
```

#### Keep Background Processes Running After a Shell Exits

If your connection drops or you log out of the shell session, the background processes are terminated.

One way is to remove the job from the shell’s job control using the **disown** shell builtin:

```bash
# Send a process to background
~: xeyes &

~: disown

# If you have more than one background jobs, include % and the job ID after the command:
~: disown %1

# Confirm that the job is removed from the table of active jobs:
~: jobs -l

#: To list all running processes, including the disowned use the ps aux command.
~: ps -aux | grep xeyes
```

The **nohup** command executes another program specified as its argument and ignores all SIGHUP (hangup) signals. SIGHUP is a signal that is sent to a process when its controlling terminal is closed.

By default, nohup redirects the command output to the nohup.out file. This file is created in the current working directory . If the user running the command doesn’t have write permissions to the working directory, the file is created in the user’s home directory.

```bash
~: nohup command &

~: nohup xeyes &

~: nohup ping 4.2.2.4 &

# nohup and redirect stdout to a file
~: nohup ls  tmp*  > output.txt  &  

# nohup and redirect both stdout and stderr to a file
~: nohup ping 4.2.2.4  &> output.txt  &  
```

## Find the path of a process

```bash
# first find the pid of process
~: ps -aux | grep -i php
root     16579  0.6  0.0 221880  7448 ?        S    14:06   0:11 php-fpm: pool cwpsrv

# then read the info of pid
~: readlink -f /proc/16579/exe
/usr/local/cwp/php71/sbin/php-fpm
```
