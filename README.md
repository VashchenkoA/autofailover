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
