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
> ─ везде заменяем старое название на новое.
> 
> Примечание: `/etc/grub2.cfg` ссылка на `/boot/grub2/grub.cfg`
