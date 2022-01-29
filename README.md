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

(11) Создал mkfs.ext4 ФС на получившемся LV.

```
root@vagrant:~# mkfs.ext4 /dev/vol_grp1/lvol0
mke2fs 1.45.5 (07-Jan-2020)
Creating filesystem with 25600 4k blocks and 25600 inodes

Allocating group tables: done
Writing inode tables: done
Creating journal (1024 blocks): done
Writing superblocks and filesystem accounting information: done
```

(12,13) Смонтировал этот раздел в директорию /tmp/new .  Поместил туда тестовый файл:

```
root@vagrant:~# mkdir /tmp/new
root@vagrant:~# mount /dev/vol_grp1/lvol0 /tmp/new
root@vagrant:~# cd /tmp/new
root@vagrant:/tmp/new# cd ..
root@vagrant:/tmp# cd ..
root@vagrant:/# ls
bin   cdrom  etc   lib    lib64   lost+found  mnt  proc  run   snap  swap.img  tmp  vagrant
boot  dev    home  lib32  libx32  media       opt  root  sbin  srv   sys       usr  var
root@vagrant:/# wget https://mirror.yandex.ru/ubuntu/ls-lR.gz -O /tmp/new/test.gz
--2022-01-29 15:56:34--  https://mirror.yandex.ru/ubuntu/ls-lR.gz
Resolving mirror.yandex.ru (mirror.yandex.ru)... 213.180.204.183, 2a02:6b8::183
Connecting to mirror.yandex.ru (mirror.yandex.ru)|213.180.204.183|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 22010128 (21M) [application/octet-stream]
Saving to: ‘/tmp/new/test.gz’

/tmp/new/test.gz          100%[===================================>]  20.99M  1.24MB/s    in 17s

2022-01-29 15:56:53 (1.22 MB/s) - ‘/tmp/new/test.gz’ saved [22010128/22010128]

root@vagrant:/# ls -l /tmp/new
total 21512
drwx------ 2 root root    16384 Jan 29 15:50 lost+found
-rw-r--r-- 1 root root 22010128 Jan 29 10:46 test.gz
```

(14) вывод lsblk

```
root@vagrant:/# lsblk
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
    └─vol_grp1-lvol0      253:1    0  100M  0 lvm   /tmp/new
sdc                         8:32   0  2.5G  0 disk
├─sdc1                      8:33   0    2G  0 part
│ └─md0                     9:0    0    2G  0 raid1
└─sdc2                      8:34   0  500M  0 part
  └─md1                     9:1    0  996M  0 raid0
    └─vol_grp1-lvol0      253:1    0  100M  0 lvm   /tmp/new
```


(15) Проверил целостность файла

root@vagrant:/# gzip -t /tmp/new/test.gz
root@vagrant:/# echo $?
0
(16) Используя pvmove, переместил содержимое PV с RAID0 на RAID1

```
root@vagrant:/# pvmove /dev/md1 /dev/md0
  /dev/md1: Moved: 16.00%
  /dev/md1: Moved: 100.00%
```

(17) Сделаk --fail на устройство в RAID1 md

```
root@vagrant:/# mdadm --fail /dev/md0 /dev/sdc1
mdadm: set /dev/sdc1 faulty in /dev/md0
root@vagrant:/# mdadm --detail /dev/md0
/dev/md0:
           Version : 1.2
     Creation Time : Fri Jan 28 15:00:56 2022
        Raid Level : raid1
        Array Size : 2094080 (2045.00 MiB 2144.34 MB)
     Used Dev Size : 2094080 (2045.00 MiB 2144.34 MB)
      Raid Devices : 2
     Total Devices : 2
       Persistence : Superblock is persistent

       Update Time : Sat Jan 29 16:33:28 2022
             State : clean, degraded
    Active Devices : 1
   Working Devices : 1
    Failed Devices : 1
     Spare Devices : 0

Consistency Policy : resync

              Name : vagrant:0  (local to host vagrant)
              UUID : 724e3c9b:a23c51bb:e7062ce8:dfdd852a
            Events : 19

    Number   Major   Minor   RaidDevice State
       0       8       17        0      active sync   /dev/sdb1
       -       0        0        1      removed

       1       8       33        -      faulty   /dev/sdc1
```

(18)

```
root@vagrant:/# dmesg |grep md0
[15096.214716] md/raid1:md0: not clean -- starting background reconstruction
[15096.214718] md/raid1:md0: active with 2 out of 2 mirrors
[15096.214742] md0: detected capacity change from 0 to 2144337920
[15096.223788] md: resync of RAID array md0
[15106.901313] md: md0: resync done.
[107048.055557] md/raid1:md0: Disk failure on sdc1, disabling device.
                md/raid1:md0: Operation continuing on 1 devices.
```

(19) Протестировал целостность файла, несмотря на "сбойный" диск

```
root@vagrant:/# gzip -t /tmp/new/test.gz
root@vagrant:/# echo $?
0
```

(20)

```
λ vagrant destroy
    default: Are you sure you want to destroy the 'default' VM? [y/N] y
==> default: Forcing shutdown of VM...
==> default: Destroying VM and associated drives...
```
