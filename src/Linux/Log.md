# Log

## rsyslog

The older utility to manage logs was called syslog, then we had syslog-ng (new-generation) and later rsyslog which is what most distros were using till a couple of years ago.

rsyslog is still present and is part of the GNU/Linux universe. It can receive logs from different programs (even over network) and save them in different places (files or files in network or even run some actions on them).

The rsyslog service keeps various log files in the /var/log directory. You can open these files using native commands such as tail, head, more, less, cat, and so forth, depending on what you are looking for.

```bash
~: ls -lsh /var/log
total 4.7M
 36K -rw-r--r--  1 root              root             34K 2023-12-11 10:18:03 alternatives.log
4.0K drwxr-xr-x  2 root              root            4.0K 2023-12-14 10:20:07 apt
 72K -rw-r-----  1 syslog            adm              66K 2023-12-15 12:17:01 auth.log
   0 -rw-------  1 root              root               0 2023-12-13 00:00:01 boot.log
108K -rw-r--r--  1 root              root            106K 2023-08-08 02:23:13 bootstrap.log
   0 -rw-rw----  1 root              utmp               0 2023-08-08 02:22:50 btmp
4.0K drwxr-xr-x  2 root              root            4.0K 2023-12-15 12:11:46 cups
4.0K drwxr-xr-x  2 root              root            4.0K 2023-08-02 19:23:09 dist-upgrade
 48K -rw-r-----  1 root              adm              47K 2023-12-12 22:20:30 dmesg
 48K -rw-r-----  1 root              adm              47K 2023-12-09 10:45:59 dmesg.0
1.2M -rw-r--r--  1 root              root            1.2M 2023-12-14 10:20:20 dpkg.log
 12K -rw-r--r--  1 root              root             32K 2023-12-10 10:54:39 faillog
 12K -rw-r--r--  1 root              root             11K 2023-12-04 01:25:48 fontconfig.log
4.0K drwx--x--x  2 root              gdm             4.0K 2023-12-04 01:33:54 gdm3
4.0K -rw-r--r--  1 root              root            1.3K 2023-12-12 22:20:27 gpu-manager.log
4.0K drwxr-xr-x  3 root              root            4.0K 2023-08-08 02:24:18 hp
4.0K drwxrwxr-x  2 root              root            4.0K 2023-12-04 01:32:04 installer
4.0K drwxr-sr-x+ 3 root              systemd-journal 4.0K 2023-12-04 01:33:25 journal
 76K -rw-r-----  1 syslog            adm              73K 2023-12-14 18:02:52 kern.log
 16K -rw-rw-r--  1 root              utmp            286K 2023-12-15 12:11:54 lastlog
4.0K drwx------  2 root              root            4.0K 2023-08-08 02:22:58 private
1.2M -rw-r-----  1 syslog            adm             1.2M 2023-12-15 12:17:01 syslog
 80K -rw-r--r--  1 root              root             75K 2023-12-15 12:12:17 ubuntu-advantage.log
4.0K drwxr-x---  2 root              adm             4.0K 2023-12-04 11:01:57 unattended-upgrades
 24K -rw-rw-r--  1 root              utmp             20K 2023-12-15 12:11:54 wtmp
```

### rsyslog service

```bash
~: systemctl status rsyslog.service
● rsyslog.service - System Logging Service
     Loaded: loaded (/lib/systemd/system/rsyslog.service; enabled; vendor preset: enabled)
     Active: active (running) since Tue 2023-12-12 22:20:26 +0330; 2 days ago
TriggeredBy: ● syslog.socket
       Docs: man:rsyslogd(8)
             man:rsyslog.conf(5)
             https://www.rsyslog.com/doc/
   Main PID: 584 (rsyslogd)
      Tasks: 4 (limit: 4599)
     Memory: 3.6M
        CPU: 1.181s
     CGroup: /system.slice/rsyslog.service
             └─584 /usr/sbin/rsyslogd -n -iNONE
```

### rsyslog configuration

The actual rsyslog configuration is managed via a configuration file in the /etc/rsyslog.conf and it also reads anything in /etc/rssylog.d/ directory.

#### Facility

The facility represents the process that created the Syslog event. For example, in the event created by the kernel, by the mail system, by security/authorization processes, etc.

