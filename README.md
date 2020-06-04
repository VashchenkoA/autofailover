# autofailover
Тестовый стенд состоит из трёх виртуальных машин Ubuntu 16.04:

Arbiter – 192.168.145.134
![Image alt](https://github.com/VashchenkoA/autofailover/raw/master/images/1.png)

Slave – 192.168.145.133
![Image alt](https://github.com/VashchenkoA/autofailover/raw/master/images/2.png)

Master – 192.168.145.132
![Image alt](https://github.com/VashchenkoA/autofailover/raw/master/images/3.png)

Настраивать autofailover будем при помощи pgpool.

На master и slave устанавливаем:

sudo apt install postgresql-9.5

sudo -u postgres createuser ngw_admin -P -e

sudo apt install postgresql-9.5-postgis-2.2

sudo apt install postgresql-9.5-pgpool2

На arbiter:
sudo apt install pgpool2

Настраиваем потоковую репликацию:
на мастере:
sudo nano /etc/postgresql/9.5/main/postgresql.conf

listen_addresses = '192.168.145.132'

wal_level = hot_standby

max_wal_senders = 2

wal_keep_segments = 32

#hot_standby = on

настраиваем для своих адресов в сети .145.0

sudo nano /etc/postgresql/9.5/main/pg_hba.conf

![Image alt](https://github.com/VashchenkoA/autofailover/raw/master/images/4.png)

sudo service postgresql restart

на слейве:

sudo service postgresql stop

sudo nano /etc/postgresql/9.5/main/postgresql.conf

listen_addresses = '192.168.145.133'

hot_standby = on

sudo nano /etc/postgresql/9.5/main/pg_hba.conf
![Image alt](https://github.com/VashchenkoA/autofailover/raw/master/images/5.png)
![Image alt](https://github.com/VashchenkoA/autofailover/raw/master/images/6.png)

Для передачи бэкапа установил shh, добавил в sshd_config:

PermitRootLogin yes

PasswordAuthentication yes

UseLogin yes

После перезапустил сервис sshd и передал бэкап

![Image alt](https://github.com/VashchenkoA/autofailover/raw/master/images/7.png)
![Image alt](https://github.com/VashchenkoA/autofailover/raw/master/images/8.png)

Создаем конфигурационный файл реплики на слейве:

![Image alt](https://github.com/VashchenkoA/autofailover/raw/master/images/9.png)

Раздаем права:
![Image alt](https://github.com/VashchenkoA/autofailover/raw/master/images/10.png)

И стартуем:

sudo service postgresql start

Проверяем репликацию:
![Image alt](https://github.com/VashchenkoA/autofailover/raw/master/images/11.png)
![Image alt](https://github.com/VashchenkoA/autofailover/raw/master/images/12.png)

Создал на мастере базу:
![Image alt](https://github.com/VashchenkoA/autofailover/raw/master/images/13.png)

Заходим на слэйва:
![Image alt](https://github.com/VashchenkoA/autofailover/raw/master/images/14.png)

Приступаем к настройке арбитра:

изменим конфигурационный файл /etc/pgpool2/pgpool.conf:

sudo nano /etc/pgpool2/pgpool.conf

# Устанавливаем весь диапазон прослушиваемых адресов

listen_addresses = '192.168.145.134'

# Параметры подключения к базе на сервере pgmaster

backend_hostname0 = '192.168.145.132'

backend_port0 = 5432

backend_weight0 = 1

backend_data_directory0 = '/var/lib/postgresql/9.5/main'

# Параметры подключения к базе на сервере pgslave

backend_hostname1 = '192.168.145.133'

backend_port1 = 5432

backend_weight1 = 1

backend_data_directory1 = '/var/lib/postgresql/9.5/main'

# Используем pool_hba.conf для авторизации клиентов

enable_pool_hba = true

sr_check_user = 'postgres'

health_check_user = 'postgres'

memory_cache_enabled = on

memqcache_oiddir = '/var/log/postgresql/oiddir'

Изменим конфигурационный файл /etc/pgpool2/pool_hba.conf:
![Image alt](https://github.com/VashchenkoA/autofailover/raw/master/images/16.png)

Добавим пароли в файл /etc/pgpool2/pool_passwd:
![Image alt](https://github.com/VashchenkoA/autofailover/raw/master/images/17.png)

Раздаем права и перезапускаемся:
![Image alt](https://github.com/VashchenkoA/autofailover/raw/master/images/18.png)

Проверим работоспособность pgpool:
![Image alt](https://github.com/VashchenkoA/autofailover/raw/master/images/19.png)

Везде статус 2, значит все работает.
Настраиваем автофейловер:
задаем пароль юзеру постгреса

sudo passwd postgres

Генерируем ключи (для фейловера необходимо ssh соединение без пароля):
![Image alt](https://github.com/VashchenkoA/autofailover/raw/master/images/20.png)

Перебрасываем ключи:
![Image alt](https://github.com/VashchenkoA/autofailover/raw/master/images/21.png)

Добавим в конфигурационный файл /etc/pgpool2/pgpool.conf следующую строчку:
![Image alt](https://github.com/VashchenkoA/autofailover/raw/master/images/22.png)

Файл для логов:

![Image alt](https://github.com/VashchenkoA/autofailover/raw/master/images/23.png)

Создадим скрипт /etc/pgpool2/failover.sh:
скрипт из туторов не отработал, попросил помощи у другой бригады
![Image alt](https://github.com/VashchenkoA/autofailover/raw/master/images/24.png)

755 права на скрипт.

Идем на мастера, вырубаем его и смотрим со слейва что получилось:

![Image alt](https://github.com/VashchenkoA/autofailover/raw/master/images/25.png)

Мастер с адресом 132 валяется, и теперь слейв стал мастером. Успех.
