# Домашнее задание 3: работа с LVM

## Задание
 На виртуальной машине с Ubuntu 24.04 и LVM.<br>
 Уменьшить том под / до 8G.<br>
 Выделить том под /home.<br>
 Выделить том под /var - сделать в mirror.<br>
 /home - сделать том для снапшотов.<br>
 Прописать монтирование в fstab. Попробовать с разными опциями и разными файловыми системами (на выбор).<br>
 Работа со снапшотами:<br>
  сгенерить файлы в /home/;<br>
  снять снапшот;<br>
  удалить часть файлов;<br>
  восстановиться со снапшота.<br>


## Выполнение

### Уменьшить том под / до 8G
```
root@otus-homework:~# lsblk
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda      8:0    0   16G  0 disk 
├─sda1   8:1    0    1M  0 part 
├─sda2   8:2    0  100M  0 part /boot/efi
└─sda3   8:3    0 15.9G  0 part /
sdb      8:16   0   20G  0 disk 
sdc      8:32   0   20G  0 disk 
sdd      8:48   0   20G  0 disk 
sde      8:64   0   20G  0 disk 
root@otus-homework:~# pvcreate /dev/sdb
  Physical volume "/dev/sdb" successfully created.
root@otus-homework:~# vgcreate vg_root /dev/sdb
  Volume group "vg_root" successfully created
root@otus-homework:~# lvcreate -n lv_root -l +100%FREE /dev/vg_root
  Logical volume "lv_root" created.
root@otus-homework:~# mkfs.ext4 /dev/vg_root/lv_root
mke2fs 1.47.0 (5-Feb-2023)
Discarding device blocks: done                            
Creating filesystem with 5241856 4k blocks and 1310720 inodes
Filesystem UUID: df5f720f-358d-4a6b-b31e-78e0d19d0dd0
Superblock backups stored on blocks: 
  32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208, 
  4096000

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (32768 blocks): done
Writing superblocks and filesystem accounting information: done   

root@otus-homework:~# mount /dev/vg_root/lv_root /mnt
root@otus-homework:~# lsblk
NAME              MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda                 8:0    0   16G  0 disk 
├─sda1              8:1    0    1M  0 part 
├─sda2              8:2    0  100M  0 part /boot/efi
└─sda3              8:3    0 15.9G  0 part /
sdb                 8:16   0   20G  0 disk 
└─vg_root-lv_root 252:0    0   20G  0 lvm  /mnt
sdc                 8:32   0   20G  0 disk 
sdd                 8:48   0   20G  0 disk 
sde                 8:64   0   20G  0 disk 



root@otus-homework:~# rsync -avxHAX --progress / /mnt/

sent 2,300,638,585 bytes  received 1,526,787 bytes  28,247,427.88 bytes/sec
total size is 2,298,983,810  speedup is 1.00

root@otus-homework:~# ls /mnt
bin                dev   lib64              mnt   run                 srv  var
bin.usr-is-merged  etc   lib.usr-is-merged  opt   sbin                sys
boot               home  lost+found         proc  sbin.usr-is-merged  tmp
cdrom              lib   media              root  snap                usr

root@otus-homework:~# for i in /proc/ /sys/ /dev/ /run/ /boot/; \
 do mount --bind $i /mnt/$i; done
root@otus-homework:~# ls -la /mnt
total 80
drwxr-xr-x  23 root root  4096 Jul  9  2024 .
drwxr-xr-x  22 root root   325 Jul  9  2024 ..
lrwxrwxrwx   1 root root     7 Apr 22  2024 bin -> usr/bin
drwxr-xr-x   2 root root  4096 Feb 26  2024 bin.usr-is-merged
drwxr-xr-x   4 root root   236 Jul  9  2024 boot
dr-xr-xr-x   2 root root  4096 Apr 23  2024 cdrom
drwxr-xr-x  21 root root  4220 Apr 15 11:49 dev
drwxr-xr-x 110 root root  4096 Apr 15 11:42 etc
drwxr-xr-x   2 root root  4096 Jul  9  2024 home
lrwxrwxrwx   1 root root     7 Apr 22  2024 lib -> usr/lib
lrwxrwxrwx   1 root root     9 Apr 22  2024 lib64 -> usr/lib64
drwxr-xr-x   2 root root  4096 Feb 26  2024 lib.usr-is-merged
drwx------   2 root root 16384 Apr 15 11:49 lost+found
drwxr-xr-x   2 root root  4096 Apr 23  2024 media
drwxr-xr-x   2 root root  4096 Apr 15 11:49 mnt
drwxr-xr-x   2 root root  4096 Apr 23  2024 opt
dr-xr-xr-x 181 root root     0 Apr 15 11:42 proc
drwx------   4 root root  4096 Apr 15 11:43 root
drwxr-xr-x  28 root root   900 Apr 15 11:43 run
lrwxrwxrwx   1 root root     8 Apr 22  2024 sbin -> usr/sbin
drwxr-xr-x   2 root root  4096 Apr  3  2024 sbin.usr-is-merged
drwxr-xr-x   2 root root  4096 Jul  9  2024 snap
drwxr-xr-x   2 root root  4096 Apr 23  2024 srv
dr-xr-xr-x  13 root root     0 Apr 15 11:51 sys
drwxrwxrwt  12 root root  4096 Apr 15 11:43 tmp
drwxr-xr-x  12 root root  4096 Apr 23  2024 usr
drwxr-xr-x  13 root root  4096 Jul  9  2024 var
root@otus-homework:~# ls /mnt
bin                dev   lib64              mnt   run                 srv  var
bin.usr-is-merged  etc   lib.usr-is-merged  opt   sbin                sys
boot               home  lost+found         proc  sbin.usr-is-merged  tmp
cdrom              lib   media              root  snap                usr
root@otus-homework:~# chroot /mnt
root@otus-homework:/# grub-mkconfig -o /boot/grub/grub.cfg
Sourcing file `/etc/default/grub'
Generating grub configuration file ...
Found linux image: /boot/vmlinuz-6.8.0-36-generic
Found initrd image: /boot/initrd.img-6.8.0-36-generic
Warning: os-prober will not be executed to detect other bootable partitions.
Systems on them will not be added to the GRUB boot configuration.
Check GRUB_DISABLE_OS_PROBER documentation entry.
Adding boot menu entry for UEFI Firmware Settings ...
done
root@otus-homework:/# update-initramfs -u
update-initramfs: Generating /boot/initrd.img-6.8.0-36-generic
root@otus-homework:/# 

