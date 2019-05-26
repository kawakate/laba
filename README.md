**Отчет по лабораторной работе №2**
***
<li>Задание №1</li>


4) Выполнили копирование содержимого раздела /boot с диска sda (ssd1) на диск sdb (ssd2) `dd if=/dev/sda1 of=/dev/sdb1`:

![disks](https://github.com/kawakate/laba/blob/master/lab%202/1.4.png)

5) Затем просматриваем, что RAID-массив проинициализирован корректно `cat /proc/mdstat`, в этом файле отражается текущее состояние RAID-массива:

![RAID](https://github.com/kawakate/laba/blob/master/lab%202/1.5.4.png)

Команда `pvs`: отображает информацию о физическом томе, `lvs`: отображает информацию о логическом томе, `vgs`: отображает информацию о группе физических томов, `mount`: просмотр примонтированых устройств.

В итоге проделанного задания мы создали виртуальную машину с дисками ssd1 и ssd2, настроили LVM(Logical Volume Manager), RAID.

<li>Задание №2</li>

4) Проверили статус RAID-массива: `cat /proc/mdstat`:

![2.4](https://github.com/kawakate/laba/blob/master/lab%202/2.4.png)

6) Посмотрели, что новый диск приехал в систему командой `fdisk -l`:

![2.6](https://github.com/kawakate/laba/blob/master/lab%202/2.6.1.png)

 Копируем таблицу разделов со старого диска на новый: `sfdisk -d /dev/sda | sfdisk /dev/sdb`:

![TABLICA](https://github.com/kawakate/laba/blob/master/lab%202/2.6.2.png)

 Посмотрели результат командой `fdisk -l`:

![2.6](https://github.com/kawakate/laba/blob/master/lab%202/2.6.3.png)

 Добавляем в RAID-массив новый диск: `mdadm —manage /dev/md0 —add /dev/sdb` и смотрим результат: `cat /proc/mdstat`:

![2.6.5](https://github.com/kawakate/laba/blob/master/lab%202/2.6.5.png)

7) Cинхронизация разделов, не входящих в RAID, копируя с "живого" диска на новый: `dd if=/dev/sda1 of=/dev/sdb1`:

![noRAID](https://github.com/kawakate/laba/blob/master/lab%202/2.7.png)

9) Выполнили перезагрузку ВМ, для того чтобы убедиться что все работает:

![proverka](https://github.com/kawakate/laba/blob/master/lab%202/2.9.png)

В итоге проделанного задания мы проэмулировали отказ одного из дисков, удалили диск ssd1, сохранён диск ssd2, добавлен диск ssd3.

<li>Задание №3</li>

1) Проэмулировали отказ диска ssd2, удалив из свойств ВМ диск и перезагрузившись:

![ssd2](https://github.com/kawakate/laba/blob/master/lab%202/3.1.png)

4) Добавим один новый ssd диск, назвав его ssd4:

![ssd3](https://github.com/kawakate/laba/blob/master/lab%202/3.4.png)

5) Копируем файловую таблицу со старого диска на новый: `sfdisk -d /dev/sda | sfdisk /dev/sdb`:

![TABLICA2](https://github.com/kawakate/laba/blob/master/lab%202/3.5.1.png)

Выполним команду `lsblk -o NAME,SIZE,FSTYPE,TYPE,MOUNTPOINT` (При выводе мы замечаем, что в новом диске sdb появились разделы: sdb1, sdb2.) и затем с помощью команды dd скопируем данные /boot на новый диск :

![lsblk](https://github.com/kawakate/laba/blob/master/lab%202/3.5.2-3.png)

Устанавливаем загрузчик на новый диск sdb: `grub-install /dev/sdb` - это загрузчик, который загружает нашу операционную систему, и он нам нужен на новом диске после удаления старого. Далее создаем новый рейд-массив(`mdadm --create --verbose /dev/md63 --level=1 --raid-devices=1 /dev/sdb2`. В итоге у нас установлен новый RAID-массив md63, проверяем при помощи команды `cat /proc/mdstat`:

![md63](https://github.com/kawakate/laba/blob/master/lab%202/3.5.6.png)

6) Настраиваем LVM(Logical Volume Manager)

Создаём новый физический том, включив в него ранее созданный RAID-массив: `pvcreate /dev/md63` и проверяем командой `lsblk -o NAME,SIZE,FSTYPE,TYPE,MOUNTPOINT`:

![md633](https://github.com/kawakate/laba/blob/master/lab%202/3.6.1-4.png)

LV var,log,root находятся на диске sda:

![lv](https://github.com/kawakate/laba/blob/master/lab%202/3.6.6.png)

Выполнили перемещение данных со старого диска на новый:

![var](https://github.com/kawakate/laba/blob/master/lab%202/3.6.7.png)

Теперь все данные находятся на одном диске:

![sdb2md63](https://github.com/kawakate/laba/blob/master/lab%202/3.6.8.(2).png)

Изменили Volume Group, удалив из него диск старого raid и также проверили что раздел /boot не пустой :

![LSBLK](https://github.com/kawakate/laba/blob/master/lab%202/3.6.10-11.png)

8) Проверим, что произошло после добавления дисков (диски появились):

![ssds](https://github.com/kawakate/laba/blob/master/lab%202/3.8.png)

9) Когда мы скопировали таблицу разделов со старого диска оказалось, что новый размер не использует весь объем жесткого диска:

![9.2](https://github.com/kawakate/laba/blob/master/lab%202/3.9.2.png)

10-11) Cкопируем загрузочный раздел /boot с диска ssd4 на ssd5 `dd if=/dev/sda1 of=/dev/sdb1` и установим grub на новый диск (ssd5):

![copy](https://github.com/kawakate/laba/blob/master/lab%202/3.10-11.png)

12-13) Сначала изменили размер второго раздела диска ssd5 (sdb), затем перечитали таблицу разделов и провели результат:

![ssd5](https://github.com/kawakate/laba/blob/master/lab%202/3.13.1.png) 

Добавляем новый диск к текущему RAID-массиву, расширяем количество дисков в нашем массиве до 2-х штук.

![two](https://github.com/kawakate/laba/blob/master/lab%202/3.13.2.png)

14-15) Сначала увеличили размер раздела на диске ssd4 (sda), а затем перечитали таблицу разделов и проверили результат:

![ssd4](https://github.com/kawakate/laba/blob/master/lab%202/3.15.png) 

Получили sda2 и sdb2 разделы, которые имеют размер > чем размер RAID-устройство.

16) Расширяем размер RAID 

![R](https://github.com/kawakate/laba/blob/master/lab%202/3.16.png) 

![pv](https://github.com/kawakate/laba/blob/master/lab%202/3.17.png)


Увеличили объём памяти ssd4, ssd5.

18) Завершили миграцию основного массива на новые диски, работа с ssd4 и ssd5 закончена.

![done](https://github.com/kawakate/laba/blob/master/lab%202/3.18.png) 

19) Перемещаем `/var/log` на новые диски, для этого создаем новый RAID-массив, создаём логический том:

![var2](https://github.com/kawakate/laba/blob/master/lab%202/3.19.2.png) 

20) Переносим данные логов со старого раздела на новый:

![log](https://github.com/kawakate/laba/blob/master/lab%202/3.20.png) 

21) Правим /etc/fstab fstab - файл, в котором записываются правила, по которым при загрузке будут смонтированы разделы (поправляем устройство 'system-log' на 'data-var_log') и затем используем команду resize2fs для изменения ФС:

![etc](https://github.com/kawakate/laba/blob/master/lab%202/3.21-22.png)

23) Выполним перезагрузку, а затем проверку, что сделано всё, что хотели: 
Завершили установку: 

![final](https://github.com/kawakate/laba/blob/master/lab%202/3.23.png) 

Завершили установку.
Что было сделано: замена дисков на лету, с добавлением новых дисков и переносом разделов.
