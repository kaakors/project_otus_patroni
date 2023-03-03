1) Копируем на хосты архив с etcd (разворачивается на всех хостах стенда)
2) разархивировать

>   sudo tar -xvf etcd-v3.3.10-linux-amd64.tar.gz -C /usr/bin/ --strip-components=1

3) на первом хосте заполняем файл create-config.sh, комментируем строки переменных для других хостов, оставляем только адрес локального хоста

4) Делаем файл исполняемым и формируем файл для запуска сервиса etcd

>   sudo chmod +x create-config.sh
>   ./create-config etcd

5) запускаем демон etcd

>   sudo systemctl start etcd
>   sudo systemctl status etcd

для проверки кластера можно выполнить:

>   etcdctl cluster-health

если статус демона running добавляем его в автозапуск

>   sudo systemctl enable etcd

6) Добавление нового узла etcd происходит в 2 этапа:

объявляем о добавлении нового узла на действующем узле:

>   etcdctl member add host http://IP:port
	
после этого устанавливаем и запускаем etcd на объявленном узле

в настройках будет первый хост + новый добавленный. При следующем добавлении будет уже 3 и т.д.
После добавления всего кластера необходимо привести файл запуска службы к единому виду

7) добавить пользователя и включить авторизацию по логину

>   etcdctl user add root
>   etcdctl user get root
>   etcdctl auth enable
>   etcdctl user get root
>   etcdctl --username root user get root