root@otus-homework:/# lsblk
NAME              MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda                 8:0    0   16G  0 disk 
├─sda1              8:1    0    1M  0 part 
├─sda2              8:2    0  100M  0 part 
└─sda3              8:3    0 15.9G  0 part /boot
sdb                 8:16   0   20G  0 disk 
└─vg_root-lv_root 252:0    0   20G  0 lvm  /
sdc                 8:32   0   20G  0 disk 
sdd                 8:48   0   20G  0 disk 
sde                 8:64   0   20G  0 disk 


root@otus-homework:/# pvcreate /dev/sdc
  Physical volume "/dev/sdc" successfully created.
root@otus-homework:/# vgcreate ubuntu-vg /dev/sdc
  Volume group "ubuntu-vg" successfully created
root@otus-homework:/# lvcreate -n ubuntu-vg/ubuntu-lv -L 8G /dev/ubuntu-vg
  Logical volume "ubuntu-lv" created.
root@otus-homework:/# mkfs.ext4 /dev/ubuntu-vg/ubuntu-lv
mke2fs 1.47.0 (5-Feb-2023)
Discarding device blocks: done                            
Creating filesystem with 2097152 4k blocks and 524288 inodes
Filesystem UUID: a903f07b-d1d8-4f5d-baa3-97539e1981b1
Superblock backups stored on blocks: 
  32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (16384 blocks): done
Writing superblocks and filesystem accounting information: done 

root@otus-homework:/# mount /dev/ubuntu-vg/ubuntu-lv /mnt
root@otus-homework:/# lsblk
NAME                    MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda                       8:0    0   16G  0 disk 
├─sda1                    8:1    0    1M  0 part 
├─sda2                    8:2    0  100M  0 part 
└─sda3                    8:3    0 15.9G  0 part /boot
sdb                       8:16   0   20G  0 disk 
└─vg_root-lv_root       252:0    0   20G  0 lvm  /
sdc                       8:32   0   20G  0 disk 
└─ubuntu--vg-ubuntu--lv 252:1    0    8G  0 lvm  /mnt
sdd                       8:48   0   20G  0 disk 
sde                       8:64   0   20G  0 disk 
root@otus-homework:/# 

