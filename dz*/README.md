Добавим в Vagrantfile не которые блоки
vi ~/my_project/2_raid/dz/Vagrantfile

config.vm.provision "shell", path: "./scripts/install.sh"

В файле install.sh пропишим все команды для создания raid10


Ссылка на методичку:
1. https://www.vagrantup.com/docs/provisioning/shell.html
2. Методичка (RAID, NVME).pdf
3. Методичка_Дисковая_подсистема_RAID_Linux-5373-0fc5b2.pdf
