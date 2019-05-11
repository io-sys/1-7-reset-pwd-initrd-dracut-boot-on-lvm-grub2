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

Редактируется `/etc/fstab`  добавляется `/dev/mapper/OtusRoot-LogVolBoot`
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
/dev/mapper/OtusRoot-LogVolBoot /boot                   xfs     defaults        0 0
/dev/mapper/OtusRoot-LogVol01 swap                    swap    defaults        0 0
[root@linux]# 
```
Редактируется `/etc/default/grub` добавляется `rd.lvm.lv=OtusRoot/LogVolBoot`
```php
[root@linux]# vi /etc/default/grub
GRUB_TIMEOUT=1
GRUB_DISTRIBUTOR="$(sed 's, release .*$,,g' /etc/system-release)"
GRUB_DEFAULT=saved
GRUB_DISABLE_SUBMENU=true
GRUB_TERMINAL_OUTPUT="console"
GRUB_CMDLINE_LINUX="no_timer_check console=tty0 console=ttyS0,115200n8 net.ifnames=0 biosdevname=0 elevator=noop crashkernel=auto rd.lvm.lv=OtusRoot/LogVolBoot rd.lvm.lv=OtusRoot/LogVol00 rd.lvm.lv=OtusRoot/LogVol01 rhgb quiet"
GRUB_DISABLE_RECOVERY="true"
[root@linux]# 
```
Показать `UUID` разделов `/dev/sda2` и `LogVolBoot`, нужны для редактирования `/boot/grub2/grub.cfg`
```php
[root@linux]# blkid
/dev/mapper/OtusRoot-LogVol00: UUID="b60e9498-0baa-4d9f-90aa-069048217fee" TYPE="xfs"
/dev/sda3: UUID="vrrtbx-g480-HcJI-5wLn-4aOf-Olld-rC03AY" TYPE="LVM2_member"
/dev/sdb: UUID="mePb3p-gNuS-pZxJ-VYao-cVqt-3z0F-XkJMI1" TYPE="LVM2_member"
/dev/sda2: UUID="570897ca-e759-4c81-90cf-389da6eee4cc" TYPE="xfs"
/dev/mapper/OtusRoot-LogVol01: UUID="c39c5bed-f37c-4263-bee8-aeb6a6659d7b" TYPE="swap"
/dev/mapper/OtusRoot-LogVolBoot: UUID="86ec917f-ae2c-45c0-ac56-65e94e47fe63" TYPE="xfs"
[root@linux]# 
```
Редактирование `/boot/grub2/grub.cfg`, делаю  замену `UUID` от `/boot` старого на новый.
```php
[root@linux]# sed -i.bak01 s/"570897ca-e759-4c81-90cf-389da6eee4cc"/"86ec917f-ae2c-45c0-ac56-65e94e47fe63"/g /boot/grub2/grub.cfg
```

Редактирование /boot/grub2/grub.cfg добавлен `rd.lvm.lv=OtusRoot/LogVolBoot`
```php
[root@linux]# vi /boot/grub2/grub.cfg
...
          search --no-floppy --fs-uuid --set=root --hint='lvmid/SA8LTU-F2yz-FEV1-RdgT-hw0Z-iRxh-yHFKuU/Jd6uB7-q8Q5-qmZv-
5FEV-t2zq-AFeL-8fGu7s'  86ec917f-ae2c-45c0-ac56-65e94e47fe63
        else
          search --no-floppy --fs-uuid --set=root 86ec917f-ae2c-45c0-ac56-65e94e47fe63
        fi
        linux16 /vmlinuz-3.10.0-862.2.3.el7.x86_64 root=/dev/mapper/OtusRoot-LogVol00 ro no_timer_check console=tty0 con
sole=ttyS0,115200n8 net.ifnames=0 biosdevname=0 elevator=noop crashkernel=auto rd.lvm.lv=OtusRoot/LogVolBoot rd.lvm.lv=O
tusRoot/LogVol00 rd.lvm.lv=OtusRoot/LogVol01 rhgb quiet
        initrd16 /initramfs-3.10.0-862.2.3.el7.x86_64.img
...
[root@linux]# 
```
__Создается конфиг `GRUB2`__
```php
[root@linux]# grub2-mkconfig -o /etc/grub2.cfg
```
__Образ `initrd` узнает о новых `lvm`  томах и их именах__
```php
[root@linux]# dracut -f -v
```
Копировать директорию `/boot` на новый `LogVolBoot`
```php
[root@linux]# cp -aR /boot/* /mnt/boot
```
__При перезагрузке будет использован `GRUB2` и `UUID` старого `/boot` раздела на `/dev/sda2`__
```php
[root@linux]# reboot
```
Проверить монтированные разделы.
```php
[root@linux]# lsblk
NAME                  MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda                     8:0    0   40G  0 disk
├─sda1                  8:1    0    1M  0 part
├─sda2                  8:2    0    1G  0 part
└─sda3                  8:3    0   39G  0 part
  ├─OtusRoot-LogVol00 253:1    0 37.5G  0 lvm  /
  └─OtusRoot-LogVol01 253:2    0  1.5G  0 lvm  [SWAP]
sdb                     8:16   0    1G  0 disk
└─OtusRoot-LogVolBoot 253:0    0  992M  0 lvm  /boot
[root@linux]# 
```
__Удалить старый `/boot`, заполнить его 0'и__
```php
[root@linux]# dd if=/dev/zero of=/dev/sda2
dd: writing to ‘/dev/sda2’: No space left on device
2097153+0 records in
2097152+0 records out
1073741824 bytes (1.1 GB) copied, 34.2592 s, 31.3 MB/s
[root@linux]# 
```
__Надо обязательно восстановить `GRUB2` на `sda` после заполнения 0'и раздела `/dev/sda2` или не взлетит.__
```php
[root@linux]# grub2-install /dev/sda
Installing for i386-pc platform.
Installation finished. No error reported.
[root@linux]#
```
Перезагрузка
```php
[root@linux]# reboot
```
Проверка после перезагрузки примонтированных разделов. 
`/dev/sda2` без ФС, был стерт.
```php
[root@linux]# lsblk -fs
NAME                FSTYPE      LABEL UUID                                   MOUNTPOINT
sda1
└─sda
sda2
└─sda
OtusRoot-LogVol00   xfs               b60e9498-0baa-4d9f-90aa-069048217fee   /
└─sda3              LVM2_member       vrrtbx-g480-HcJI-5wLn-4aOf-Olld-rC03AY
  └─sda
OtusRoot-LogVol01   swap              c39c5bed-f37c-4263-bee8-aeb6a6659d7b   [SWAP]
└─sda3              LVM2_member       vrrtbx-g480-HcJI-5wLn-4aOf-Olld-rC03AY
  └─sda
OtusRoot-LogVolBoot xfs               86ec917f-ae2c-45c0-ac56-65e94e47fe63   /boot
└─sdb               LVM2_member       mePb3p-gNuS-pZxJ-VYao-cVqt-3z0F-XkJMI1
[root@linux]# 
```