root@otus-homework:/# rsync -avxHAX --progress / /mnt/

sent 2,202,082,218 bytes  received 1,520,987 bytes  28,805,270.65 bytes/sec
total size is 2,200,184,312  speedup is 1.00


root@otus-homework:/# for i in /proc/ /sys/ /dev/ /run/ /boot/; \
 do mount --bind $i /mnt/$i; done

 root@otus-homework:/# lsblk
NAME                    MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda                       8:0    0   16G  0 disk 
├─sda1                    8:1    0    1M  0 part 
├─sda2                    8:2    0  100M  0 part 
└─sda3                    8:3    0 15.9G  0 part /mnt/boot
                                                 /boot
sdb                       8:16   0   20G  0 disk 
└─vg_root-lv_root       252:0    0   20G  0 lvm  /
sdc                       8:32   0   20G  0 disk 
└─ubuntu--vg-ubuntu--lv 252:1    0    8G  0 lvm  /mnt
sdd                       8:48   0   20G  0 disk 
sde                       8:64   0   20G  0 disk 
root@otus-homework:/# chroot /mnt
root@otus-homework:/# lsblk
NAME                    MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda                       8:0    0   16G  0 disk 
├─sda1                    8:1    0    1M  0 part 
├─sda2                    8:2    0  100M  0 part 
└─sda3                    8:3    0 15.9G  0 part /boot
sdb                       8:16   0   20G  0 disk 
└─vg_root-lv_root       252:0    0   20G  0 lvm  
sdc                       8:32   0   20G  0 disk 
└─ubuntu--vg-ubuntu--lv 252:1    0    8G  0 lvm  /
sdd                       8:48   0   20G  0 disk 
sde                       8:64   0   20G  0 disk 
root@otus-homework:/# 


root@otus-homework:/# grub-mkconfig -o /boot/grub/grub.cfg
Sourcing file `/etc/default/grub'
Generating grub configuration file ...
Found linux image: /boot/vmlinuz-6.8.0-36-generic
Found initrd image: /boot/initrd.img-6.8.0-36-generic
Warning: os-prober will not be executed to detect other bootable partitions.
Systems on them will not be added to the GRUB boot configuration.
Check GRUB_DISABLE_OS_PROBER documentation entry.
Adding boot menu entry for UEFI Firmware Settings ...
done


root@otus-homework:/# update-initramfs -u
update-initramfs: Generating /boot/initrd.img-6.8.0-36-generic

```