| Facility Number | Keyword | Facility Description |
| --- | --- | --- |
| 0 | kern | kernel messages|
| 1 | user | user-level messages|
| 2 | mail | mail system|
| 3 | daemon| system daemons|
| 4 | auth |security/authorization messages|
| 5 | syslog |messages generated internally by syslogd|

#### Severity(Priority)

| Code Severity | Keyword |
| --- | --- |
| 0 | Emergency emerg (panic) |
| 1 | Alert |
| 2 | Critical |
| 3 | Error |
| 4 | Warning |
| 5 | Notice |
| 6 | Informational |
| 7 | Debug |

<details>
<summary>Sample config</summary>

```bash
~: cat /etc/rsyslog.conf
# /etc/rsyslog.conf configuration file for rsyslog
#
# For more information install rsyslog-doc and see
# /usr/share/doc/rsyslog-doc/html/configuration/index.html
#
# Default logging rules can be found in /etc/rsyslog.d/50-default.conf


#################
#### MODULES ####
#################

module(load="imuxsock") # provides support for local system logging
#module(load="immark")  # provides --MARK-- message capability

# provides UDP syslog reception
#module(load="imudp")
#input(type="imudp" port="514")

# provides TCP syslog reception
#module(load="imtcp")
#input(type="imtcp" port="514")

# provides kernel logging support and enable non-kernel klog messages
module(load="imklog" permitnonkernelfacility="on")

###########################
#### GLOBAL DIRECTIVES ####
###########################

#
# Use traditional timestamp format.
# To enable high precision timestamps, comment out the following line.
#
$ActionFileDefaultTemplate RSYSLOG_TraditionalFileFormat

# Filter duplicated messages
$RepeatedMsgReduction on

#
# Set the default permissions for all log files.
#
$FileOwner syslog
$FileGroup adm
$FileCreateMode 0640
$DirCreateMode 0755
$Umask 0022
$PrivDropToUser syslog
$PrivDropToGroup syslog

#
# Where to place spool and state files
#
$WorkDirectory /var/spool/rsyslog

#
# Include all config files in /etc/rsyslog.d/
#
$IncludeConfig /etc/rsyslog.d/*.conf

###############
#### RULES ####
###############

#
# Log anything besides private authentication messages to a single log file
#
*.*;auth,authpriv.none      -/var/log/syslog

#
# Log commonly used facilities to their own log file
#
auth,authpriv.*         /var/log/auth.log
cron.*              -/var/log/cron.log
kern.*              -/var/log/kern.log
mail.*              -/var/log/mail.log
user.*              -/var/log/user.log

#
# Emergencies are sent to everybody logged in.
#
*.emerg             :omusrmsg:*
```

</details>

</br>

**Configuration file has three parts:**

1. **MODULES**, modules used. For example to let the rsyslog use UDP connections
2. **GLOBAL DIRECTIVES**, general configurations like directories accesses
3. **RULES**, A combination of facilities, priorities and actions tells the rsyslog what to do with each log.

### logger

To send log to *rsyslog* we can use *logger* tool.

```bash
~: logger "This is log from mojtaba"

~: tail /var/log/syslog
Dec 15 13:25:18 mojtaba-VirtualBox mojtaba: This is log from mojtaba
```

## systemd-journald

systemd introduces its own logging system called journal. There is no need to run a syslog based service anymore, as all system events are written in the journal.

The journal itself is a system service managed by systemd. Its full name is systemd-journald.service. It collects and stores logging data by maintaining structured indexed journals based on logging information received from the kernel, user processes, standard input, and system service errors.

```bash
~:  systemctl status systemd-journald.service
● systemd-journald.service - Journal Service
     Loaded: loaded (/lib/systemd/system/systemd-journald.service; static)
     Active: active (running) since Tue 2023-12-12 22:20:06 +0330; 2 days ago
TriggeredBy: ● systemd-journald-dev-log.socket
             ● systemd-journald.socket
             ● systemd-journald-audit.socket
       Docs: man:systemd-journald.service(8)
             man:journald.conf(5)
   Main PID: 190 (systemd-journal)
     Status: "Processing requests..."
      Tasks: 1 (limit: 4599)
     Memory: 42.5M
        CPU: 6.662s
     CGroup: /system.slice/systemd-journald.service
             └─190 /lib/systemd/systemd-journald
```

