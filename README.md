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
