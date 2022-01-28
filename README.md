# Домашнее задание к занятию "3.5. Файловые системы"

(1)

Изучил.

(2)

Жесткая ссылка имеет те же права доступа, владельца и время последней модификации, что и целевой файл. Жесткая ссылка и файл, для которой она создавалась имеют одинаковые inode.

(3) Запустил VM с новым содержимым Vagrantfile:

```
Disk /dev/sdb: 2.51 GiB, 2684354560 bytes, 5242880 sectors
Disk model: VBOX HARDDISK
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


Disk /dev/sdc: 2.51 GiB, 2684354560 bytes, 5242880 sectors
Disk model: VBOX HARDDISK
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
```

(4) Разбил диск на два раздела

```
root@vagrant:~# lsblk
NAME                      MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
loop0                       7:0    0 55.5M  1 loop /snap/core18/2253
loop1                       7:1    0 70.3M  1 loop /snap/lxd/21029
loop2                       7:2    0 55.5M  1 loop /snap/core18/2284
loop3                       7:3    0 43.4M  1 loop /snap/snapd/14549
loop4                       7:4    0 67.2M  1 loop /snap/lxd/21835
loop5                       7:5    0 61.9M  1 loop /snap/core20/1270
loop6                       7:6    0 43.3M  1 loop /snap/snapd/14295
loop7                       7:7    0 61.9M  1 loop /snap/core20/1328
sda                         8:0    0   64G  0 disk
├─sda1                      8:1    0    1M  0 part
├─sda2                      8:2    0    1G  0 part /boot
└─sda3                      8:3    0   63G  0 part
  └─ubuntu--vg-ubuntu--lv 253:0    0 31.5G  0 lvm  /
sdb                         8:16   0  2.5G  0 disk
├─sdb1                      8:17   0    2G  0 part
└─sdb2                      8:18   0  500M  0 part
sdc                         8:32   0  2.5G  0 disk
```

(5) Скопировал разделы с одного диска на другой

```
root@vagrant:~# sfdisk -d /dev/sdb | sfdisk /dev/sdc
Checking that no-one is using this disk right now ... OK

Disk /dev/sdc: 2.51 GiB, 2684354560 bytes, 5242880 sectors
Disk model: VBOX HARDDISK
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes

>>> Script header accepted.
>>> Script header accepted.
>>> Script header accepted.
>>> Script header accepted.
>>> Created a new DOS disklabel with disk identifier 0xaa6dce09.
/dev/sdc1: Created a new partition 1 of type 'Linux' and of size 2 GiB.
/dev/sdc2: Created a new partition 2 of type 'Linux' and of size 500 MiB.
/dev/sdc3: Done.

New situation:
Disklabel type: dos
Disk identifier: 0xaa6dce09

Device     Boot   Start     End Sectors  Size Id Type
/dev/sdc1          2048 4196351 4194304    2G 83 Linux
/dev/sdc2       4196352 5220351 1024000  500M 83 Linux

The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.
root@vagrant:~# lsblk
NAME                      MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
loop0                       7:0    0 55.5M  1 loop /snap/core18/2253
loop1                       7:1    0 70.3M  1 loop /snap/lxd/21029
loop2                       7:2    0 55.5M  1 loop /snap/core18/2284
loop3                       7:3    0 43.4M  1 loop /snap/snapd/14549
loop4                       7:4    0 67.2M  1 loop /snap/lxd/21835
loop5                       7:5    0 61.9M  1 loop /snap/core20/1270
loop6                       7:6    0 43.3M  1 loop /snap/snapd/14295
loop7                       7:7    0 61.9M  1 loop /snap/core20/1328
sda                         8:0    0   64G  0 disk
├─sda1                      8:1    0    1M  0 part
├─sda2                      8:2    0    1G  0 part /boot
└─sda3                      8:3    0   63G  0 part
  └─ubuntu--vg-ubuntu--lv 253:0    0 31.5G  0 lvm  /
sdb                         8:16   0  2.5G  0 disk
├─sdb1                      8:17   0    2G  0 part
└─sdb2                      8:18   0  500M  0 part
sdc                         8:32   0  2.5G  0 disk
├─sdc1                      8:33   0    2G  0 part
└─sdc2                      8:34   0  500M  0 part
```

(6) Собрал RAID1 из двух дисков по 2G

```
root@vagrant:~# mdadm --create --verbose /dev/md0 -l 1 -n 2 /dev/sd{b1,c1}
mdadm: Note: this array has metadata at the start and
    may not be suitable as a boot device.  If you plan to
    store '/boot' on this device please ensure that
    your boot-loader understands md/v1.x metadata, or use
    --metadata=0.90
mdadm: size set to 2094080K
Continue creating array?
Continue creating array? (y/n) y
mdadm: Defaulting to version 1.2 metadata
mdadm: array /dev/md0 started.
root@vagrant:~# lsblk
NAME                      MAJ:MIN RM  SIZE RO TYPE  MOUNTPOINT
loop0                       7:0    0 55.5M  1 loop  /snap/core18/2253
loop1                       7:1    0 70.3M  1 loop  /snap/lxd/21029
loop2                       7:2    0 55.5M  1 loop  /snap/core18/2284
loop3                       7:3    0 43.4M  1 loop  /snap/snapd/14549
loop4                       7:4    0 67.2M  1 loop  /snap/lxd/21835
loop5                       7:5    0 61.9M  1 loop  /snap/core20/1270
loop6                       7:6    0 43.3M  1 loop  /snap/snapd/14295
loop7                       7:7    0 61.9M  1 loop  /snap/core20/1328
sda                         8:0    0   64G  0 disk
├─sda1                      8:1    0    1M  0 part
├─sda2                      8:2    0    1G  0 part  /boot
└─sda3                      8:3    0   63G  0 part
  └─ubuntu--vg-ubuntu--lv 253:0    0 31.5G  0 lvm   /
sdb                         8:16   0  2.5G  0 disk
├─sdb1                      8:17   0    2G  0 part
│ └─md0                     9:0    0    2G  0 raid1
└─sdb2                      8:18   0  500M  0 part
sdc                         8:32   0  2.5G  0 disk
├─sdc1                      8:33   0    2G  0 part
│ └─md0                     9:0    0    2G  0 raid1
└─sdc2                      8:34   0  500M  0 part
```

