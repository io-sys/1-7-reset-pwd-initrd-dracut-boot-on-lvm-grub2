## 2. Установить систему на LVM, после чего переименовать VG

Показать все `vg`
```php
[root@linux]# vgs
  VG         #PV #LV #SN Attr   VSize   VFree
  VolGroup00   1   2   0 wz--n- <38.97g    0
[root@linux]#
```
Переименовать `vg` с именем `VolGroup00` на `OtusRoot`
```php
[root@linux]# vgrename VolGroup00 OtusRoot
  Volume group "VolGroup00" successfully renamed to "OtusRoot"
[root@linux]#
```
> Далее правим:
>	1. `/etc/fstab`
>	2. `/etc/default/grub`
>	3. `/boot/grub2/grub.cfg`  
> ─ везде заменяем старое название на новое. Примечание: `/etc/grub2.cfg` ссылка на `/boot/grub2/grub.cfg` 
>  
> Для редактирования файлов воспользуюсь `sed` с параметрами `s` (`substitute`) заменить и  
> `g` (`global`) - без оператора, операция замены будет производиться только для первого найденного совпадения,
> с заданным шаблоном, в каждой строке.  

Редактировать, заменить имя `Volume Group` старое `VolGroup00` на новое `OtusRoot` в файлах:
```php
[root@linux]# sed -i.bak01 s/VolGroup00/OtusRoot/g /etc/fstab
[root@linux]# sed -i.bak01 s/VolGroup00/OtusRoot/g /etc/default/grub
[root@linux]# sed -i.bak01 s/VolGroup00/OtusRoot/g /boot/grub2/grub.cfg
```

Пересоздать `initrd` образ, чтобы применить изменения в имени `Volume Group` или не загрузится система.
```php
[root@linux]# mkinitrd -f -v /boot/initramfs-$(uname -r).img $(uname -r)
....
*** Creating image file ***
*** Creating image file done ***
*** Creating initramfs image file '/boot/initramfs-3.10.0-862.2.3.el7.x86_64.img' done ***
[root@linux]# 
```
Перезагрузиться с кнопки.
