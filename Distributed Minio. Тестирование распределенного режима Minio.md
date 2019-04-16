Если у вас стоит задача сделать отказоустойчивый кластер совместимый с AWS S3 для тестирования или небольшой нагрузки, или по каким-то причинам вам не подходит Ceph, то стоит попробовать запустить Minio в распределенном режиме. Одно из преимуществ небольшой расход памяти. При тестировании сервис Minio на 1 ноде занимал 44.3 MB.

Minio это простое, быстрое и совместимое с AWS S3 хранилище объектов. Minio создан для размещения неструктурированных данных, таких как фотографии, видеозаписи, файлы журналов, резервные копии, а также образы виртуальных машин и контейнеров. Небольшой размер позволяет включать его в состав стека приложений, аналогичного Node.js, Redis и MySQL. В minio также поддерживается распределенный режим (distributed mode), который предоставляет возможность подключения к одному серверу хранения объектов множества дисков, в том числе расположенных на разных машинах.

В этой статье описывается установка Minio в распределенном режиме и случаи при выходе нод кластера Minio.

RPM для установки minio и minio клиента можно собрать из репозиториев <https://github.com/patsevanton/minio-client-rpm> и <https://github.com/patsevanton/minio-rpm>

Имеется 8 серверов (нод) minio с 1 диском. Сначала протестируем Minio на 4 нодах.

Указываем 4 серверов и путь до данных. По умолчанию /var/minio/data

```
В файле /etc/default/minio правим конфиг:
# Remote volumes to be used for Minio server.
MINIO_VOLUMES=http://dev-tools-minio-apatsev-1/var/minio/data http://dev-tools-minio-apatsev-2/var/minio/data http://dev-tools-minio-apatsev-3/var/minio/data http://dev-tools-minio-apatsev-4/var/minio/data 
# Access Key of the server.
MINIO_ACCESS_KEY=Server-Access-Key
# Secret key of the server.
MINIO_SECRET_KEY=Server-Secret-Key
```

Добавляем адрес и креды в конфиг клиента Minio.

```
miniocli config host add minio-test http:/ip-сервера-где-запущен-minio:9000 Server-Access-Key Server-Secret-Key
```

**По умолчанию Minio разбивает объекты на N/2 данных и N / 2 дисков четности.**

То есть если у вас 6 серверов, то данные хранятся только на 3. На остальных 3 серверах хранятся метаданные.

Но авторы Minio рекомендуют количество нод для кластера Minio кратным 4.

 При загрузке 1ГБ данных, Minio в памяти занимало

```
python ps_mem.py 
 Private  +   Shared  =  RAM used	Program

 44.3 MiB +   5.0 KiB =  44.3 MiB	minio
```



Если выключить/заблокировать 1 сервер Minio. То будет видно чтобы heal показывает 1 сервер Gray и 2 сервера Yellow.

miniocli admin heal восстанавливает кластер Minio.

miniocli admin heal нужно запускать при обновлении, замены диска

```

miniocli admin heal -r minio-test
 ◓  test2
    0/0 objects; 0 B in 1s
    ┌────────┬───┬─────────────────────┐
    │ Green  │ 0 │   0.0%              │
    │ Yellow │ 2 │  66.7% ████████     │
    │ Red    │ 0 │   0.0%              │
    │ Grey   │ 1 │  33.3% ████         │
    └────────┴───┴─────────────────────┘
```

Если 6 серверов запущены и работают, то загрузка и выгрузка 1ГБ будет просходить за 20-24 секунды.

```
time miniocli cp 1GB.bin minio-test/test2
1GB.bin:                   1.00 GiB / 1.00 GiB ┃▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓┃ 100.00% 51.35 MiB/s 19s
real  0m20.024s

time miniocli cp minio-test/test2/1GB.bin .
...01:9000/test2/1GB.bin:  1.00 GiB / 1.00 GiB ┃▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓┃ 100.00% 42.29 MiB/s 24s
real  0m24.567s
```

