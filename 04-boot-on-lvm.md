## 4* Перенести /boot на LVM

Показать текущую версию `GRUB2`
```php
[root@linux]# rpm -q grub2
grub2-2.02-0.65.el7.centos.2.x86_64
[root@linux]# 
```
Подключение репозитория `https://yum.rumyantsev.com/centos/7/x86_64/`
```php
[root@linux]# vi /etc/yum.repos.d/grublvm.repo
[rumyantsev]
name=yum.rumyantsev.com
baseurl=https://yum.rumyantsev.com/centos/7/x86_64/
```
Показать репозитории и их статусы.
```php
[root@linux]# yum repolist enable
...
rumyantsev                        yum.rumyantsev.com                                                     enabled:     14
...
[root@linux]# 
```
Установка GRUB2
```php
[root@linux]# yum --enablerepo "rumyantsev" install grub2
```
Показать установленную версию `GRUB2`
```php
[root@linux]# rpm -q grub2
grub2-2.02-0.76.el7.centos.1.x86_64
[root@linux]# 
```
## Установлен нужный пакет GRUB2

Показать блочные устройства в системе.
```php
[root@linux]# lsblk
NAME                  MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda                     8:0    0   40G  0 disk
├─sda1                  8:1    0    1M  0 part
├─sda2                  8:2    0    1G  0 part /boot
└─sda3                  8:3    0   39G  0 part
  ├─OtusRoot-LogVol00 253:0    0 37.5G  0 lvm  /
  └─OtusRoot-LogVol01 253:1    0  1.5G  0 lvm  [SWAP]
sdb                     8:16   0    1G  0 disk
[root@linux]# 
```
Показать существующие `PV`
```php
[root@linux]# pvs
  PV         VG       Fmt  Attr PSize   PFree
  /dev/sda3  OtusRoot lvm2 a--  <38.97g    0
[root@linux]# 
```
Содеется `PV` на `/dev/sdb` и инициализируется с параметром `--bootloaderareasize 1m`
```php
[root@linux]# pvcreate /dev/sdb --bootloaderareasize 1m
Physical volume "/dev/sdb" successfully created.
[root@linux]# 
```
Показать существующие `PV`
```php
[root@linux]# pvs
  PV         VG       Fmt  Attr PSize   PFree
  /dev/sda3  OtusRoot lvm2 a--  <38.97g    0
  /dev/sdb            lvm2 ---    1.00g 1.00g
[root@linux]# 
```
Расширить `VG` на новый `PV`
```php
[root@linux]# vgextend OtusRoot /dev/sdb
  Volume group "OtusRoot" successfully extended
[root@linux]# 
```
Показать существующие `lv` 
```php
[root@linux]# lvs
 LV       VG       Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  LogVol00 OtusRoot -wi-ao---- <37.47g
  LogVol01 OtusRoot -wi-ao----   1.50g
[root@linux]#
```
Создать новый `lv`
```php
[root@linux]# lvcreate -n LogVolRoot -l +100%FREE OtusRoot
  Logical volume "LogVolRoot" created.
[root@linux]# 
```
Форматировать том `LogVolRoot` в `xfs`
```php
[root@linux]# mkfs.xfs /dev/mapper/OtusRoot-LogVolRoot
```
Показать существующие `lv` 
```php
[root@linux]# lvs
  LV         VG       Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  LogVol00   OtusRoot -wi-ao---- <37.47g
  LogVol01   OtusRoot -wi-ao----   1.50g
  LogVolBoot OtusRoot -wi-a----- 992.00m
[root@linux]# 
```
Монтировать `LogVolRoot`
```php
[root@linux]# mkdir /mnt/boot
[root@linux]# mount /dev/OtusRoot/LogVolBoot /mnt/boot
```
> __Дальше редактируются файлы:__  
> 	1. `/etc/fstab`  
> 	2. `/etc/default/grub`  
>   3. `/boot/grub2/grub.cfg`  

Редактируется `/etc/fstab`
```php
[root@linux]# vi /etc/fstab
#
# /etc/fstab
# Created by anaconda on Sat May 12 18:50:26 2018
#
# Accessible filesystems, by reference, are maintained under '/dev/disk'
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info
#
/dev/mapper/OtusRoot-LogVol00 /                       xfs     defaults        0 0
`/dev/mapper/OtusRoot-LogVolBoot` /boot                   xfs     defaults        0 0
/dev/mapper/OtusRoot-LogVol01 swap                    swap    defaults        0 0
[root@linux]# 
```
Редактируется `/etc/default/grub`
```php
[root@linux]# vi /etc/default/grub
GRUB_TIMEOUT=1
GRUB_DISTRIBUTOR="$(sed 's, release .*$,,g' /etc/system-release)"
GRUB_DEFAULT=saved
GRUB_DISABLE_SUBMENU=true
GRUB_TERMINAL_OUTPUT="console"
GRUB_CMDLINE_LINUX="no_timer_check console=tty0 console=ttyS0,115200n8 net.ifnames=0 biosdevname=0 elevator=noop crashkernel=auto rd.lvm.lv=OtusRoot/LogVolBoot `rd.lvm.lv=OtusRoot/LogVol00` rd.lvm.lv=OtusRoot/LogVol01 rhgb quiet"
GRUB_DISABLE_RECOVERY="true"
[root@linux]# 
```
Показать UUID
```php
[root@linux]# blkid
/dev/mapper/OtusRoot-LogVol00: UUID="b60e9498-0baa-4d9f-90aa-069048217fee" TYPE="xfs"
/dev/sda3: UUID="vrrtbx-g480-HcJI-5wLn-4aOf-Olld-rC03AY" TYPE="LVM2_member"
/dev/sdb: UUID="mePb3p-gNuS-pZxJ-VYao-cVqt-3z0F-XkJMI1" TYPE="LVM2_member"
/dev/sda2: `UUID="570897ca-e759-4c81-90cf-389da6eee4cc"` TYPE="xfs"
/dev/mapper/OtusRoot-LogVol01: UUID="c39c5bed-f37c-4263-bee8-aeb6a6659d7b" TYPE="swap"
/dev/mapper/OtusRoot-LogVolBoot: `UUID="86ec917f-ae2c-45c0-ac56-65e94e47fe63"` TYPE="xfs"
[root@linux]# 
```



Редактируется 
```php
[root@linux]# vi /etc/default/grub

```




