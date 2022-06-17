# Observability - мониторинг, логирование, трейсинг
## Для выполнения первого задания пондобиться следуюшее:
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
   
