## Работа с mdadm
## Добавляем два диска в систему
:sata5 => {
:dfile => './sata5.vdi',
:size => 250,
:port => 5
},

:sata6 => {
:dfile => './sata6.vdi',
:size => 250,
:port => 6
},
## Проверяем блочные устройства
sudo lshw -short | grep disk
## Обнуляем суперблоки
sudo mdadm --zero-superblock --force /dev/sd{b,c,d,e,f,g}
## Создаём новый RAID-массив
sudo mdadm --create --verbose /dev/md0 -l 10 -n 6 /dev/sd{b,c,d,e,f,g}
## Проверим, что RAID собрался корректно
cat /proc/mdstat

sudo mdadm -D /dev/md0
## Проверяем информацию о RAID
sudo mdadm --detail --scan --verbose
## Создаем файл конфигурации для RAID и вносим данные
sudo touch /etc/mdadm/mdadm.conf

sudo echo "DEVICE partitions" > /etc/mdadm/mdadm.conf

sudo mdadm --detail --scan --verbose | awk '/ARRAY/ {print}' >> /etc/mdadm/mdadm.conf
## Удаляем диск из RAID
sudo mdadm /dev/md0 --fail /dev/sde
## Определяем неисправный диск помечен F
cat /proc/mdstat
## В данной команде увидим REMOVED
sudo mdadm -D /dev/md0
## Удаляем "сломанный" диск
sudo mdadm /dev/md0 --remove /dev/sde
## Добавляем новый диск
sudo mdadm /dev/md0 --add /dev/sde
## Проверяем процесс rebuid-a
cat /proc/mdstat

sudo mdadm -D /dev/md0
## Создаем таблицу разделов GPT на RAID
sudo parted -s /dev/md0 mklabel gpt
## Создаём разделы на массиве:
parted /dev/md0 mkpart primary ext4 0% 20%

parted /dev/md0 mkpart primary ext4 20% 40%

parted /dev/md0 mkpart primary ext4 40% 60%

parted /dev/md0 mkpart primary ext4 60% 80%

parted /dev/md0 mkpart primary ext4 80% 100%
## Создаем файловые системы на разделах
sudo for i in $(seq 1 5); do sudo mkfs.ext4 /dev/md0p$i; done
## Монтируем их по каталогам
sudo mkdir -p /raid/part{1,2,3,4,5}

sudo for i in $(seq 1 5); do mount /dev/md0p$i /raid/part$i; done
## Проверяем вывод
df -h
