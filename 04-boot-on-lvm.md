## 4* Перенести /boot на LVM

Проверить установленную версию `GRUB2`
```php
[root@linux]# rpm -q grub2
grub2-2.02-0.65.el7.centos.2.x86_64
[root@linux]# 
```
Подключение репозитория https://yum.rumyantsev.com/centos/7/x86_64/
```php
vi /etc/yum.repos.d/grublvm.repo
[rumyantsev]
name=yum.rumyantsev.com
baseurl=https://yum.rumyantsev.com/centos/7/x86_64/
```