При выключении 1 ноды из 6, скорость такая же.

```
time miniocli cp 1GB.bin minio-test/test2
1GB.bin:                   1.00 GiB / 1.00 GiB ┃▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓┃ 100.00% 56.91 MiB/s 17s
real  0m18.057s


time miniocli cp minio-test/test2/1GB.bin .
...01:9000/test2/1GB.bin:  1.00 GiB / 1.00 GiB ┃▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓┃ 100.00% 45.90 MiB/s 22s
real  0m22.441s


md5sum 1GB.bin*
b146c143e54a2456e86cc14cdb9a0b8f  1GB.bin
b146c143e54a2456e86cc14cdb9a0b8f  1GB.bin1
```

Если заблокировать (вышли из строя) 2 сервера из 6, то heal будет показывать 2 Red и 1 Gray.

```
miniocli admin heal -r minio-test
 ◓  test2
    0/0 objects; 0 B in 1s
    ┌────────┬───┬─────────────────────┐
    │ Green  │ 0 │   0.0%              │
    │ Yellow │ 0 │   0.0%              │
    │ Red    │ 2 │  66.7% ████████     │
    │ Grey   │ 1 │  33.3% ████         │
    └────────┴───┴─────────────────────┘
```

Видно что 2 ноды недоступны.

```
miniocli admin info minio-test
●  dev-tools-minio-apatsev-1:9000
   Status : online
   Uptime : 4 hours 
  Version : 2019-03-27T22:35:21Z
  Storage : Used 28 KiB
   Drives : 1/1 OK


●  dev-tools-minio-apatsev-2:9000
   Status : offline
    Error : Post http://dev-tools-minio-apatsev-2:9000/minio/peer/v1/serverinfo?: dial tcp 10.233.47.98:9000: connect: connection refused

●  dev-tools-minio-apatsev-3:9000
   Status : offline
    Error : Post http://dev-tools-minio-apatsev-3:9000/minio/peer/v1/serverinfo?: dial tcp 10.233.47.100:9000: connect: connection refused

●  dev-tools-minio-apatsev-4:9000
   Status : online
   Uptime : 4 hours 
  Version : 2019-03-27T22:35:21Z
  Storage : Used 38 KiB
   Drives : 1/1 OK


●  dev-tools-minio-apatsev-5:9000
   Status : online
   Uptime : 4 hours 
  Version : 2019-03-27T22:35:21Z
  Storage : Used 38 KiB
   Drives : 1/1 OK


●  dev-tools-minio-apatsev-6:9000
   Status : online
   Uptime : 4 hours 
  Version : 2019-03-27T22:35:21Z
  Storage : Used 38 KiB
   Drives : 1/1 OK
```

Тестирование с 2 недоступными нодами.

```

time miniocli cp 1GB.bin minio-test/test2
1GB.bin:                   1.00 GiB / 1.00 GiB ┃▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓┃ 100.00% 66.11 MiB/s 15s
real  0m15.697s

time miniocli cp minio-test/test2/1GB.bin .
...01:9000/test2/1GB.bin:  1.00 GiB / 1.00 GiB ┃▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓┃ 100.00% 43.12 MiB/s 23s
real  0m23.844s
```

Если 2 заблокированные ноды ввести в строй, то heal будет 1 Green и 2 Yellow

```
miniocli admin heal -r minio-test
 ◓  test2
    0/0 objects; 0 B in 1s
    ┌────────┬───┬─────────────────────┐
    │ Green  │ 1 │  33.3% ████         │
    │ Yellow │ 2 │  66.7% ████████     │
    │ Red    │ 0 │   0.0%              │
    │ Grey   │ 0 │   0.0%              │
    └────────┴───┴─────────────────────┘
```

Если начать закачку и заблокировать 2 ноды, то скорость падает.

