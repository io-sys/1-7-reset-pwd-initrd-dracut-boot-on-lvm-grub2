> 1-7 reset-pwd initrd-dracut-🐧module boot-on-lvm-grub2

# Домашнее задание: Работа с загрузчиком

✅  1. [Попасть в систему без пароля несколькими способами.](https://github.com/io-sys/1-7-reset-pwd-initrd-dracut-boot-on-lvm-grub2/blob/master/01-reset-pwd.md)

✅  2. [Установить систему на `LVM`, после чего переименовать `VG`.](https://github.com/io-sys/1-7-reset-pwd-initrd-dracut-boot-on-lvm-grub2/blob/master/02_rename_vg.md)

✅  3. Добавить модуль🐧 в `initrd`.

✅  4*. Сконфигурировать систему без отдельного раздела с `/boot`, а только с `LVM`
Репозиторий с пропатченым `grub`: https://yum.rumyantsev.com/centos/7/x86_64/
`PV` необходимо инициализировать с параметром `--bootloaderareasize 1m`

__Критерии оценки:__ Описать действия, описать разницу между методами получения шелла в процессе загрузки.
Где получится - используем `script`, где не получается - словами или копипастой описываем действия.
