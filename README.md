# Observability - мониторинг, логирование, трейсинг
## Для выполнения первого задания требуется выполнить следуюшие действия:
1) Устанавливаем на Вашу VM OS CentOS Linux v 7
2) Обновляем
    ```
    sudo yum update
4) Утсанавливаем  дополнительные пакеты, которые помогут нам в будушем 
    ```
    yum install mc 
    yum install htop 
    yum install epel-release 
    yum install telnet
    yum install wget
    yum install vim
4) Устанавливаем CMS wordpress по официальной инструкции [Как установить Wordpress](https://wordpress.org/support/article/how-to-install-wordpress/)
5) Устанавливаем nginx
    ```
    yum install nginx
6) Поризводим конфигурацию nginx [nginx.conf](https://github.com/defmeg/otus/blob/main/GAP-1/nginx.conf)
7) Устанавливаем Prometheus и Node Exporter
   
   Создаем для них пользователей
   ```
    useradd --no-create-home --shell /bin/false prometheus
    useradd --no-create-home --shell /bin/false node_exporter
   ```
   Создаем каталоги для системных файлов
   ```
    mkdir /etc/prometheus
    mkdir /var/lib/prometheus
   ```
   Применям разрешаем созданным пользователям на каталоги
   ```
    chown prometheus:prometheus /etc/prometheus
    chown prometheus:prometheus /var/lib/prometheus
   ```
   Устанавливаем Prometheus
   ```
    wget https://github.com/prometheus/prometheus/releases/download/v2.36.1/prometheus-2.36.1.linux-amd64.tar.gz
    tar -xf prometheus-2.36.1.linux-amd64.tar.gz
    cp prometheus-2.36.1.linux-amd64/prometheus /usr/local/bin/
    cp prometheus-2.36.1.linux-amd64/promtool /usr/local/bin/
    chown prometheus:prometheus /usr/local/bin/prometheus
    chown prometheus:prometheus /usr/local/bin/promtool
    cp -r prometheus-2.36.1.linux-amd64/consoles /etc/prometheus/
    cp -r prometheus-2.36.1.linux-amd64/console_libraries /etc/prometheus/
    chown -R prometheus:prometheus /etc/prometheus/consoles
    chown -R prometheus:prometheus /etc/prometheus/console_libraries
    rm -rf prometheus-2.36.1.linux-amd64*
   ```
   Насторим Prometheus, пример конфигурации [prometheus.yml](https://github.com/defmeg/otus/blob/main/GAP-1/prometheus.yml)
   ```
    vim /etc/prometheus/prometheus.yml
   ```
   Применяем разрегения к файлу конфигурации Prometheus
   ```
    chown prometheus:prometheus /etc/prometheus/prometheus.yml
   ```
   Создаем демон для управления сервисом Prometheus
   ```
    vim /etc/systemd/system/prometheus.service
    
    [Unit]
    Description=Prometheus
    Wants=network-online.target
    After=network-online.target

    [Service]
    User=prometheus
    Group=prometheus
    Type=simple
    ExecStart=/usr/local/bin/prometheus \
        --config.file /etc/prometheus/prometheus.yml \
        --storage.tsdb.path /var/lib/prometheus/ \
        --web.console.templates=/etc/prometheus/consoles \
        --web.console.libraries=/etc/prometheus/console_libraries

    [Install]
    WantedBy=multi-user.target
   ```
   Применяяем конфигурацию демона и стартуем Prometheus
   ```
    systemctl daemon-reload
    systemctl enable prometheus
    systemctl start prometheus  
   ```
   Устанавливаем Node Exporter
   ```
    wget https://github.com/prometheus/node_exporter/releases/download/v0.17.0/node_exporter-1.3.1.linux-amd64.tar.gz
    tar -xf node_exporter-1.3.1.linux-amd64.tar.gz
    cp node_exporter-1.3.1.linux-amd64/node_exporter /usr/local/bin
    chown node_exporter:node_exporter /usr/local/bin/node_exporter
    rm -rf node_exporter-1.3.1.linux-amd64*
   ```
   Создаем демон для управления сервисом Node Exporter
   ```
    vim /etc/systemd/system/node_exporter.service
    
    [Unit]
    Description=Node Exporter
    Wants=network-online.target
    After=network-online.target

    [Service]
    User=node_exporter
    Group=node_exporter
    Type=simple
    ExecStart=/usr/local/bin/node_exporter

    [Install]
    WantedBy=multi-user.target
   ```
   Применяяем конфигурацию демона и стартуем Node Exporter
   ```
    systemctl daemon-reload
    systemctl enable node_exporter
    systemctl start node_exporter
   ```
   Добавляем в файл конфигурации Prometheus [prometheus.yml](https://github.com/defmeg/otus/blob/main/GAP-1/prometheus.yml) job Node Exporter
   Перезапускаем Prometheus
   ```
    systemctl restart prometheus
   ```
   Добавляем в nignx.conf Prometheus, пример [nginx.conf](https://github.com/defmeg/otus/blob/main/GAP-1/nginx.conf)
   Открываем  Prometheus http://prometeus.wordpressvm.io/targets и проверям добавление Node Exporter
   ![image](https://user-images.githubusercontent.com/104725435/174321989-71ec18c2-16f2-4deb-815d-d61d3a999713.png)

8) Устанавливаем Alertmanager \
   Установлка Alertmanager происходит аналогично Prometheus и Node Exporter
   ссылка на [Alertmanager](https://github.com/prometheus/alertmanager/releases/download/v0.24.0/alertmanager-0.24.0.linux-amd64.tar.gz) \
   Пример конфигурации Alertmanager с отправкой алертов в Telegram_bot [alertmanager.yml](https://github.com/defmeg/otus/blob/main/GAP-1/alertmanager.yml)
   
9) Установка Blackbox Exporter \
   Установлка Blackbox Exporter происходит аналогично Prometheus и Node Exporter
   ссылка на [Blackbox Exporter](https://github.com/prometheus/blackbox_exporter/releases/download/v0.21.0/blackbox_exporter-0.21.0.linux-amd64.tar.gz)
   Пример конфигурации Blackbox Exporter [blackbox.yml](https://github.com/defmeg/otus/blob/main/GAP-1/blackbox.yml)
   Далее длобавть Blackbox Exporter в конфигурацию [prometheus.yml](https://github.com/defmeg/otus/blob/main/GAP-1/prometheus.yml)
 
10) Нужно сконфигурировать алерты
    ```
    vim /etc/prometheus/alert.rules.yml
    ```
    Пример файла конфигурации [alert.rules.yml](https://github.com/defmeg/otus/blob/main/GAP-1/alert.rules.yml) \
    После перезапуска Prometheus заходим в Prometheus status/rules \
    ![image](https://user-images.githubusercontent.com/104725435/174665634-ad297054-b4a1-4418-a04b-284f4bfcf851.png)
11) Устанавливам в docker [prometheus_bot](https://github.com/inCaller/prometheus_bot)

## Для выполнения второго задания требуется выполнить следуюшие действия:
1) Устанавливаем Grafana
   Создаем файл конфигурации репозитория для графаны:
   ```
    vim /etc/yum.repos.d/grafana.repo
    ...
    [grafana]
    name=grafana
    baseurl=https://packages.grafana.com/oss/rpm
    repo_gpgcheck=1
    enabled=1
    gpgcheck=1
    gpgkey=https://packages.grafana.com/gpg.key
    sslverify=1
    sslcacert=/etc/pki/tls/certs/ca-bundle.crt
   ```
   Устанавливаем:
   ```
    yum install grafana
   ```
   Зпускаем сервис grafana-server
   ```
    systemctl enable grafana-server
    systemctl start grafana-server
   ```
   Добавляем Data Sourse Prometheus \
   И создаем следующие дашбоарды \
   1 - [Сводная информация по инфраструктуре (CPU, RAM, Network, etc.)](https://github.com/defmeg/otus/blob/main/GAP-2/Grafana_dashboard_nod_exporter.png) \
   2 - [Cводная информация о CMS (доступность компонентов, время ответа, etc.)](https://github.com/defmeg/otus/blob/main/GAP-2/Grafana_dashboard_CMS_nginx.png)
   
## Для выполнения третьего и четвертого задания требуется выполнить следуюшие действия:
1) Скачать дитрибутивы hearthbeat, filebeat и metricbeat с официального сайта: [Страница загрузки](https://www.elastic.co/downloads/beats/)
   скачиваем версию rpm x86_64
2) Устанавливаем скаченные пакеты:
   ```
    yum install filebeat-8.2.2-x86_64.rpm
    yum install metricbeat-8.2.2-x86_64.rpm
    yum install heartbeat-8.2.2-x86_64.rpm
   ```
   Файлы конфигураий beats: \
    [filebeat.yml](https://github.com/defmeg/otus/blob/main/GAP-3/filebeat.yml) \
    [heartbeat.yml](https://github.com/defmeg/otus/blob/main/GAP-3/heartbeat.yml) \
    [metricbeat.yml](https://github.com/defmeg/otus/blob/main/GAP-3/metricbeat.yml) \
3) Скачать дистрибутивы logstash, elasticsearch и kibana с официального сайта: \
   так же скачиваем версию rpm x86_64 \
   [logstash](https://www.elastic.co/downloads/logstash) \
   [elasticsearch](https://www.elastic.co/downloads/elasticsearch) \
   [kibana](https://www.elastic.co/downloads/kibana)
    
4) Устанавливаем скаченные пакеты:
   ```
    yum install logstash-8.2.2-x86_64.rpm
    yum install elasticsearch-8.2.2-x86_64.rpm
    yum install kibana-8.2.2-x86_64.rpm
   ```
   По заданию делаем отправку логов через logstash \
   Файлы конфигураий logstash расположенных в /etc/logstash/conf.d : \
    [logstash.conf](https://github.com/defmeg/otus/blob/main/GAP-4/logstash.conf) \
    [filter.conf](https://github.com/defmeg/otus/blob/main/GAP-4/filter.conf)
    
   
    
