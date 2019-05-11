## 3. Добавить `dracut` модуль🐧 в `initrd`

> Конфигурация `initrd` с помощью `dracut`  собирается из модулей.
> Скрипты модулей хранятся в каталоге `/usr/lib/dracut/modules.d/`

### Действия:
Создать каталог и перейти в него.mkdir /usr/lib/dracut/modules.d/01test  && cd "$_"
```php
vi module-setup.sh
vi test.sh
```

