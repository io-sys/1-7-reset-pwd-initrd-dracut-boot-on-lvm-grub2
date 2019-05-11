## 3. Добавить `dracut` модуль🐧 в `initrd`

> Конфигурация `initrd` с помощью `dracut`  собирается из модулей.
> Скрипты модулей хранятся в каталоге `/usr/lib/dracut/modules.d/`

### Действия:
Создать новый каталог для модуля и перейти в него.
```php
mkdir /usr/lib/dracut/modules.d/01test  && cd "$_"
```
__Создать файлы модуля `01test`:__
```php
vi module-setup.sh
#!/bin/bash

check() {
    return 0
}

depends() {
    return 0
}

install() {
    inst_hook cleanup 00 "${moddir}/test.sh"
}
```
Скрипт, который будет вызываться, в нём рисуется 🐧пингвин.
```php
vi test.sh
#!/bin/bash

exec 0<>/dev/console 1<>/dev/console 2<>/dev/console
cat <<'EOF'
Hello! You are in dracut module!
 ___________________
< I'm dracut module >
 -------------------
   \
    \
        .--.
       |o_o |
       |:_/ |
      //   \ \
     (|     | )
    /'\_   _/`\
    \___)=(___/
EOF
sleep 10
echo " continuing...."
```

Пересобрать образ `initrd`
```php
dracut -f -v 
...
*** Creating image file ***
*** Creating image file done ***
*** Creating initramfs image file '/boot/initramfs-3.10.0-862.2.3.el7.x86_64.img' done ***
```
Проверить, что модуль `test` загружен в образ `initrd`
```php
lsinitrd -m /boot/initramfs-$(uname -r).img | grep te
test
terminfo
systemd
```