```

time miniocli cp 1GB.bin minio-test/test2
1GB.bin:                   1.00 GiB / 1.00 GiB ┃▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓┃ 100.00% 3.23 MiB/s 5m16s
real	5m16.746s


time miniocli cp minio-test/test2/1GB.bin .
...01:9000/test2/1GB.bin:  1.00 GiB / 1.00 GiB ┃▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓┃ 100.00% 42.32 MiB/s 24s
real	0m24.347s


md5sum 1GB.bin*
912cf2c6ed889b6ffaa66a97ea04efd5  1GB.bin
912cf2c6ed889b6ffaa66a97ea04efd5  1GB.bin1
```

```

export AWS_ACCESS_KEY_ID=Server-Access-Key
export AWS_SECRET_ACCESS_KEY=Server-Secret-Key
goofys --endpoint=http://10.233.47.99:9000 test2 /mnt

mount
test2 on /mnt type fuse (rw,nosuid,nodev,relatime,user_id=0,group_id=0,default_permissions)


time dd if=/dev/urandom of=/mnt/1GB.bin bs=64M count=16 iflag=fullblock
16+0 records in
16+0 records out
1073741824 bytes (1.1 GB) copied, 51.5215 s, 20.8 MB/s

real	0m51.737s
```



Собираем кластер Minio из 4 нод с 2 дисками каждый.

```

miniocli admin info minio-test
●  dev-tools-minio-apatsev-1:9000
   Status : online
   Uptime : 2 minutes 
  Version : 2019-03-27T22:35:21Z
  Storage : Used 352 MiB
   Drives : 2/2 OK


●  dev-tools-minio-apatsev-2:9000
   Status : online
   Uptime : 2 minutes 
  Version : 2019-03-27T22:35:21Z
  Storage : Used 352 MiB
   Drives : 2/2 OK


●  dev-tools-minio-apatsev-3:9000
   Status : online
   Uptime : 2 minutes 
  Version : 2019-03-27T22:35:21Z
  Storage : Used 352 MiB
   Drives : 2/2 OK


●  dev-tools-minio-apatsev-4:9000
   Status : online
   Uptime : 2 minutes 
  Version : 2019-03-27T22:35:21Z
  Storage : Used 352 MiB
   Drives : 2/2 OK
```

Копируем файл 

```

time miniocli cp 1GB.bin minio-test/test2
1GB.bin:                              512.00 MiB / 512.00 MiB ┃▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓┃ 100.00% 33.45 MiB/s 15s
real	0m15.541s
```

Выгружаем файл

```
time miniocli cp minio-test/test2/1GB.bin .
...10.233.47.104:9000/test2/1GB.bin:  512.00 MiB / 512.00 MiB ┃▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓┃ 100.00% 97.57 MiB/s 5s
real	0m5.396s
```



Если выключаем 2 сервера из 4. На каждом сервере по 2 диска.

```

miniocli admin info minio-test
●  dev-tools-minio-apatsev-1:9000
   Status : online
   Uptime : 2 hours 
  Version : 2019-03-27T22:35:21Z
  Storage : Used 176 MiB
   Drives : 2/2 OK


●  dev-tools-minio-apatsev-2:9000
   Status : offline
    Error : Post http://dev-tools-minio-apatsev-2:9000/minio/peer/v1/serverinfo?: dial tcp 10.233.47.106:9000: connect: connection refused

●  dev-tools-minio-apatsev-3:9000
   Status : offline
    Error : Post http://dev-tools-minio-apatsev-3:9000/minio/peer/v1/serverinfo?: dial tcp 10.233.47.105:9000: connect: connection refused

●  dev-tools-minio-apatsev-4:9000
   Status : online
   Uptime : 2 hours 
  Version : 2019-03-27T22:35:21Z
  Storage : Used 176 MiB
   Drives : 2/2 OK
```

То получаем ошибку