(7) Собрал RAID0 на двух оставшихся дисках

```
root@vagrant:~# mdadm --create --verbose /dev/md1 -l 0 -n 2 /dev/sd{b2,c2}
mdadm: chunk size defaults to 512K
mdadm: Defaulting to version 1.2 metadata
mdadm: array /dev/md1 started.
root@vagrant:~# lsblk
NAME                      MAJ:MIN RM  SIZE RO TYPE  MOUNTPOINT
loop0                       7:0    0 55.5M  1 loop  /snap/core18/2253
loop1                       7:1    0 70.3M  1 loop  /snap/lxd/21029
loop2                       7:2    0 55.5M  1 loop  /snap/core18/2284
loop3                       7:3    0 43.4M  1 loop  /snap/snapd/14549
loop4                       7:4    0 67.2M  1 loop  /snap/lxd/21835
loop5                       7:5    0 61.9M  1 loop  /snap/core20/1270
loop6                       7:6    0 43.3M  1 loop  /snap/snapd/14295
loop7                       7:7    0 61.9M  1 loop  /snap/core20/1328
sda                         8:0    0   64G  0 disk
├─sda1                      8:1    0    1M  0 part
├─sda2                      8:2    0    1G  0 part  /boot
└─sda3                      8:3    0   63G  0 part
  └─ubuntu--vg-ubuntu--lv 253:0    0 31.5G  0 lvm   /
sdb                         8:16   0  2.5G  0 disk
├─sdb1                      8:17   0    2G  0 part
│ └─md0                     9:0    0    2G  0 raid1
└─sdb2                      8:18   0  500M  0 part
  └─md1                     9:1    0  996M  0 raid0
sdc                         8:32   0  2.5G  0 disk
├─sdc1                      8:33   0    2G  0 part
│ └─md0                     9:0    0    2G  0 raid1
└─sdc2                      8:34   0  500M  0 part
  └─md1                     9:1    0  996M  0 raid0
```

(8) Создал 2 независимых PV

```
root@vagrant:~# pvcreate /dev/md0 /dev/md1
  Physical volume "/dev/md0" successfully created.
  Physical volume "/dev/md1" successfully created.
root@vagrant:~# pvscan
  PV /dev/sda3   VG ubuntu-vg       lvm2 [<63.00 GiB / <31.50 GiB free]
  PV /dev/md0                       lvm2 [<2.00 GiB]
  PV /dev/md1                       lvm2 [996.00 MiB]
  Total: 3 [<65.97 GiB] / in use: 1 [<63.00 GiB] / in no VG: 2 [<2.97 GiB]
```


(9) Создал общую volume-group

```
root@vagrant:~# vgcreate vol_grp1 /dev/md0 /dev/md1
  Volume group "vol_grp1" successfully created
root@vagrant:~# vgdisplay
  --- Volume group ---
  VG Name               ubuntu-vg
  System ID
  Format                lvm2
  Metadata Areas        1
  Metadata Sequence No  2
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                1
  Open LV               1
  Max PV                0
  Cur PV                1
  Act PV                1
  VG Size               <63.00 GiB
  PE Size               4.00 MiB
  Total PE              16127
  Alloc PE / Size       8064 / 31.50 GiB
  Free  PE / Size       8063 / <31.50 GiB
  VG UUID               aK7Bd1-JPle-i0h7-5jJa-M60v-WwMk-PFByJ7

  --- Volume group ---
  VG Name               vol_grp1
  System ID
  Format                lvm2
  Metadata Areas        2
  Metadata Sequence No  1
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                0
  Open LV               0
  Max PV                0
  Cur PV                2
  Act PV                2
  VG Size               2.96 GiB
  PE Size               4.00 MiB
  Total PE              759
  Alloc PE / Size       0 / 0
  Free  PE / Size       759 / 2.96 GiB
  VG UUID               UXFT36-kCJX-CGhD-mgGk-9sJw-6zKe-Mod0Yq
```


(10) Создал LV размером 100М

```
root@vagrant:~# lvcreate -L 100M vol_grp1 /dev/md1
  Logical volume "lvol0" created.
root@vagrant:~# vgscan
  Found volume group "ubuntu-vg" using metadata type lvm2
  Found volume group "vol_grp1" using metadata type lvm2
root@vagrant:~# vgs
  VG        #PV #LV #SN Attr   VSize   VFree
  ubuntu-vg   1   1   0 wz--n- <63.00g <31.50g
  vol_grp1    2   1   0 wz--n-   2.96g  <2.87g
root@vagrant:~# lvs
  LV        VG        Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  ubuntu-lv ubuntu-vg -wi-ao----  31.50g
  lvol0     vol_grp1  -wi-a----- 100.00m
```

(11)
