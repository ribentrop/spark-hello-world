## Ввдение

В текущей конфигурации ОТ Платформа состоит из следующих компонентов:

- Spark Cluster: Spark Master и Spark Workers
- PostgreSQL - база, которая содержит необходимую информацию (часто в моменте): есть ли схема, запускался ли диспетчер, запущен ли какой-то запрос, выполнен ли subsearch и тп
- SuperDispatcher - комплексное приложение, содержащее несколько модулей, некоторые из них: 
  - Запускает приложение в Spark, которое работает в бесконечном цикле (состоит из двух частей: системный и пользовательский)
  - принимает запросы, конвертирует их в команды Spark,и передает их в Spark
  - получает результат и сохраняет его в кэше
  - и тп.
- ot_simple_rest - принимает запросы по HTTP, распарсивает SPL (в том числе определяет есть ли subsearch), делает записи в postgres для отслеживания состояния системы
- Nifi (обычно в кластерной конфигурации) - принмает данные из внешних источников (базы данных, файлы, lookup-ы и многое другое)
- Опционально: Splunk с установленным приложением ot_simple - в текущем виде является веб-интерфейсом для запуска запросов и отрисовки дашбордов (в том числе для алертов)
- Опционально: Zeppelin - веб-интерфейс для выполнения запросов и отрисовки графиков
- Gluster FS - высокопроизводительная распределенная файловая система для хранения проиндексированных данных


## Предустановка параметров

Дальнейшее описание предусматривает, что установка ведется на операционной системе Centos 7.
Устанавливать будем в  /opt/otp
- На всякий случай обновляем пакеты:
```sh
$ yum -y update
```
1. Создаем  в /opt/otp необходимые директории для установки:

> logs 
> config
> caches
> indexes (если данные в той же директории)
> cache (если кэш индексов в той же директории)
```sh
$ mkdir /opt/otp/ /opt/otp/logs  /opt/otp/config  /opt/otp/caches  /opt/otp/indexes  /opt/otp/cache 
```
2. Из стандартного репозитория устанавливаем следующие необходимые программы:
```bash
$ yum -y install java-1.8.0-openjdk python3 postgresql-server wget git
```
3. Прописываем переменные среды:

