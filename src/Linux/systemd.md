# systemd

Owner: Mojtaba Safaei
Verification: Verified

The systemd is made around **unit**s. A unit can be a service, a group of services, or an action. Units do have a name, a type, and a configuration file. There are 12 unit types: automount, device, mount, path, scope, service, slice, snapshot, socket, swap, target & timer.

```bash
# systemctl list-units
# systemctl list-units --type=service  # will show all services
# systemctl list-units --type=target  # will show all targets
# systemctl get-default   #default target (groups of services are started via target unit files)
# systemctl cat ntpd.service
# systemctl daemon-reload sshd # re-reads the configuration of the systemd configs of this service

```

The units can be found in these places (sorted by priority):

1. `/etc/systemd/system/`
2. `/run/systemd/system/`
3. `/usr/lib/systemd/system`
4. `/lib/systemd/system`

**How target works:**

services running in a runlevel are controlled from these directories: `/etc/rc[0-6].d`

in these directories we have some links refers to a script in`/etc/init.d`

You can find the scripts files in `/etc/init.d` and runlevels in `/etc/rc[0-6].d` directories where S indicates Start and K indicates Kill.

And start/stop on runlevels are controlled from these directories: `/etc/rc[0-6].d`

```bash
user@vm:~$ ls -lthr /etc/rc3.d/
total 0
lrwxrwxrwx 1 root root 14 Apr 28  2023 S01dbus -> ../init.d/dbus
lrwxrwxrwx 1 root root 14 Apr 28  2023 S01cron -> ../init.d/cron
lrwxrwxrwx 1 root root 26 Apr 28  2023 S01console-setup.sh -> ../init.d/console-setup.sh
lrwxrwxrwx 1 root root 15 Apr 28  2023 S01uuidd -> ../init.d/uuidd
lrwxrwxrwx 1 root root 15 Apr 28  2023 S01rsync -> ../init.d/rsync
lrwxrwxrwx 1 root root 20 Apr 28  2023 S01irqbalance -> ../init.d/irqbalance
lrwxrwxrwx 1 root root 29 Apr 28  2023 S01unattended-upgrades -> ../init.d/unattended-upgrades
lrwxrwxrwx 1 root root 23 Apr 28  2023 S01open-vm-tools -> ../init.d/open-vm-tools
lrwxrwxrwx 1 root root 13 Apr 28  2023 S01ssh -> ../init.d/ssh
lrwxrwxrwx 1 root root 18 Apr 28  2023 S01plymouth -> ../init.d/plymouth
lrwxrwxrwx 1 root root 25 Apr 28  2023 S01multipath-tools -> ../init.d/multipath-tools
lrwxrwxrwx 1 root root 16 Apr 28  2023 S01apport -> ../init.d/apport
lrwxrwxrwx 1 root root 23 Apr 28  2023 S01lvm2-lvmpolld -> ../init.d/lvm2-lvmpolld
lrwxrwxrwx 1 root root 15 May  2  2023 S01acpid -> ../init.d/acpid
lrwxrwxrwx 1 root root 18 May  2  2023 S01watchdog -> ../init.d/watchdog
lrwxrwxrwx 1 root root 22 May  2  2023 S01wd_keepalive -> ../init.d/wd_keepalive
lrwxrwxrwx 1 root root 26 May  2  2023 S01qemu-guest-agent -> ../init.d/qemu-guest-agent
lrwxrwxrwx 1 root root 21 May  2  2023 S01grub-common -> ../init.d/grub-common
```

**Targets (Runlevel):**

| SysV runlevel | systemd target | Purpose |
| --- | --- | --- |
| 0 | runlevel0.target, halt.target, poweroff.target | System shutdown |
| 1, S | runlevel1.target, rescue.target, | Single-user mode |
| 2 | runlevel2.target, multi-user.target, | Local multiuser without remote network |
| 3 | runlevel3.target, multi-user.target, | Full multiuser with network |
| 4 | runlevel4.target | Unused/User-defined |
| 5 | runlevel5.target, graphical.target, | Full multiuser with network and display manager |
| 6 | runlevel6.target, reboot.target, | System reboot |

**Commands to change targets:**

| Task | systemd Command | System V init Command |
| --- | --- | --- |
| Change the current target/runlevel | systemctl isolate TARGET-NAME.target | telinit X |
| Change to the default target/runlevel | systemctl default | n/a |
| Get the current target/runlevel | systemctl list-units --type=target
With systemd there is usually more than one active target. The command lists all currently active targets. | who -r
or
runlevel |
| persistently change the default runlevel | Use the Services Manager or run the following command:
ln -sf /usr/lib/systemd/system/ TARGET-NAME.target /etc/systemd/system/default.target | Use the Services Manager or change the line
id: X:initdefault:
in /etc/inittab |
| Change the default runlevel for the current boot process | Enter the following option at the boot prompt
systemd.unit= TARGET-NAME.target | Enter the desired runlevel number at the boot prompt. |
| Show a target's/runlevel's dependencies | systemctl show -p "Requires" TARGET-NAME.target
systemctl show -p "Wants" TARGET-NAME.target | n/a |

```bash
user@vm:~$ systemctl cat multi-user.target
# /lib/systemd/system/multi-user.target
#  SPDX-License-Identifier: LGPL-2.1-or-later
#
#  This file is part of systemd.
#
#  systemd is free software; you can redistribute it and/or modify it
#  under the terms of the GNU Lesser General Public License as published by
#  the Free Software Foundation; either version 2.1 of the License, or
#  (at your option) any later version.

[Unit]
Description=Multi-User System
Documentation=man:systemd.special(7)
Requires=basic.target
Conflicts=rescue.service rescue.target
After=basic.target rescue.service rescue.target
AllowIsolate=yes
```

```bash
user@vm:~$ systemctl show --property="Wants" rescue.target | xargs echo
Wants=grub-initrd-fallback.service systemd-update-utmp-runlevel.service
```

```bash
user@vm:~$ systemctl show --property="Requires" rescue.target | xargs echo
Requires=rescue.service sysinit.target
```

```bash
switch to multi-user target
user@vm:~$ systemctl isolate multi-user.target

switch to graphical target
user@vm:~$ systemctl isolate graphical.target
```

Find and read a service file:

```bash
user@vm:~$ systemctl cat sshd.service
# /lib/systemd/system/ssh.service
[Unit]
Description=OpenBSD Secure Shell server
Documentation=man:sshd(8) man:sshd_config(5)
After=network.target auditd.service
ConditionPathExists=!/etc/ssh/sshd_not_to_be_run

[Service]
EnvironmentFile=-/etc/default/ssh
ExecStartPre=/usr/sbin/sshd -t
ExecStart=/usr/sbin/sshd -D $SSHD_OPTS
ExecReload=/usr/sbin/sshd -t
ExecReload=/bin/kill -HUP $MAINPID
KillMode=process
Restart=on-failure
RestartPreventExitStatus=255
Type=notify
RuntimeDirectory=sshd
RuntimeDirectoryMode=0755

[Install]
WantedBy=multi-user.target
Alias=sshd.service
```

**Journaling**

```
# journalctl # show all journal
# journalctl --no-pager # do not use less
# journalctl -n 10 # only 10 lines
# journalctl -S -1d # last 1 day
# journalctl -xe # last few logs
# journalctl -u ntp # only ntp unit
# journalctl _PID=1234
```