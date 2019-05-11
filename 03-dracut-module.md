## 3. Добавить `dracut` модуль🐧 в `initrd`

> Конфигурация `initrd` с помощью `dracut`  собирается из модулей.
> Скрипты модулей хранятся в каталоге `/usr/lib/dracut/modules.d/`

### Действия:
Создать новый каталог и перейти в него.
```php
mkdir /usr/lib/dracut/modules.d/01test  && cd "$_"
```
Создать файлы модуля.
```php
vi module-setup.sh
```
```php
vi test.sh
```