### Выделить том под /home
```
root@otus-homework:/# lvcreate -n LogVol_Home -L 2G /dev/ubuntu-vg
  Logical volume "LogVol_Home" created.
root@otus-homework:/# mkfs.ext4 /dev/ubuntu-vg/LogVol_Home
mke2fs 1.47.0 (5-Feb-2023)
Discarding device blocks: done                            
Creating filesystem with 524288 4k blocks and 131072 inodes
Filesystem UUID: f952eb37-99cb-4204-b057-7a16728232e6
Superblock backups stored on blocks: 
  32768, 98304, 163840, 229376, 294912

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (16384 blocks): done
Writing superblocks and filesystem accounting information: done 

root@otus-homework:/# mount /dev/ubuntu-vg/LogVol_Home /mnt
root@otus-homework:~# mount /dev/ubuntu-vg/LogVol_Home /mnt/
root@otus-homework:~# cp -aR /home/* /mnt/
root@otus-homework:~# ls -la /home
total 8
drwxr-xr-x  2 root root 4096 Apr 15 12:30 .
drwxr-xr-x 23 root root 4096 Apr 15 12:29 ..
-rw-r--r--  1 root root    0 Apr 15 12:30 file1
-rw-r--r--  1 root root    0 Apr 15 12:30 file10
-rw-r--r--  1 root root    0 Apr 15 12:30 file2
-rw-r--r--  1 root root    0 Apr 15 12:30 file3
-rw-r--r--  1 root root    0 Apr 15 12:30 file4
-rw-r--r--  1 root root    0 Apr 15 12:30 file5
-rw-r--r--  1 root root    0 Apr 15 12:30 file6
-rw-r--r--  1 root root    0 Apr 15 12:30 file7
-rw-r--r--  1 root root    0 Apr 15 12:30 file8
-rw-r--r--  1 root root    0 Apr 15 12:30 file9
root@otus-homework:~# ls -la /mnt
total 8
drwxr-xr-x  2 root root 4096 Apr 15 12:35 .
drwxr-xr-x 23 root root 4096 Apr 15 12:29 ..
-rw-r--r--  1 root root    0 Apr 15 12:30 file1
-rw-r--r--  1 root root    0 Apr 15 12:30 file10
-rw-r--r--  1 root root    0 Apr 15 12:30 file2
-rw-r--r--  1 root root    0 Apr 15 12:30 file3
-rw-r--r--  1 root root    0 Apr 15 12:30 file4
-rw-r--r--  1 root root    0 Apr 15 12:30 file5
-rw-r--r--  1 root root    0 Apr 15 12:30 file6
-rw-r--r--  1 root root    0 Apr 15 12:30 file7
-rw-r--r--  1 root root    0 Apr 15 12:30 file8
-rw-r--r--  1 root root    0 Apr 15 12:30 file9
root@otus-homework:~# rm -rf /home/*
root@otus-homework:~# umount /mnt
root@otus-homework:~# ls -la /home
total 8
drwxr-xr-x  2 root root 4096 Apr 15 12:35 .
drwxr-xr-x 23 root root 4096 Apr 15 12:29 ..
root@otus-homework:~# mount /dev/ubuntu-vg/LogVol_Home /home/
root@otus-homework:~# ls -la /home
total 8
drwxr-xr-x  2 root root 4096 Apr 15 12:35 .
drwxr-xr-x 23 root root 4096 Apr 15 12:29 ..
-rw-r--r--  1 root root    0 Apr 15 12:30 file1
-rw-r--r--  1 root root    0 Apr 15 12:30 file10
-rw-r--r--  1 root root    0 Apr 15 12:30 file2
-rw-r--r--  1 root root    0 Apr 15 12:30 file3
-rw-r--r--  1 root root    0 Apr 15 12:30 file4
-rw-r--r--  1 root root    0 Apr 15 12:30 file5
-rw-r--r--  1 root root    0 Apr 15 12:30 file6
-rw-r--r--  1 root root    0 Apr 15 12:30 file7
-rw-r--r--  1 root root    0 Apr 15 12:30 file8
-rw-r--r--  1 root root    0 Apr 15 12:30 file9
root@otus-homework:~# echo "`blkid | grep Home | awk '{print $2}'` \
 /home xfs defaults 0 0" >> /etc/fstab

root@otus-homework:~# cat /etc/fstab
# /etc/fstab: static file system information.
#
# Use 'blkid' to print the universally unique identifier for a
# device; this may be used with UUID= as a more robust way to name devices
# that works even if disks are added and removed. See fstab(5).
#
# <file system> <mount point>   <type>  <options>       <dump>  <pass>
# / was on /dev/vda3 during curtin installation
/dev/disk/by-uuid/80469199-1ae8-4cef-ac54-5d9bd0adcfc7 / xfs defaults 0 1
# /boot/efi was on /dev/vda2 during curtin installation
/dev/disk/by-uuid/aef83a3e-d677-49bd-80c6-bebbe0aa66c8 /boot/efi ext4 defaults 0 1
UUID="e9333ab1-78e0-41ee-a513-9d6aaecaa361"  /var ext4 defaults 0 0
UUID="f952eb37-99cb-4204-b057-7a16728232e6"  /home xfs defaults 0 0
```



