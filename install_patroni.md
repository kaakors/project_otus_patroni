#### 1) настройка репозитория pip ####


>   vi /root/.pypirc
```
[distutils]
index-servers = local
[local]
repository: https://

	cd /root/
	mkdir .pip
	vi /root/.pip/pip.conf

[global]
index-url = https://
[search]
index = https://
```
#### 2) установка patroni и зависимостей ####
```
	sudo yum install -y python3
	sudo python3 -m pip install --upgrade pip
	python3 --version
	sudo yum install -y gcc python3-devel
	sudo python3 -m pip install psycopg2-binary
	sudo python3 -m pip install patroni[etcd]
	patroni --version
```	

#### 3) создаем каталог настроек ####
```
	sudo mkdir /etc/patroni
	sudo chown postgres:postgres /etc/patroni
	sudo chmod 700 /etc/patroni

```

#### 4) создаем	файл /etc/patroni/patroni.yml : ####
	
name: host
namespace: /db/
scope: postgres
restapi:
    listen: 0.0.0.0:8008
    connect_address: ip:port
    authentication:
        username: x
        password: x
etcd:
    hosts: host1:port1, host2:port2, host3:port3
    username: x
    password: x
bootstrap:
    dcs:
        ttl: 30
        loop_wait: 10
        retry_timeout: 10
        maximum_lag_on_failover: 1048576
        master_start_timeout: 300
        postgresql:
            use_pg_rewind: true
            use_slots: true
            parameters:
                wal_level: replica
                hot_standby: "on"
                wal_keep_segments: 8
                max_wal_senders: 5
                max_replication_slots: 5
                checkpoint_timeout: 30
                max_connections: 200
                shared_buffers: 1024MB
            initdb:
              - auth-host: md5
              - auth-local: peer
              - encoding: UTF8
              - data-checksums
              - locale: ru_RU.UTF-8
            pg_hba:
              - local   all             all                                     md5
              - host    all             all             127.0.0.1/32            md5
              - host    all             all             ::1/128                 md5
              - local   replication     all                                     md5
              - host    replication     all             127.0.0.1/32            md5
              - host    replication     all             ::1/128                 md5
              - host    all             all             all                     md5
			        - host    replication     replicator      127.0.0.1/32            md5
              - host    replication     replicator      10.230.84.67/0          md5
              - host    replication     replicator      10.230.84.70/0          md5
              - host    replication     replicator      10.230.84.73/0          md5
            users:
                admin:
                    password: x
                options:
                  - superuser
postgresql:
    listen: 0.0.0.0:5432
    connect_address: ip:port
    config_dir: /var/lib/pgsql/12/data
    bin_dir: /usr/pgsql-12/bin/
    data_dir: /var/lib/pgsql/12/data
    pgpass: 
    authentication:
        superuser:
            username: x
            password: x
        replication:
            username: x
            password: x
        rewind:
            username: x
            password: x
    parameters:
        unix_socket_directories: '/var/run/postgresql/'
tags:
    nofailover: false
    noloadbalance: false
    clonefrom: false
    nosync: false
```
	
#### 5) Создаём unit файл /etc/systemd/system/patroni.service

```
[Unit]
Description=Runners to orchestrate a high-availability PostgreSQL
After=syslog.target network.target
[Service]
Type=simple
User=postgres
Group=postgres
ExecStart=/usr/local/bin/patroni /etc/patroni/patroni.yml
ExecReload=/bin/kill -s HUP $MAINPID
KillMode=process
TimeoutSec=30
Restart=no
[Install]
WantedBy=multi-user.target
```

###### Перезагружаем и запускаем ######

>   sudo systemctl daemon-reload
>   sudo systemctl start patroni.service


#### 6) Далее проверяем успешность настройки ####

- проверяем папку, пусть к которой указали в "data_dir", должна создаться структура папок

- должен быть слушаетель порта :
	
>   ss -ltn | grep port

- пробуем подключиться к psql:

>   psql -U postgres -d postgres

если все корректно добавляем в автозагрузку

>   sudo systemctl enable patroni.service