```bash
$ echo 'export JAVA_HOME=/usr/lib/jvm/jre-1.8.0-openjdk' >> ~/.bash_profile
$ echo 'export JRE_HOME=/usr/lib/jvm/jre' >> ~/.bash_profile
$ source ~/.bash_profile
```
## Установка Spark standalone cluster
В данный момент используется Spark 2.4.3 вместе со скалой 2.11.12 поэтому есть смысл скачивать именно этот дистрибутив для установки [https://archive.apache.org/dist/spark/spark-2.4.3/spark-2.4.3-bin-hadoop2.7.tgz](https://archive.apache.org/dist/spark/spark-2.4.3/spark-2.4.3-bin-hadoop2.7.tgz)
Ссылка может измениться.
```sh
$ cd ~
$ wget https://archive.apache.org/dist/spark/spark-2.4.3/spark-2.4.3-bin-hadoop2.7.tgz
$ tar -xvzf spark-2.4.3-bin-hadoop2.7.tgz
$ sudo mv spark-2.4.4-bin-hadoop2.7 /opt/otp/spark-master
$ cd /opt/otp/spark-master/conf
$ cp spark-env.sh.template spark-env.sh
$ vi spark-env.sh
    export SPARK_MASTER_HOST=<'Spark Host IP'>
$ vi slaves
    <Spark Host name>
$ cd /opt/otp/spark-master/
$ ./sbin/start-all.sh
```
Spark поднялся:
```sh
[root@platform-1 SuperDispatcher]# netstat -an | grep '8080\|8081\|7077'
tcp6       0      0 10.128.0.50:7077        :::*                    LISTEN
tcp6       0      0 :::8080                 :::*                    LISTEN
tcp6       0      0 :::8081                 :::*                    LISTEN
[root@platform-1 SuperDispatcher]# ps axf | grep spark
14047 pts/1    S+     0:00          \_ grep --color=auto spark
10260 pts/1    Sl     0:22 /usr/lib/jvm/jre-1.8.0-openjdk/bin/java -cp /opt/otp/spark-master/conf/:/opt/otp/spark-master/jars/* -Xmx1g org.apache.spark.deploy.master.Master --host 10.128.0.50 --port 7077 --webui-port 8080
10344 ?        Sl     0:21 /usr/lib/jvm/java-1.8.0-openjdk-1.8.0.242.b08-0.el7_7.x86_64/jre/bin/java -cp /opt/otp/spark-master/conf/:/opt/otp/spark-master/jars/* -Xmx1g org.apache.spark.deploy.worker.Worker --webui-port 8081 spark://10.128.0.50:7077
```
## Установка nifi (в кластерной конфигурации еще дописывается)
1. Скачиваем дистрибутивы и распаковываем в /opt/otp (именно эту директорию, иначе придется править еще больше конфигурационных файлов)
Ссылки могут поменяться.
```bash
$ cd ~
$ wget http://apache-mirror.rbc.ru/pub/apache/nifi/1.11.1/nifi-1.11.1-bin.tar.gz
$ wget http://apache-mirror.rbc.ru/pub/apache/nifi/1.11.1/nifi-toolkit-1.11.1-bin.tar.gz
$ tar -xvf nifi-1.11.1-bin.tar.gz -C /opt/otp/
$ tar -xvf nifi-toolkit-1.11.1-bin.tar.gz -C /opt/otp/
```
2. Создаем символьные ссылки на директории.
```bash
$ ln -s /opt/otp/nifi-1.11.1/conf/ /opt/otp/config/nifi
$ mkdir /opt/otp/nifi-1.11.1/logs/ 
$ ln -s /opt/otp/nifi-1.11.1/logs/ /opt/otp/logs/nifi
```
3. Пишем start.sh для старта nifi
```bash
#!/bin/sh
DIR="$(cd "$(dirname "$0")" && pwd)"
echo $DIR
$DIR/bin/nifi.sh start
```
4. Меняем NiFi порт, т.к. 8080 уже занят Spark'ом:
```sh
$ vi /opt/otp/nifi-1.11.1/conf/nifi.properties
```
Меняем 'nifi.web.http.port=8080' на 'nifi.web.http.port=9090'
5. Запускаем и смотрим
```sh
$ /opt/otp/nifi-1.11.1/conf/start.sh
[root@platform-1 otp]# netstat -an | grep 9090
tcp        0      0 0.0.0.0:9090            0.0.0.0:*               LISTEN
[root@platform-1 otp]# ps -axf | grep nifi
15099 pts/1    S+     0:00          \_ grep --color=auto nifi
14983 pts/1    S      0:00 /bin/sh /opt/otp/nifi-1.11.1/bin/nifi.sh start
14985 pts/1    Sl     0:00  \_ /usr/lib/jvm/jre-1.8.0-openjdk/bin/java ...  org.apache.nifi.bootstrap.RunNiFi start
15002 pts/1    Sl     1:04      \_ /usr/lib/jvm/jre-1.8.0-openjdk/bin/java -classpath ... -Dorg.apache.nifi.bootstrap.config.log.dir=/opt/otp/nifi-1.11.1/logs org.apache.nifi.NiFi
```
## Установка Postgre

yum -y install java-1.8.0-openjdk python3 postgresql-server
sudo -u postgres postgresql-setup initdb
systemctl enable postgresql.service
vi /var/lib/pgsql/data/pg_hba.conf

записывать над сущесвующими настройками, а не после них
===
local   dispatcher      dispatcher                              trust
host    dispatcher      dispatcher      127.0.0.1/32            trust  
host    dispatcher      dispatcher      ::1/128                 trust  
===

systemctl start postgresql.service
sudo -u postgres psql
===
create database dispatcher;
create user dispatcher with password 'password';
grant all privileges on database dispatcher to dispatcher;
\q





5. ## Установка SuperDispatcher
```sh
$ cd $
$ git clone https://github.com/otdeveloper/SuperDispatcher
$ cp ~/SuperDispatcher/deploydir/* /opt/otp/SuperDispatcher
mkdir /opt/otp/SuperDispatcher
```
Скачиваем с гитхаба (берем дистрибы) SuperDispatcher последней версии.

Переносим все из директории репозитория deploydir в директорию /opt/otp/SuperDispatcher (остальные директории и файлы можно удалить).

Правим конфиги

`application.conf` (элементы на которые стоит обратить внимание выделены жирным )

> jdbc {
>   driver = "org.postgresql.Driver"
>   url = "jdbc:postgresql://**127.0.0.1**:5432/dispatcher"
>   username = "dispatcher"
>   password = **"password"**
> }
>
> indexes {
>   fs_disk = "file:/"
>   **path_disk = "///opt/otp/indexes/"**
>   fs_cache = "file:/"
>   **path_cache = "///mnt/g_flow/indexes/"**
>   duration_cache = 0
>   max_cols = 50
> }
>
> memcache {
>   fs = "file:/"
>   **path = "///opt/otp/caches/"**
> }
>
> lookups {
>   fs = "file:/"
>   **path = "///opt/otp/lookups/"**
> }
>
> files {
>   **log_localisation = "/opt/otp/SuperDispatcher/log_localisation.conf"**
> }

Как видно из конфига в директории  /opt/otp должны быть следующие директории:

`indexes, caches, lookups`

`path_cache` по идее тоже нужно указать на директорию /opt/otp/cache (но это не точно)

Создаем start.sh для диспетчера (например, копируя пример из файла `deploy_example.bash`, надеясь, что он будет актуальным) и правим его (основное выделено жирным, правится в зависмости от среды, а заодно не помешает проверить версии jar-файлов)

> !/bin/sh
>
> **/opt/otp/spark-master/bin/spark-submit** \
> --verbose \
> --master spark://**otp1**:7077 \
> --deploy-mode cluster \
> --supervise \
> --driver-cores **2** \
> --driver-memory **4G** \
> --executor-cores **1** \
> --executor-memory **2G** \
> --conf "spark.application.config=**/opt/otp**/SuperDispatcher/application.conf" \
> --conf "spark.blacklist.enable=true" \
> --conf "spark.driver.maxResultSize=**2G**" \
> --conf "spark.dynamicAllocation.enabled=false" \
> --conf "spark.locality.wait=0" \
> --conf "spark.scheduler.allocation.file=**/opt/otp**/SuperDispatcher/fairscheduler.xml" \
> --conf "spark.scheduler.mode=FAIR" \
> --conf "spark.serializer=org.apache.spark.serializer.KryoSerializer" \
> --conf "spark.shuffle.service.enabled=false" \
> --conf "spark.speculation=true" \
> --conf "spark.sql.caseSensitive=true" \
> --conf "spark.sql.crossJoin.enabled=true" \
> --conf "spark.sql.files.ignoreCorruptFiles=true" \
> --conf "spark.sql.files.ignoreMissingFiles=true" \
> --jars **config-1.3.4.jar,postgresql-42.2.5.jar,json4s-native_2.11-3.5.5.jar,json4s-ast_2.11-3.5.5.jar,evaluator_2.11-1.1.0.jar** \
> --class SuperDriver **dispatcher_2.11-0.18.0.jar**

Не забываем 

```bash
$ chmod +x start.sh
```

При запуске можно наткнуться на такую ошибку, которая как раз говорит о несовпадении файлов в директории и в submit:

> 20/01/21 16:53:54 INFO ClientEndpoint: State of driver-20200121165349-0003 is ERROR
> 20/01/21 16:53:54 **ERROR ClientEndpoint: Exception from cluster was:** java.nio.file.NoSuchFileException: /opt/otp/SuperDispatcher/**dispatcher_2.11-0.17.3.jar**
> java.nio.file.NoSuchFileException: /opt/otp/SuperDispatcher/dispatcher_2.11-0.17.3.jar
>         at sun.nio.fs.UnixException.translateToIOException(UnixException.java:86)
>         at sun.nio.fs.UnixException.rethrowAsIOException(UnixException.java:102)
>         at sun.nio.fs.UnixException.rethrowAsIOException(UnixException.java:107)
>         at sun.nio.fs.UnixCopyFile.copy(UnixCopyFile.java:526)
>         at sun.nio.fs.UnixFileSystemProvider.copy(UnixFileSystemProvider.java:253)
>         at java.nio.file.Files.copy(Files.java:1274)
>         at org.apache.spark.util.Utils$.org$apache$spark$util$Utils$$copyRecursive(Utils.scala:664)
>         at org.apache.spark.util.Utils$.copyFile(Utils.scala:635)
>         at org.apache.spark.util.Utils$.doFetchFile(Utils.scala:719)
>         at org.apache.spark.util.Utils$.fetchFile(Utils.scala:509)
>         at org.apache.spark.deploy.worker.DriverRunner.downloadUserJar(DriverRunner.scala:155)
>         at org.apache.spark.deploy.worker.DriverRunner.prepareAndRunDriver(DriverRunner.scala:173)
>         at org.apache.spark.deploy.worker.DriverRunner$$anon$1.run(DriverRunner.scala:92)
> 20/01/21 16:53:54 INFO ShutdownHookManager: Shutdown hook called
> 20/01/21 16:53:54 INFO ShutdownHookManager: Deleting directory /tmp/spark-b31c4f11-a519-4a80-83cd-0de74efa0e66
> [root@otp1 otp]# ls -l /opt/otp/SuperDispatcher/dispatcher_2.11-0.17.3.jar
> ls: cannot access /opt/otp/SuperDispatcher/dispatcher_2.11-0.17.3.jar: **No such file or directory**
> [root@otp1 otp]# ls -l /opt/otp/SuperDispatcher
> total 2724
> -rw-r--r-- 1 root root     518 Jan 21 14:05 application.conf
> -rw-r--r-- 1 root root  292739 Jan 21 13:55 config-1.3.4.jar
> -rw-r--r-- 1 root root    1100 Jan 21 13:55 deploy_example.bash
> -rw-r--r-- 1 root root 1204445 Jan 21 13:55 **dispatcher_2.11-0.18.0.jar**
> -rw-r--r-- 1 root root  239425 Jan 21 13:55 evaluator_2.11-1.1.0.jar
> -rw-r--r-- 1 root root    1135 Jan 21 13:55 fairscheduler.xml
> -rw-r--r-- 1 root root   90667 Jan 21 13:55 json4s-ast_2.11-3.5.5.jar
> -rw-r--r-- 1 root root  100084 Jan 21 13:55 json4s-native_2.11-3.5.5.jar
> -rw-r--r-- 1 root root     834 Jan 21 13:55 log_localisation.conf
> -rw-r--r-- 1 root root  825943 Jan 21 13:55 postgresql-42.2.5.jar

Как видим версии jar-файла диспетчера не совпадают.

После запуска start.sh SuperDispatcher увидим, что в Spark master web-консоли появилось запущенное приложение:

![]()

Это значит SuperDispatcher запустился.

6. ## Установка ot_simple_rest

Скачиваем с гитхаба (берем дистрибы): 

- ot_simple
- ot_simple_rest

На ot_simple будет ссылка из splunk/etc/app/

??Requirements для ot_simple_rest: python3, postgresql-devel?? (но похоже, что уже все встроено)

Настройка ot_simple_rest.

Конфиг `ot_simple_rest.conf`

> [general]
> level = DEBUG
>
> [db_conf]
> host = **127.0.0.1**
> database = dispatcher
> user = dispatcher
> password = **password** 
>
> [mem_conf]
> path = **/opt/otp/caches** 
>
> [dispatcher]
> tracker_max_interval = 60
>
> [resolver]
> no_subsearch_commands = foreach,appendpipe

**[root@otp1 ot_simple_rest]# vi start.sh**

```bash
#!/bin/bash

cd logs
source /opt/otp/ot_simple_rest/venv/bin/activate && /opt/otp/ot_simple_rest/venv/bin/python3 /opt/otp/ot_simple_rest/ot_simple_rest.py > stdout.log 2> stderr.log &
```

**[root@otp1 ot_simple_rest]# vi stop.sh**

``` bash
#!/bin/bash

kill `ps ax | grep "/opt/otp/ot_simple_rest/venv/bin/python3 /opt/otp/ot_simple_rest/ot_simple_rest.py" | grep -v grep | awk '{print $1}'`
```



7. ## Установка zeppelin(опционально)

```bash
$ wget http://apache-mirror.rbc.ru/pub/apache/zeppelin/zeppelin-0.8.2/zeppelin-0.8.2-bin-all.tgz
```



8. ## Установка splunk(опционально)

В нашем примере установка Splunk обязательна, так как нам нужно будет через ot_simple с помощью Splunk получить доступ к данным, которые находятся или приходят в /opt/otp/indexes

Ставим Splunk стандартным способом.

Для простоты создаем символьную ссылку splunk/etc/apps/ot_simple -> /opt/otp/ot_simple

Естественно, если Splunk будет стоять на другой машине, нужно будет просто установить на него приложение ot_simple и настроить через web-консоль.

После настройки приложения в web-консоли splunk может выскочить ошибка:

> Encountered the following error while trying to update: Error while posting to url=/servicesNS/nobody/ot_simple/otsimple/config/general
>

, типа не удалось настроить, но настройка на самом деле прошла (это видно из логов ot_simple_rest) - создалась схема и ролевая модель. 

![]()

Правда запустить ot_simple все равно не получится, придется вручную в настройках приложения создать и вписать в `app.conf`,  что приложение сконфигурировано:

> ```bash
> $  cat ot_simple/local/app.conf
> ```
>
> [install]
> **is_configured = 1**

Я пока незнаю, как это исправить, чтобы сразу настраивалось (раньше получалось сразу), и возможно ли.

9. ## Настройка индексов и установка gluster 

10. ## Установка lustre
