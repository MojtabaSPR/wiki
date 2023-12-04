# Storage


**List drives:**

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
```
    
**Detect new disks:**
    
```bash
 ~: for host in /sys/class/scsi_host/*; do echo "- - -" | sudo tee $host/scan; ls /dev/sd* ; done
```
    

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