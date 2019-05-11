### Разница между методами получения шелла в процессе загрузки.

(нумерация по методичке)
1) `init=/bin/sh`  - во время загрузки системы `Unix` должен запуститься первый процесс после старта ядра системы, 
традиционно этим первым процессом является процесс `init`. 
Когда передается свой параметр вместо `init`, указывается, что вместо первого процесса `init` надо запустить `/bin/sh`

2) `rd.break` прерывание на ранней стадии процесса загрузки, на этапе `initrd`. Корень `/(root)` будет находиться в `/sysroot` 
Запускается `shell` перед выполнением функции `pivot_root()`


#### 1) Попасть в систему без пароля несколькими способами
`init=/bin/bash`

> Из параметров загрузки `GRUB2` из строки `linux16`, были удалены все лишние ключи, которые мешали загрузке ядра в `emergence mode`, 
> не тронут `ro` и добавлен `init=/bin/bash`

__ В GRUB2__ 'e' (edit)

__Было:__
```php
linux16 /vmlinuz-3.10.0-862.2.3.el7.x86_64 root=/dev/mapper/VolGroup00-LogVol00 ro no_timer_check console=tty0 console=ttyS0,115200n8 net.ifnames=0 biosdevname=0 elevator=noop crashkernel=auto rd.lvm.lv=VolGroup00/LogVol00 rd.lvm.lv=VolGroup00/LogVol01 rhgb quiet 
```
__Стало:__
```php
linux16 /vmlinuz-3.10.0-862.2.3.el7.x86_64 root=/dev/mapper/VolGroup00-LogVol00 ro init=/bin/bash  rd.lvm.lv=VolGroup00/LogVol00 rd.lvm.lv=VolGroup00/LogVol01
```

__В emergence mode__
Перемонтировать для чтения и записи, чтобы изменить пароль
```php
mount -o remount, rw /
```
Проверить маркеры SELinux
```php
ls -Z /etc/shadow
```
---------- root root ?                                /etc/shadow