### Выделить том под /var в зеркало
```
root@otus-homework:/# pvcreate /dev/sd{d,e}
  Physical volume "/dev/sdd" successfully created.
  Physical volume "/dev/sde" successfully created.
root@otus-homework:/# vgcreate vg_var /dev/sd{d,e}
  Volume group "vg_var" successfully created
root@otus-homework:/# lvcreate -L 950M -m 1 -n lv_var vg_var
  Rounding up size to full physical extent 952.00 MiB
  Logical volume "lv_var" created.
root@otus-homework:/# mkfs.ext4 /dev/vg_var/lv_var
mke2fs 1.47.0 (5-Feb-2023)
Discarding device blocks: done                            
Creating filesystem with 243712 4k blocks and 60928 inodes
Filesystem UUID: e9333ab1-78e0-41ee-a513-9d6aaecaa361
Superblock backups stored on blocks: 
  32768, 98304, 163840, 229376

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (4096 blocks): done
Writing superblocks and filesystem accounting information: done

root@otus-homework:/# mount /dev/vg_var/lv_var /mnt


root@otus-homework:/# cp -aR /var/* /mnt
root@otus-homework:/# mkdir /tmp/oldvar && mv /var/* /tmp/oldvar
root@otus-homework:/# umount /mnt
root@otus-homework:/# mount /dev/vg_var/lv_var /var
root@otus-homework:/# echo "`blkid | grep var: | awk '{print $2}'` \
 /var ext4 defaults 0 0" >> /etc/fstab
root@otus-homework:/# cat /etc/fstab
# /etc/fstab: static file system information.
#
# Use 'blkid' to print the universally unique identifier for a
# device; this may be used with UUID= as a more robust way to name devices
# that works even if disks are added and removed. See fstab(5).
#
# <file system> <mount point>   <type>  <options>       <dump>  <pass>
# / was on /dev/vda3 during curtin installation
/dev/disk/by-uuid/80469199-1ae8-4cef-ac54-5d9bd0adcfc7 / xfs defaults 0 1
# /boot/efi was on /dev/vda2 during curtin installation
/dev/disk/by-uuid/aef83a3e-d677-49bd-80c6-bebbe0aa66c8 /boot/efi ext4 defaults 0 1
UUID="e9333ab1-78e0-41ee-a513-9d6aaecaa361"  /var ext4 defaults 0 0
```




### /home - сделать том для снапшотов.
```
root@otus-homework:~# lvcreate -n home_snap-L 1G /dev/ubuntu-vg
  No command with matching syntax recognised.  Run 'lvcreate --help' for more information.
root@otus-homework:~# lvcreate -n home_snap -L 1G /dev/ubuntu-vg
  Logical volume "home_snap" created.
root@otus-homework:~# lsblk
NAME                     MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda                        8:0    0   16G  0 disk 
├─sda1                     8:1    0    1M  0 part 
├─sda2                     8:2    0  100M  0 part /boot/efi
└─sda3                     8:3    0 15.9G  0 part 
sdb                        8:16   0   20G  0 disk 
├─vg_var-lv_var_rmeta_1  252:8    0    4M  0 lvm  
│ └─vg_var-lv_var        252:10   0  952M  0 lvm  /var
└─vg_var-lv_var_rimage_1 252:9    0  952M  0 lvm  
  └─vg_var-lv_var        252:10   0  952M  0 lvm  /var
sdc                        8:32   0   20G  0 disk 
├─vg_var-lv_var_rmeta_0  252:6    0    4M  0 lvm  
│ └─vg_var-lv_var        252:10   0  952M  0 lvm  /var
└─vg_var-lv_var_rimage_0 252:7    0  952M  0 lvm  
  └─vg_var-lv_var        252:10   0  952M  0 lvm  /var
sdd                        8:48   0   20G  0 disk 
├─ubuntu--vg-ubuntu--lv  252:1    0    8G  0 lvm  /
├─ubuntu--vg-home_snap   252:2    0    1G  0 lvm  
└─ubuntu--vg-LogVol_Home 252:4    0    2G  0 lvm  /home
sde                        8:64   0   20G  0 disk 
└─vg_root-lv_root        252:0    0   20G  0 lvm 

```