```
time miniocli cp 1GB.bin minio-test/test2
miniocli: <ERROR> Failed to copy `1GB.bin`. Multiple disks failures, unable to write data.
miniocli: <ERROR> Session safely terminated. To resume session `mc session resume gHlaSKoR
```

Minio не проверяет название дубликата загружаемого файла на сервер.

Если запустить загрузку файлов с одинаковым названием на сервер, то никакой ошибка не произойдет.

```
md5sum /mnt/1GB.bin-2019_04_12_03_38_PM
0ff537fc4c4886ce307129627e325a65  /mnt/1GB.bin-2019_04_12_03_38_PM
miniocli cp /mnt/1GB.bin-2019_04_12_03_38_PM minio-test/test
/mnt/1GB.bin-2019_04_12_03_38_PM:     1.00 GiB / 1.00 GiB ┃▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓┃ 100.00% 14.59 MiB/s 1m10s


md5sum /mnt/1GB.bin-2019_04_12_03_38_PM
0b90201ef7174f1b4e654387fd2277ae  /mnt/1GB.bin-2019_04_12_03_38_PM
miniocli cp /mnt/1GB.bin-2019_04_12_03_38_PM minio-test/test
/mnt/1GB.bin-2019_04_12_03_38_PM:     1.00 GiB / 1.00 GiB ┃▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓┃ 100.00% 14.73 MiB/s 1m9s


md5sum /mnt/1GB.bin-2019_04_12_03_38_PM
8e54db472ac7f6321ae05e5f6991a588  /mnt/1GB.bin-2019_04_12_03_38_PM
miniocli cp /mnt/1GB.bin-2019_04_12_03_38_PM minio-test/test
/mnt/1GB.bin-2019_04_12_03_38_PM:     1.00 GiB / 1.00 GiB ┃▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓┃ 100.00% 14.40 MiB/s 1m11s


md5sum /mnt/1GB.bin-2019_04_12_03_38_PM
c4e11b3d8f3a51e0527cd0c2ad7369d3  /mnt/1GB.bin-2019_04_12_03_38_PM
miniocli cp /mnt/1GB.bin-2019_04_12_03_38_PM minio-test/test
/mnt/1GB.bin-2019_04_12_03_38_PM:     1.00 GiB / 1.00 GiB ┃▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓┃ 100.00% 15.10 MiB/s 1m7s
```



На каждой ноде по 2 диска. Выключаем 2 ноды из 6 нод.

```

miniocli admin info minio-test
●  dev-tools-minio-apatsev-1:9000
   Status : online
   Uptime : 47 minutes 
  Version : 2019-03-27T22:35:21Z
  Storage : Used 352 MiB
   Drives : 2/2 OK


●  dev-tools-minio-apatsev-2:9000
   Status : online
   Uptime : 47 minutes 
  Version : 2019-03-27T22:35:21Z
  Storage : Used 352 MiB
   Drives : 2/2 OK


●  dev-tools-minio-apatsev-3:9000
   Status : offline
    Error : Post http://dev-tools-minio-apatsev-3:9000/minio/peer/v1/serverinfo?: dial tcp 10.233.47.105:9000: connect: connection refused

●  dev-tools-minio-apatsev-4:9000
   Status : offline
    Error : Post http://dev-tools-minio-apatsev-4:9000/minio/peer/v1/serverinfo?: dial tcp 10.233.47.108:9000: connect: connection refused

●  dev-tools-minio-apatsev-5:9000
   Status : online
   Uptime : 47 minutes 
  Version : 2019-03-27T22:35:21Z
  Storage : Used 352 MiB
   Drives : 2/2 OK


●  dev-tools-minio-apatsev-6:9000
   Status : online
   Uptime : 47 minutes 
  Version : 2019-03-27T22:35:21Z
  Storage : Used 352 MiB
   Drives : 2/2 OK
```

Пытаемся скопировать файл. Файл копируется дольше, но копируется.

```

time miniocli cp /mnt/1GB.bin-2019_04_12_03_37_PM minio-test/test
/mnt/1GB.bin-2019_04_12_03_37_PM:     1.00 GiB / 1.00 GiB ┃▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓┃ 100.00% 41.84 MiB/s 24s
real	4m44.633s
```
