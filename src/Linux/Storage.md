# Storage

**List drives(block devices):**

```bash
 # block devices are shown with a b at the first column
 ~: ls /dev/ -l | grep "^b"
brw-rw----  1 root disk      8,   0 Feb  3  2023 sda
brw-rw----  1 root disk      8,  16 Feb  3  2023 sdb
brw-rw----  1 root disk      8,  17 Feb  3  2023 sdb1
brw-rw----  1 root disk      8,  18 Feb  3  2023 sdb2
brw-rw----  1 root disk      8,  19 Feb  3  2023 sdb3
brw-rw----+ 1 root cdrom    11,   0 Feb  3  2023 sr0
```

```bash
 ~: lsblk
NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
vda    252:0    0    25G  0 disk
├─vda1 252:1    0   550M  0 part /boot/efi
├─vda2 252:2    0     8M  0 part
└─vda3 252:3    0  24.5G  0 part /

```

```bash
 ~: lsblk -f
NAME   FSTYPE   FSVER LABEL           UUID                                 FSAVAIL FSUSE% MOUNTPOINTS
vda
├─vda1 vfat     FAT32 MKFS_ESP        61F5-D9AE                             548.6M     0% /boot/efi
├─vda2
└─vda3 ext4     1.0   cloudimg-rootfs e4ae5b75-468e-4eed-8567-9816c908aeaf   16.8G    22% /
```

```bash
 ~: df -Th
Filesystem     Type   Size  Used Avail Use% Mounted on
tmpfs          tmpfs   96M  1.4M   95M   2% /run
/dev/vda3      ext4    23G  5.1G   17G  24% /
tmpfs          tmpfs  479M     0  479M   0% /dev/shm
tmpfs          tmpfs  5.0M     0  5.0M   0% /run/lock
/dev/vda1      vfat   549M  296K  549M   1% /boot/efi
tmpfs          tmpfs   96M  4.0K   96M   1% /run/user/1000
```

```bash
 # display all partitions info
 ~: sudo fdisk -l /dev/sda
Disk /dev/sda: 200 GiB, 214748364800 bytes, 419430400 sectors
Disk model: Virtual disk
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: 205BF1DC-5415-4AAA-A17F-179C4A6A13D3

Device       Start       End   Sectors  Size Type
/dev/sda1     2048      4095      2048    1M BIOS boot
/dev/sda2     4096   4198399   4194304    2G Linux filesystem
/dev/sda3  4198400 419428351 415229952  198G Linux filesystem

 # display one partition info
 ~: sudo fdisk -l /dev/sda3
Disk /dev/sda3: 198 GiB, 212597735424 bytes, 415229952 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
```

**List disks with uuid:**

```bash
 ~: lsblk -o+uuid
NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS       UUID
vda    252:0    0    25G  0 disk
├─vda1 252:1    0   550M  0 part /boot/efi         61F5-D9AE
├─vda2 252:2    0     8M  0 part
└─vda3 252:3    0  24.5G  0 part /                 e4ae5b75-468e-4eed-8567-9816c908aeaf
```

**File and Directory Disk Usage:**

```bash
 # Check storage usage of specific dir
 ~: du -h -s  dir_name

 # To list usage individually
 ~: du -h dir_name

 # Sort based on usage
 ~: sudo du -ah  dir_name  | sort -n -r

 # Sort and print top 10
 ~: sudo du -ah   dir_name |   sort -n -r   |    head -n 10

 # display only a total for each argument
 ~: du -sh dir_name
```

**Detect new disks:**

```bash
 ~: for host in /sys/class/scsi_host/*; do echo "- - -" | sudo tee $host/scan; ls /dev/sd* ; done
```

**Partition Table:**

It is possible to create partitions on a block device and even split it and use it as multiple disks. Systems with old BIOS boot loaders use the Master Boot Record (MBR) method for patitioning and newer UEFI systems use GUID Parition Table (GPT) formats.

Linux systems use **udev** to add block devices and their paritions to the /dev in the form of /dev/sdb1 (2nd disk (b) and first parition (1)).

we can use *fdisk* in an interactive mode:

```bash
 # select disk e.g sdc
 ~: sudo fdisk /dev/sdc
 Command (m for help): # p will print the partition table info

```

<details>
<summary> Create new partition</summary>

```bash
 # select disk
 ~: sudo fdisk /dev/sdc
 # 1. Run the  n command to create a new partition.
 Command (m for help): n
 # 2. Select Partition type.
 Select (default p): 
 # 3. Select the partition number, press enter for default.
 Partition number (2-4, default 2):
 # 4. After that, you are asked for the starting and ending sector of your hard drive. press enter for default.
 First sector (2099200-8388607, default 2099200):
 # 5. The last prompt is related to the size of the partition. You can choose to have several sectors or to set the size in megabytes or gigabytes. e.g Type +20GB to set the size of the partition to 20GB.
 Last sector, +/-sectors or +/-size{K,M,G,T,P} (2099200-8388607, default 8388607): +20GB
 # 6. write the changes with w.
 Command (m for help): w
 # 7. print partitions info to verify
 Command (m for help): p
```

```bash
 # Format partition
 ~: sudo mkfs -t ext4 /dev/sdc1
```

</details>

<br/>
**Mount:**

```bash
 ~: mount -v -t nfs4 -o vers=4,loud  10.0.2.15:/dir  /nfs_share
```

**ISCSI:**

```bash
 ~: sudo iscsiadm -m discovery -t st -p 192.168.21.20

 ~: iscsiadm -m session
 ~: iscsiadm -m session -o show

 ~: lsblk

 ~: mount /dev/sdb1 /mnt/lun1

 ~: sudo systemctl status iscsi.service

 ~: sudo iscsiadm --mode node --targetname iqn.2020-04.netbina:swarm --portal 192.168.21.20 -u
 ~: sudo iscsiadm --mode node --targetname iqn.2020-04.netbina:swarm --portal 192.168.21.20 -o delete

 ~: sudo umount /dev/sdb1

 ~: sudo ls /etc/iscsi/send_targets/
 ~: sudo ls /etc/iscsi/nodes/

 ~: sudo rm -dfr /etc/iscsi/send_targets/
 ~: sudo rm -dfr /etc/iscsi/nodes/
```
