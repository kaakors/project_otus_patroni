#### 1) устанавливаем пакет vip-manager ####

>   rpm -ivh vip-manager-1.0.2-1.x86_64.rpm
	
#### 2) заполняем конфиг файл /etc/default/vip-manager.yml #### 
```
interval: 1000
trigger-key: "/db/postgres/leader"
trigger-value: "host_balansir"
ip: ip_balansir
netmask: 26
interface: eth0
hosting-type: basic
dcs-type: etcd
dcs-endpoints:
  - http://127.0.0.1:port
  - http://ip1:port
  - http://ip2:port
  - http://ip3:port
etcd-user: "user"
etcd-password: "x"
retry-num: 2
retry-after: 250
verbose: false
```
#### 3) проверяем корректность настроек службы /usr/lib/systemd/system/vip-manager.service и запускаем службу ####

>   systemctl start vip-manager
