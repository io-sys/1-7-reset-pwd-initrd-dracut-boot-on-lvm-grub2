### Разница между методами получения шелла в процессе загрузки.

(нумерация по методичке)
1) `init=/bin/sh`  - во время загрузки системы `Unix` должен запуститься первый процесс после старта ядра системы, 
традиционно этим первым процессом является процесс `init`. 
Когда передается свой параметр вместо `init`, указывается, что вместо первого процесса `init` надо запустить `/bin/sh`

2) `rd.break` прерывание на ранней стадии процесса загрузки, на этапе `initrd`. Корень `/(root)` будет находиться в `/sysroot` 
Запускается `shell` перед выполнением функции `pivot_root()`


#### 1) Попасть в систему без пароля несколькими способами
`init=/bin/bash`

Удалены все лишние ключи для загрузки ядра кроме тех, которые указывают, где лежат разделы на lvm и файл ядра Linux
не тронут ключ `ro` и добавлен `init=/bin/bash`
__GRUB2__ `e`
__Было__
```php
linux16 /vmlinuz-3.10.0-862.2.3.el7.x86_64 root=/dev/mapper/VolGroup00-LogVol00 ro no_timer_check console=tty0 console=ttyS0,115200n8 net.ifnames=0 biosdevname=0 elevator=noop crashkernel=auto rd.lvm.lv=VolGroup00/LogVol00 rd.lvm.lv=VolGroup00/LogVol01 rhgb quiet 
```
__Стало__
```php
linux16 /vmlinuz-3.10.0-862.2.3.el7.x86_64 root=/dev/mapper/VolGroup00-LogVol00 ro init=/bin/bash  rd.lvm.lv=VolGroup00/LogVol00 rd.lvm.lv=VolGroup00/LogVol01
```
#