### Работа со снапшотами
```
root@otus-homework:~# touch /home/file{1..20}
root@otus-homework:~# lsblk
NAME                     MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda                        8:0    0   16G  0 disk 
├─sda1                     8:1    0    1M  0 part 
├─sda2                     8:2    0  100M  0 part /boot/efi
└─sda3                     8:3    0 15.9G  0 part 
sdb                        8:16   0   20G  0 disk 
├─vg_var-lv_var_rmeta_1  252:8    0    4M  0 lvm  
│ └─vg_var-lv_var        252:10   0  952M  0 lvm  /var
└─vg_var-lv_var_rimage_1 252:9    0  952M  0 lvm  
  └─vg_var-lv_var        252:10   0  952M  0 lvm  /var
sdc                        8:32   0   20G  0 disk 
├─vg_var-lv_var_rmeta_0  252:6    0    4M  0 lvm  
│ └─vg_var-lv_var        252:10   0  952M  0 lvm  /var
└─vg_var-lv_var_rimage_0 252:7    0  952M  0 lvm  
  └─vg_var-lv_var        252:10   0  952M  0 lvm  /var
sdd                        8:48   0   20G  0 disk 
├─ubuntu--vg-ubuntu--lv  252:1    0    8G  0 lvm  /
├─ubuntu--vg-home_snap   252:2    0    1G  0 lvm 
└─ubuntu--vg-LogVol_Home 252:4    0    2G  0 lvm  /home
sde                        8:64   0   20G  0 disk 
└─vg_root-lv_root        252:0    0   20G  0 lvm  
root@otus-homework:~# lvcreate -L 100MB -s -n home_snap \
 /dev/ubuntu-vg/LogVol_Home
  Logical volume "home_snap" created.
root@otus-homework:~# rm -f /home/file{11..20}
root@otus-homework:~# ls -la /home
total 8
drwxr-xr-x  2 root root 4096 Apr 15 12:36 .
drwxr-xr-x 23 root root 4096 Apr 15 12:29 ..
-rw-r--r--  1 root root    0 Apr 15 12:36 file1
-rw-r--r--  1 root root    0 Apr 15 12:36 file10
-rw-r--r--  1 root root    0 Apr 15 12:36 file2
-rw-r--r--  1 root root    0 Apr 15 12:36 file3
-rw-r--r--  1 root root    0 Apr 15 12:36 file4
-rw-r--r--  1 root root    0 Apr 15 12:36 file5
-rw-r--r--  1 root root    0 Apr 15 12:36 file6
-rw-r--r--  1 root root    0 Apr 15 12:36 file7
-rw-r--r--  1 root root    0 Apr 15 12:36 file8
-rw-r--r--  1 root root    0 Apr 15 12:36 file9
root@otus-homework:~# umount /home
root@otus-homework:~# lvconvert --merge /dev/ubuntu-vg/home_snap
  Merging of volume ubuntu-vg/home_snap started.
  ubuntu-vg/LogVol_Home: Merged: 100.00%
root@otus-homework:~# mount /dev/mapper/ubuntu--vg-LogVol_Home /home
mount: (hint) your fstab has been modified, but systemd still uses
       the old version; use 'systemctl daemon-reload' to reload.
root@otus-homework:~# ls -la /home
total 8
drwxr-xr-x  2 root root 4096 Apr 15 12:36 .
drwxr-xr-x 23 root root 4096 Apr 15 12:29 ..
-rw-r--r--  1 root root    0 Apr 15 12:36 file1
-rw-r--r--  1 root root    0 Apr 15 12:36 file10
-rw-r--r--  1 root root    0 Apr 15 12:36 file11
-rw-r--r--  1 root root    0 Apr 15 12:36 file12
-rw-r--r--  1 root root    0 Apr 15 12:36 file13
-rw-r--r--  1 root root    0 Apr 15 12:36 file14
-rw-r--r--  1 root root    0 Apr 15 12:36 file15
-rw-r--r--  1 root root    0 Apr 15 12:36 file16
-rw-r--r--  1 root root    0 Apr 15 12:36 file17
-rw-r--r--  1 root root    0 Apr 15 12:36 file18
-rw-r--r--  1 root root    0 Apr 15 12:36 file19
-rw-r--r--  1 root root    0 Apr 15 12:36 file2
-rw-r--r--  1 root root    0 Apr 15 12:36 file20
-rw-r--r--  1 root root    0 Apr 15 12:36 file3
-rw-r--r--  1 root root    0 Apr 15 12:36 file4
-rw-r--r--  1 root root    0 Apr 15 12:36 file5
-rw-r--r--  1 root root    0 Apr 15 12:36 file6
-rw-r--r--  1 root root    0 Apr 15 12:36 file7
-rw-r--r--  1 root root    0 Apr 15 12:36 file8
-rw-r--r--  1 root root    0 Apr 15 12:36 file9
root@otus-homework:~# 
```
