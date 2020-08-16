Перенос работающей системы с однго диска на RAID 1.
В системе есть 2 диска
sda (40G) основной на котором работает ОС Linux, и sdb (40G) пустой.

Выполним все команды из root
[vagrant@raid ~]$ sudo -i

Убедимся что текущий системный диск это sda:
[root@raid ~]$ df -h | grep -Po "/dev/..."

Создадим идентичную схему разделов как sda
[root@raid ~]$ sfdisk -d /dev/sda | sfdisk /dev/sdb

Преобразуем новых разделов диска sdb в "Linux raid autodetect"
[root@raid ~]$ fdisk /dev/sdb (t->fd->w)

Содадим raid1 для раздела sdb1
[root@raid ~]$ mdadm --create /dev/md0 --level=1 --raid-devices=2 missing /dev/sdb1

Проверим что RAID собрался нормально:
[root@raid ~]$ cat /proc/mdstat

Создадим файловую систему raid 1
[root@raid ~]$ mkfs.xfs /dev/md0

Монтирует raid1 md0 в каталог /mnt
[root@raid ~]$ mount /dev/md0 /mnt/

Копируем существующие данные
[root@raid ~]$ rsync -auxHAXSv --exclude=/dev/* --exclude=/proc/* --exclude=/sys/* --exclude=/tmp/* --exclude=/mnt/*  /* /mnt

Подключаем системные каталоги, входим в новую систему.
Теперь необходимо сделать настройки в новой системе, при этом не трогая оригинал. Для этого подключаем системные каталоги и переходим в новую систему.
[root@raid ~]$ mount --bind /proc /mnt/proc
[root@raid ~]$ mount --bind /dev /mnt/dev
[root@raid ~]$ mount --bind /sys /mnt/sys
[root@raid ~]$ mount --bind /run /mnt/run
[root@raid ~]$ chroot /mnt/

Информация UUID
[root@raid ~]$ blkid /dev/md*

Меняем UUID на md0 в fstab
[root@raid ~]$ nano /etc/fstab

Создание конфигурации mdadm
[root@raid ~]$ mdadm --detail --scan > /etc/mdadm.conf

Резервное копирование текущего и создание новых initramfs
[root@raid ~]$ cp /boot/initramfs-$(uname -r).img /boot/initramfs-$(uname -r).img.bak
[root@raid ~]$ dracut --mdadmconf --fstab --add="mdraid" --filesystems "xfs ext4 ext3 tmpfs devpts sysfs proc" --add-drivers="raid1" --force /boot/initramfs-$(uname -r).img $(uname -r) -M

Добавьте некоторые параметры по умолчанию в grub
[root@raid ~]$ nano /etc/default/grub
GRUB_CMDLINE_LINUX="rd.auto rd.auto=1 rhgb quiet"
GRUB_PRELOAD_MODULES="mdraid1x"

Сделайте новую конфигурацию grub
[root@raid ~]$ grub2-mkconfig -o /boot/grub2/grub.cfg

Установите grub на новый диск /dev/sdb
[root@raid ~]$ grub2-install /dev/sdb

Перезагружаем виртуалку и в биосе выставляем загрузку с sdb

Убедимся что текущая системна md0:
[root@raid ~]$ df -h | grep -Po "/dev/..."

Подготовим диск sda для raid1 
[root@raid ~]# fdisk /dev/sda (d->n->p->t->fd->w)

Добавим sda в raid1
[root@raid ~]# mdadm --manage /dev/md0 --add /dev/sda1

Промерим состояние raid1
[root@raid ~]# watch -n1 "cat /proc/mdstat"

Установите grub на новый диск /dev/sda
[root@raid ~]$ grub2-install /dev/sda

Перезагружаем виртуалку и в биосе выставляем загрузку с sda

Убедимся что текущая системна md0:
[root@raid ~]$ df -h | grep -Po "/dev/..."

На этом все. Перенос рабочей системы на raid1 закончин.

Утилитой asciinema можно воспроизвести файл dz_raid1.cast

Ссылка на методичку:
https://wiki.rtzra.ru/ubuntu/upgrade-hdd-raid
