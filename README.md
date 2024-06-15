Задание 1. Определение алгоритма с наилучшим сжатием:
Определить какие алгоритмы сжатия поддерживает zfs (gzip, zle, lzjb, lz4);
создать 4 файловых системы на каждой применить свой алгоритм сжатия;
для сжатия использовать либо текстовый файл, либо группу файлов
Смотрим список всех дисков с помощью команды lsblk

![image](https://github.com/DronZap/linux_ZFS/assets/145288949/525fb7fb-2680-483e-a352-ba235c75fe3e)

Создаем 4 рейда 1, в каждом из которых по 2 диска и смотрим о них информацию 
[root@zfs ~]# zpool create otus1 mirror /dev/sdb /dev/sdc
[root@zfs ~]# zpool create otus2 mirror /dev/sdd /dev/sde
[root@zfs ~]# zpool create otus3 mirror /dev/sdf /dev/sdg
[root@zfs ~]# zpool create otus4 mirror /dev/sdh /dev/sdi
[root@zfs ~]# zpool list

![image](https://github.com/DronZap/linux_ZFS/assets/145288949/915a0269-b0c8-4627-8af6-e42a83fa36f5)

Используем разные алгоритмы сжатия для каждой файловой системы
[root@zfs ~]# zfs set compression=lzjb otus1
[root@zfs ~]# zfs set compression=lz4 otus2
[root@zfs ~]# zfs set compression=gzip-9 otus3
[root@zfs ~]# zfs set compression=zle otus4
Проверяем методы сжатия
[root@zfs ~]# zfs get all | grep compression

![image](https://github.com/DronZap/linux_ZFS/assets/145288949/16475d5f-f346-4ca3-978f-bf419c6adfa4)

Скачиваем один и тот же файл в каждый рейд
[root@zfs ~]# for i in {1..4}; do wget -P /otus$i https://gutenberg.org/cache/epub/2600/pg2600.converter.log; done

Проверяем, что файл скачан
[root@zfs ~]# ls -l /otus*

![image](https://github.com/DronZap/linux_ZFS/assets/145288949/f8c84abd-7d90-4ad1-bacc-a88259dee987)

![image](https://github.com/DronZap/linux_ZFS/assets/145288949/59024517-4986-4b60-a6f1-39d3b256f112)

Выясняем, что самый эффективный алгоритм сжатия gzip-9

Задание 2. Определить настройки пула.
С помощью команды zfs import собрать pool ZFS.
Командами zfs определить настройки:
- размер хранилища;
- тип pool;
- значение recordsize;
- какое сжатие используется;   
- какая контрольная сумма используется.

Скачиваем архив в домашний каталог и разархивируем его
[root@zfs ~]# wget -O archive.tar.gz --no-check-certificate 'https://drive.usercontent.google.com/download?id=1MvrcEp-WgAQe57aDEzxSRalPAwbNN1Bb&export=download'
[root@zfs ~]# tar -xzvf archive.tar.gz

Проверяем возможно ли импортировать данный каталог в пул
[root@zfs ~]# zpool import -d zpoolexport/

![image](https://github.com/DronZap/linux_ZFS/assets/145288949/32a278dc-6435-4407-8d86-d17d1e5c0786)

Производим импорт данного пула в ОС
[root@zfs ~]# zpool import -d zpoolexport/ otus

![image](https://github.com/DronZap/linux_ZFS/assets/145288949/4b8136a9-15d8-4e0f-8ed6-d8464c5e6ed4)

Смотрим все параметры файловой системы
[root@zfs ~]# zfs get all otus

Определяем размер хранилища
[root@zfs ~]# zfs get available otus

![image](https://github.com/DronZap/linux_ZFS/assets/145288949/bc793537-a8dd-4f18-b5aa-929ecc6af34d)

Определяем тип pool
[root@zfs ~]# zfs get readonly otus

![image](https://github.com/DronZap/linux_ZFS/assets/145288949/c6a19045-334d-49c6-90c9-41f0ac0087c8)

Определяем значение recordsize
[root@zfs ~]# zfs get recordsize otus

![image](https://github.com/DronZap/linux_ZFS/assets/145288949/47cceb06-d502-440b-8054-8b4300aa5992)

Определеяем какое сжатие используется
[root@zfs ~]# zfs get compression otus

![image](https://github.com/DronZap/linux_ZFS/assets/145288949/e6803010-3c0d-4640-9bed-5e222c0fcaad)

Определяем какая контрольная сумма используется
[root@zfs ~]# zfs get checksum otus

![image](https://github.com/DronZap/linux_ZFS/assets/145288949/ddd3ab7f-925e-4ee7-ac04-759f7d36b843)

Задание 3.Работа со снапшотами:
скопировать файл из удаленной директории;
восстановить файл локально. zfs receive;
найти зашифрованное сообщение в файле secret_message.

Скачиваем файл
[root@zfs ~]# wget -O otus_task2.file --no-check-certificate https://drive.usercontent.google.com/download?id=1wgxjih8YZ-cqLqaZVa0lA3h3Y029c3oI&export=download

Восстанавливаем файловую систему из снапшота
[root@zfs ~]# zfs receive otus/test@today < otus_task2.file

Далее, ищем в каталоге /otus/test файл с именем “secret_message” и смотрим содержимое этого файла:

![image](https://github.com/DronZap/linux_ZFS/assets/145288949/beacc470-9153-4803-80e2-e891908b28b4)