The journal stores log data in /run/log/journal/ by default. Because the /run/ directory is volatile by nature, log data is lost at reboot. To make the log data persistent, the directory **/var/log/journal/{machine-id}/** with correct ownership and permissions must exist, where the systemd-journald service can store its data. By default systemd will create the directory for you and switch to persistent logging.

The systemd-journald service does not keep separate files, as rsyslog does. The idea is to avoid checking different files for issues. Systemd-journald saves the events and messages in a binary format that cannot be read with a text editor.

We can query the journal with the **journalctl** command.

### Journal configuration

<details>
<summary>Sample configuration</summary>

```bash
~:  cat /etc/systemd/journald.conf
#  This file is part of systemd.
#
#  systemd is free software; you can redistribute it and/or modify it under the
#  terms of the GNU Lesser General Public License as published by the Free
#  Software Foundation; either version 2.1 of the License, or (at your option)
#  any later version.
#
# Entries in this file show the compile time defaults. Local configuration
# should be created by either modifying this file, or by creating "drop-ins" in
# the journald.conf.d/ subdirectory. The latter is generally recommended.
# Defaults can be restored by simply deleting this file and all drop-ins.
#
# Use 'systemd-analyze cat-config systemd/journald.conf' to display the full config.
#
# See journald.conf(5) for details.

[Journal]
#Storage=auto
#Compress=yes
#Seal=yes
#SplitMode=uid
#SyncIntervalSec=5m
#RateLimitIntervalSec=30s
#RateLimitBurst=10000
#SystemMaxUse=
#SystemKeepFree=
#SystemMaxFileSize=
#SystemMaxFiles=100
#RuntimeMaxUse=
#RuntimeKeepFree=
#RuntimeMaxFileSize=
#RuntimeMaxFiles=100
#MaxRetentionSec=
#MaxFileSec=1month
#ForwardToSyslog=yes
#ForwardToKMsg=no
#ForwardToConsole=no
#ForwardToWall=yes
#TTYPath=/dev/console
#MaxLevelStore=debug
#MaxLevelSyslog=debug
#MaxLevelKMsg=notice
#MaxLevelConsole=info
#MaxLevelWall=emerg
#LineMax=48K
#ReadKMsg=yes
#Audit=no
```

</details>
</br>

For example with the **Storage** option we can config that logs should be persist or not.

```bash
Storage=volatile     # keep the logs in memory
Storage=persistent   # keep on disk, if needed create the directories
Storage=auto         # will write to disk only if the directory exists
Storage=none         # do not keep logs
```

### journal commands

#### journalctl

If you run the **journalctl** you will get all the logs... too much. So there are some useful switches:

| Switch | Meaning |
| --- | --- |
| -r | show in reverse; newer on top|
| -f | keep showing the tail|
| -e | show and go to the end|
| -n | show this many lines from the end (newest ones)|
| -k | show kernel message (equal to dmesg)|
| -b | show from a specific boot, -1 is the previous one, 0 is the current one. You can check the list using --list-boots|
| -p | from specific priority, say -p err|
| -u | from a specific systemd unit|

You can also use the --since and --until to show a specific time range, It is possible to provide time as YYYY-MM-DD HH:MM:SS or use things like yesterday, today, tomorrow, now or even --since "10 minutes ago" which is equals to --since "-2 minutes".

#### systemd-cat

This tools is used when you want to manually send logs to the journaling system. It sends its input to the journal or if provided, runs the command and sends the result to the journal.

```bash
~: echo "This is my first test" | systemd-cat
~: systemd-cat -p info uptime #sending priority too
~: journalctl -n 3
Jun 30 06:31:01 debian systemd[1]: Finished apt-daily.service - Daily apt download activities.
Jun 30 06:34:18 debian cat[1020]: This is my first test
Jun 30 06:34:48 debian uptime[1022]:  06:34:48 up  1:05,  4 users,  load average: 0.00, 0.00, 0.00
```

### journald and containers

Typically, a Docker container won’t have systemd, because it would make it too “heavy”. As a consequence, it won’t have journald, either. That said, you probably have journald on the host, if the host is running Linux. This means you can use the journald logging driver to send all the logs of a host’s containers to that host’s journal. It’s as easy as:

```bash
~: docker run my_container --log-driver=journald

~: journalctl CONTAINER_NAME=my_container --all
Apr 09 13:03:28 localhost.localdomain dockerd-current[25558]: hello journal
```

If you want to use journald by default, you can make the change in daemon.json and restart Docker:

```bash
~: cat /etc/docker/daemon.json
{
 "log-driver": "journald"
}
~: systemctl restart docker
```
