1. Добавить в Vagrantfile еще дисков:

nano ~/my_project/2_raid/dz/Vagrantfile

в раздели ":disks => {" добавим блок

:sata5 => {
                        :dfile => home + '/VirtualBox VMs/disks/sata5.vdi',
                        :size => 250,
                        :port => 5
                }

Собираю RAID10. Для этого командой посмотрю кол-во дисков (нужно 4 шт.)

[vagrant@otuslinux ~]$ lsblk

Занулим на всякий случай суперблоки:

[vagrant@otuslinux ~]$ sudo mdadm --zero-superblock --force /dev/sd{b,c,d,e,}

-------------------------------------------------------------------------------------------------------

2. Создадим REID10

[vagrant@otuslinux ~]$ sudo mdadm --create --verbose /dev/md0 -l 10 -n 4 /dev/sd{b,c,d,e}

-l 10 уровень REID
-n 4 кол-во устройств в RAID

Проверим что RAID собрался нормально:

[vagrant@otuslinux ~]$ cat /proc/mdstat

Personalities : [raid10] 
md0 : active raid10 sde[3] sdd[2] sdc[1] sdb[0]
      507904 blocks super 1.2 512K chunks 2 near-copies [4/4] [UUUU]

-------------------------------------------------------------------------------------------------------

3. Создание конфигурационного файла mdadm.conf
Проверим

[vagrant@otuslinux ~]$ sudo mdadm --detail --scan --verbose

Логимся под root
Создадим каталог mdadm

[root@otuslinux ~]$ mkdir /etc/mdadm

создадим файл mdadm.conf

[root@otuslinux ~]$ echo "DEVICE partitions" > /etc/mdadm/mdadm.conf

[root@otuslinux ~]# mdadm --detail --scan --verbose | awk '/ARRAY/ {print}' >> /etc/mdadm/mdadm.conf

-------------------------------------------------------------------------------------------------------

4. Сломать/починить RAID
Выведим один диск из строя sde

[vagrant@otuslinux ~]$ sudo mdadm /dev/md0 --fail /dev/sde

Проверим состояние RAID

[vagrant@otuslinux ~]$ cat /proc/mdstat

Personalities : [raid10] 
md0 : active raid10 sde[3](F) sdd[2] sdc[1] sdb[0]
      507904 blocks super 1.2 512K chunks 2 near-copies [4/3] [UUU_]

Видно что диск sde со знаком F вышел из строя

Удалим “сломанный” диск sde из массива:

[vagrant@otuslinux ~]$ sudo mdadm /dev/md0 --remove /dev/sde

После замены диска добавив в RAID

[vagrant@otuslinux ~]$ sudo mdadm /dev/md0 --add /dev/sde

Проверим состояние RAID
[
vagrant@otuslinux ~]$ cat /proc/mdstat

-------------------------------------------------------------------------------------------------------

5. Создать GPT на RAID

[vagrant@otuslinux ~]$ sudo parted -s /dev/md0 mklabel gpt

Создаем партиции

[vagrant@otuslinux ~]$ sudo parted /dev/md0 mkpart primary ext4 0% 20%

[vagrant@otuslinux ~]$ sudo parted /dev/md0 mkpart primary ext4 20% 40%

[vagrant@otuslinux ~]$ sudo parted /dev/md0 mkpart primary ext4 40% 60%

[vagrant@otuslinux ~]$ sudo parted /dev/md0 mkpart primary ext4 60% 80%

[vagrant@otuslinux ~]$ sudo parted /dev/md0 mkpart primary ext4 80% 100%

Под root логинемся
создать на этих партициях ФС

[root@otuslinux ~]$ for i in $(seq 1 5); do sudo mkfs.ext4 /dev/md0p$i; done

смонтировать их по каталогам

[root@otuslinux ~]$ mkdir -p /raid/part{1,2,3,4,5}

[root@otuslinux ~]# for i in $(seq 1 5); do mount /dev/md0p$i /raid/part$i; done

Проверяем

[root@otuslinux ~]# lsblk

Утилитой asciinema можно воспроизвести файл dz_raid.cast

методички:
1. Методичка (RAID, NVME).pdf
2. Методичка_Дисковая_подсистема_RAID_Linux-5373-0fc5b2.pdf
