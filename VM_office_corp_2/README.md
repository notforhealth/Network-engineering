# Создание виртуальной сети офиса компании
- В данном проекте была создана сеть офиса компании с использованием firewall, серверов и клиента.

## Содержание
- [Используемые средства](#используемые-средства)
- [Логическая схема](#логическая-схема)
- [Импортирование всех виртуальных машин](#импортирование-всех-виртуальных-машин)
- [Настройка сетевых адаптеров](#настройка-сетевых-адаптеров)
- [Установка ОС](#установка-ос)
- [Настройка pfSense](#настройка-pfsense)
- [Настройка DNS сервера](#настройка-dns-сервера)
- [Настройка DHCP сервера](#настройка-dhcp-сервера)
- [Настройка МЭ](#настройка-мэ)
- [Настройка NTP сервера](#настройка-ntp-сервера)
- [Настройка Веб сервера](#настройка-веб-сервера)
- [Система мониторинга](#система-мониторинга)
- [Система резервного копирования](#система-резервного-копирования)
- [Тестирование](#тестирование)
- [Итоги](#итоги)

## Используемые средства
- Персональный компьютер
- Операционная система [pfSense](https://www.pfsense.org/download/)
- Операционная система [Ubuntu Desktop 24.04.3 LTS](https://ubuntu.com/download/desktop)
- Операционная система [Ubuntu Server 24.04.3 LTS](https://ubuntu.com/download/server)
- [Oracle VirtualBox](https://www.virtualbox.org)


## Логическая схема
![Логическая схема](https://github.com/notforhealth/Network-engineering/blob/main/VM_office_corp_2/images/VM_logical.png)


## Импортирование всех виртуальных машин
### VirtualBox
- Для установки VB необходимо перейти по ссылке https://www.virtualbox.org и произвести базовую установку приложения.
### pfSense
- Для установки pfSense переходим по ссылке https://www.pfsense.org/download/ и с использованием прокси сервера скачать установочный файл (т.к. сайт pfsense недоступен в России). Далее необходимо импортировать iso файл в VirtualBox. Нажимаем создать вписываем название, выбираем образ iso, выбираем необходимое количество оперативной памяти и число ядер процессора.
### Ubuntu
- Для установки Ubuntu скачиваем образ Ubuntu 24.04.3 с сайтов https://ubuntu.com/download/server и https://ubuntu.com/download/desktop. Здесь необходимо создать три сервера и один клиент.
- В итоге получаем 5 виртуальных машин в VirtualBox:

- ![Виртуальные машины](https://github.com/notforhealth/Network-engineering/blob/main/VM_office_corp_2/images/VM.png)

## Настройка сетевых адаптеров
### Для pfSense
- Необходимо добавить три сетевых адаптера с внутренней сетью:
![Внутренняя сеть](https://github.com/notforhealth/Network-engineering/blob/main/VM_office_corp_2/images/firewall_adapter1.png)
![Внутренняя сеть](https://github.com/notforhealth/Network-engineering/blob/main/VM_office_corp_2/images/firewall_adapter_2.png)
![Внутренняя сеть](https://github.com/notforhealth/Network-engineering/blob/main/VM_office_corp_2/images/firewall_adapter_3.png)
### Для серверов и клиента
- Необходимо переключить с NAT на внутреннюю сеть. Для каждого из виртуальных машин представленны изображения:
![domaincontroller](https://github.com/notforhealth/Network-engineering/blob/main/VM_office_corp_2/images/domain_adapter.png)
![web](https://github.com/notforhealth/Network-engineering/blob/main/VM_office_corp_2/images/web_adapter.png)
![monitor](https://github.com/notforhealth/Network-engineering/blob/main/VM_office_corp_2/images/monitor_adapter.png)
![client](https://github.com/notforhealth/Network-engineering/blob/main/VM_office_corp_2/images/client_adapter.png)

## Установка ОС
### Установка pfSense
- Установка pfSense практически ничем не отличается от установки OPNsense описаной в предыдущем проекте.
### Установка Ubuntu Server
- Язык, раскладку оставляем по умолчанию. Тип установки Ubuntu Server. Сеть оставляем DHCP, прокси пропускаем. Устанавливаем на доступный диск. В профиле вводим следующие данные:
![profile](https://github.com/notforhealth/Network-engineering/blob/main/VM_office_corp_2/images/server_download.png)
- Также устанавливаем SSH, все остальное оставляем по умолчанию.
- Такую же процедуру повторить на двух оставшихся серверах.
### Установка Ubuntu Desktop
- Все оставляем по умолчанию. В этом окне заполняем профиль:
![profile](https://github.com/notforhealth/Network-engineering/blob/main/VM_office_corp_2/images/client_download.png)


## Настройка pfSense
- После установки необходимо изменить названия для серверов, то есть OPT1 меняем на SERVERS и OPT2 меняем на DMZ.
- Также необходимо настроить для всех LAN интерфейсов IP адреса. 192.168.10.1 - для LAN (клиентская подсеть), 192.168.20.1 - для серверов DC и MS, 192.168.30.1 - для WS.


## Настройка DNS сервера
### Установка и настройка на domaincontroller
- В качестве DNS сервера будет использоваться BIND9 - самое популярное ПО для создания DNS сервера.
- Сначала обновим систему и установим bind9
``````
sudo apt update && sudo apt upgrade -y
sudo apt install bind9 bind9utils bind9-doc -y
``````
- Открываем основной конфиг:
``````
sudo nano /etc/bind/named.conf.options
``````
- Добавляем:
``````
options {
    directory "/var/cache/bind";
    
    allow-query { 
        localhost; 
        192.168.10.0/24;
        192.168.20.0/24;
        192.168.30.0/24;
    };
    
    allow-recursion {
        localhost;
        192.168.10.0/24;
        192.168.20.0/24;
        192.168.30.0/24;
    };
    
    forwarders {
        8.8.8.8;
        1.1.1.1;
    };
    
    dnssec-validation auto;
    listen-on { any; };
    listen-on-v6 { any; };
};
``````

- Рекурсивные запросы нужны в том случае, если при запросе к DNS серверу, он не находит у себя нужной записи, он идет к другим серверам и спрашивает у них. Нерекурсивный DNS сервер в данном случае просто говорит - "я не знаю, но спроси у этого сервера". Это позволяет использовать DNS запросы быстрее.
- Форвадеры - те самые DNS серверы у которых рекурсивный DNS сервер делает запрос.

### Создание доменной зоны
- Доменная зона - база данных, которая содержит информацию о доменных именах и их IP-адресах
- Для начала откроем файл зоны и изменим его:
``````
sudo nano /etc/bind/named.conf.local
``````

``````
// прямая зона - преобразует имена в IP
zone "office.corp" {
    type master;
    file "/etc/bind/db.office.corp";
};

// обратная зона - преобразует IP в имена
zone "10.168.192.in-addr.arpa" {
    type master;
    file "/etc/bind/db.192.168.10";
};

zone "20.168.192.in-addr.arpa" {
    type master;
    file "/etc/bind/db.192.168.20";
};

zone "30.168.192.in-addr.arpa" {
    type master;
    file "/etc/bind/db.192.168.30";
};
``````
### Создание файлов зон
- Создаем прямую зону:
``````
sudo nano /etc/bind/db.office.corp
``````
- Изменяем содержимое:
``````
$TTL    604800
@       IN      SOA     domaincontroller.office.corp. administrator.office.corp. (
                              2024022001 ; Serial number
                              604800     ; Refresh
                              86400      ; Retry
                              2419200    ; Expire
                              604800 )   ; Negative Cache TTL

; DNS Servers
@       IN      NS      domaincontroller.office.corp.

; A Records
dc01            IN      A       192.168.20.10
web01           IN      A       192.168.30.10
mon01           IN      A       192.168.20.100
fw01            IN      A       192.168.10.1

; CNAME Records
www             IN      CNAME   webserver.office.corp.
ftp             IN      CNAME   webserver.office.corp.
``````

- Создаем обратную зону для подсети LAN:
``````
sudo nano /etc/bind/db.192.168.10
``````
``````
$TTL    604800
@       IN      SOA     domaincontroller.office.corp. administrator.office.corp. (
                              2024022001 ;
                              604800     ;
                              86400      ;
                              2419200    ;
                              604800 )   ;

@       IN      NS      domaincontroller.office.corp.

; PTR Records - преобразуют IP в имена
1       IN      PTR     firewall.office.corp.
``````
### Проверка и запуск
``````
sudo named-checkconf
sudo named-checkzone office.corp /etc/bind/db.office.corp
sudo systemctl restart bind9
sudo systemctl enable bind9
sudo systemctl status bind9
``````
- Если при проверке статуса все работает и выглядит как на изображении, значит все сделано правильно:
![bind9status](https://github.com/notforhealth/Network-engineering/blob/main/VM_office_corp_2/images/bind9.png)

## Настройка DHCP сервера

### Установка и настройка на domaincontroller
- Устанавливаем DHCP сервер:
``````
sudo apt install isc-dhcp-server -y
``````

- Настраиваем интерфейс для прослушивания:
``````
sudo nano /etc/default/isc-dhcp-server
``````
``````
INTERFACESv4="enp0s3"
``````
- Настраиваем основной конфиг:
``````
sudo nano /etc/dhcp/dhcpd.conf
``````
- Добавляем в конфиг:
``````
option domain-name "office.corp";
option domain-name-servers 192.168.20.10;
option routers 192.168.10.1;

default-lease-time 600;
max-lease-time 7200;

authoritative;

subnet 192.168.10.0 netmask 255.255.255.0 {
    range 192.168.10.10 192.168.10.100;
    option subnet-mask 255.255.255.0;
    option routers 192.168.10.1;
    option broadcast-address 192.168.10.255;
    option domain-name-servers 192.168.20.10;
}

subnet 192.168.20.0 netmask 255.255.255.0 {
}

subnet 192.168.30.0 netmask 255.255.255.0 {
}

host domaincontroller {
    hardware ethernet XX:XX:XX:XX:XX:XX; # MAC адрес domaincontroller
    fixed-address 192.168.20.10;
}

host webserver {
    hardware ethernet XX:XX:XX:XX:XX:XX; # MAC адрес webserver
    fixed-address 192.168.30.10;
}

host monitoringserver {
    hardware ethernet XX:XX:XX:XX:XX:XX; # MAC адрес monitoringserver
    fixed-address 192.168.20.100;
}

``````
### Запуск DHCP сервера
``````
sudo systemctl restart isc-dhcp-server
sudo systemctl enable isc-dhcp-server
sudo systemctl status isc-dhcp-server
``````
- По итогу должны получить:
![dhcp](https://github.com/notforhealth/Network-engineering/blob/main/VM_office_corp_2/images/dhcp_server.png)

## Настройка МЭ
- Для начала необходимо с клиента зайти в веб-интерфейс pfSense. Для этого переходим по ссылке https://192.168.10.1. Логин admin, пароль pfsense, но при первом заходе его необходимо поменять.
### Правила для LAN
- Firewall - Rules - LAN. Добавляем правило для DNS сервера. Протокол TCP/UDP. Source - LAN Subnets. Destination - single host 192.168.20.10. Port - 53.
- Также временно добавим правило разрешающее все входящие пакеты. Убрать это правило можно после настройки всей системы.

### Правила для DMZ
- Firewall - Rules - DMZ. Добавляем разрешающие правила по протоколу TCP с Destination 192.168.30.10. Порты http(80) и https(443) (для каждого порта свое правило).
- Запрещаем доступ из DMZ во внутренние сети. Добавляем правило где Action - Block, любые протоколы, Source - DMZ subnets, Destination - LAN subnets.
- Сохраняем и применяем

### Настройка NAT
- Firewall - NAT - Port Forward. Жмем Add. Здесь заполняем: Interface - WAN, Protocol - TCP, Source - any, Destination - WAN address, Port - 80. Redirect target IP - 192.168.30.10, redirect port - 80.
- Сохраняем и применяем


## Настройка NTP сервера
### Установка и настройка на domaincontroller
- Устанавливаем NTP сервер
``````
sudo apt install chrony -y
``````
- Chrony - программа для синхронизации времени в компьютерных системах. Состоит из двух компонентов: chronyd - для корректировки локального времени по внешним источникам, и chronyc - для управления и мониторинга службы.
- Настраиваем chrony
``````
sudo nano /etc/chrony/chrony.conf
``````
- Добавляем:
``````
allow 192.168.10.0/24
allow 192.168.20.0/24
allow 192.168.30.0/24

local stratum 10 //указатель stratum поставлен для примера
``````
- Включаем chrony
``````
sudo systemctl restart chrony
sudo systemctl enable chrony
sudo systemctl status chrony
``````
- Должна получиться такая картина:
![chrony](https://github.com/notforhealth/Network-engineering/blob/main/VM_office_corp_2/images/chrony.png)

### Настройка на серверах
- На webserver прописывает в терминал:
``````
sudo apt install chrony -y
sudo nano /etc/chrony/chrony.conf

#добавляем в конфиг
server dc01.office.corp iburst
#

sudo systemctl restart chrony
``````
- Аналогично на monitoringserver
### Настройка на клиенте
- На клиенте можно настроить с помощью timedatectl
``````
sudo timedatectl set-ntp false
sudo nano /etc/systemd/timesyncd.conf

# меняем конфиг
[Time]
NTP=dc01.office.corp
FallbackNTP=0.ubuntu.pool.ntp.org 1.ubuntu.pool.ntp.org
#

sudo systemctl restart systemd-timesyncd
sudo systemctl enable systemd-timesyncd
``````

## Настройка Веб сервера

### Установка apache на webserver
- Apache - популярный веб-сервер для размещения сайтов
``````
sudo apt update && sudo apt upgrade -y
sudo apt install apache2 -y
``````
### Создание простой веб-страницы
``````
sudo nano /var/www/html/index.html
``````
- Содержимое изменяем для примера

### Настройка виртуального хоста
``````
sudo nano /etc/apache2/sites-available/office.corp.conf
``````
- Изменяем конфиг:
``````
<VirtualHost *:80>
    ServerName office.corp
    ServerAlias www.office.corp
    DocumentRoot /var/www/html
    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
``````

- Включаем хост
``````
sudo a2ensite office.corp.conf
sudo a2dissite 000-default.conf // отключаем дефолтный сайт
sudo systemctl reload apache2
``````

- Для проверки заходим на http://192.168.30.10 через клиент. Откроется тестовая страница: 
![apachetest](https://github.com/notforhealth/Network-engineering/blob/main/VM_office_corp_2/images/web_server_site.png)


## Система мониторинга
- Prometheus - система мониторинга и оповещения, которая собирает метрики из различных приложений и серверов. Использует метод извлечения данных, когда сама обращается к целевым объектам, чтобы получить нужную информацию, и хранит ее как базу данных временных рядов.
- Метрики - числовые данные, которые измеряют различные аспекты работы системы.

### Установка Prometheus на monitoringserver
``````
sudo useradd --no-create-home --shell /bin/false prometheus
sudo mkdir /etc/prometheus
sudo mkdir /var/lib/Prometheus
cd /tmp
wget https://github.com/prometheus/prometheus/releases/download/v2.47.0/prometheus-2.47.0.linux-amd64.tar.gz
tar xvf prometheus-2.47.0.linux-amd64.tar.gz
cd prometheus-2.47.0.linux-amd64
sudo cp prometheus /usr/local/bin/
sudo cp promtool /usr/local/bin/
sudo chown prometheus:prometheus /usr/local/bin/prometheus
sudo chown prometheus:prometheus /usr/local/bin/promtool
sudo cp -r consoles /etc/prometheus
sudo cp -r console_libraries /etc/prometheus
sudo cp prometheus.yml /etc/prometheus/
sudo chown -R prometheus:prometheus /etc/prometheus
sudo chown -R prometheus:prometheus /var/lib/prometheus
``````
### Настройка prometheus
``````
sudo nano /etc/prometheus/prometheus.yml
``````
- Меняем содержимое
``````
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  - job_name: 'node_exporter_servers'
    static_configs:
      - targets: 
          - '192.168.20.10:9100' 
          - '192.168.30.10:9100'    
          - '192.168.20.100:9100'  
``````
- Создание systemd службы для prometheus
``````
sudo nano /etc/systemd/system/prometheus.service
``````
``````
[Unit]
Description=Prometheus Time Series Collection and Processing Server
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
``````
- Запускаем prometheus
``````
sudo systemctl daemon-reload
sudo systemctl start prometheus
sudo systemctl enable prometheus
sudo systemctl status prometheus
``````
![prometheus](https://github.com/notforhealth/Network-engineering/blob/main/VM_office_corp_2/images/prometheus.png)

### Установка Node Exporter
- Node Exporter - агент системы мониторинга prometheus, который и собирает метрики с серверов Linux и предоставляет их в формате, понятном для prometheus.
- На domaincontroller, webserver, monitoringserver выполняем:
``````
sudo useradd --no-create-home --shell /bin/false node_exporter
cd /tmp
wget https://github.com/prometheus/node_exporter/releases/download/v1.6.1/node_exporter-1.6.1.linux-amd64.tar.gz
tar xvf node_exporter-1.6.1.linux-amd64.tar.gz
cd node_exporter-1.6.1.linux-amd64
sudo cp node_exporter /usr/local/bin/
sudo chown node_exporter:node_exporter /usr/local/bin/node_exporter
sudo nano /etc/systemd/system/node_exporter.service
``````
``````
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
``````
``````
sudo systemctl daemon-reload
sudo systemctl start node_exporter
sudo systemctl enable node_exporter
sudo systemctl status node_exporter
``````
- По итогу должны получить:
![node_exporter](https://github.com/notforhealth/Network-engineering/blob/main/VM_office_corp_2/images/node_exporter.png)

### Установка Grafana на monitoringserver
- Grafana - система визуализации метрик в графическом виде
``````
wget https://dl.grafana.com/oss/release/grafana_10.2.3_amd64.deb
sudo dpkg -i grafana_10.2.3_amd64.deb
sudo systemctl start grafana-server
sudo systemctl enable grafana-server
sudo systemctl status grafana-server
``````
- Итог:
![grafana](https://github.com/notforhealth/Network-engineering/blob/main/VM_office_corp_2/images/grafana.png)

### Настройка Grafana на клиенте
- Открываем на клиенте в браузере http://192.168.20.100:3000. Логин admin, пароль admin, который при первом входе попросят поменять.
- Добавляем источник данных - prometheus - url: http://localhost:9090 - save & test.
![grafana_add_prometheus](https://github.com/notforhealth/Network-engineering/blob/main/VM_office_corp_2/images/grafana_options.png)
- Импортируем дашборд через + - import...
- Получаем такую картину:
![grafana_web_client](https://github.com/notforhealth/Network-engineering/blob/main/VM_office_corp_2/images/grafana_dashboard.png)

## Система резервного копирования
### Настройка автоматического бэкапа конфигов на domaincontroller
``````
sudo mkdir /opt/backup
sudo nano /opt/backup/backup_script.sh
``````

- Содержимое скрипта:
``````
#!/bin/bash

BACKUP_DIR="/opt/backup"
DATE=$(date +%Y%m%d_%H%M%S)
BACKUP_NAME="config_backup_$DATE.tar.gz"

mkdir -p /tmp/backup

cp -r /etc/bind /tmp/backup/
cp -r /etc/dhcp /tmp/backup/
cp /etc/netplan/* /tmp/backup/

tar -czf $BACKUP_DIR/$BACKUP_NAME -C /tmp/backup .
rm -rf /tmp/backup

find $BACKUP_DIR -name "config_backup_*.tar.gz" -mtime +7 -delete

echo "Backup completed: $BACKUP_DIR/$BACKUP_NAME"
``````
- Делаем скрипт исполняемым и добавляем в cron (автоматическое выполнение)
``````
sudo chmod +x /opt/backup/backup_script.sh
sudo crontab -e
``````
- Добавляем строку
``````
0 2 * * * /opt/backup/backup_script.sh
``````
- Означает, что скрипт будет запускаться каждый день в 2:00


## Тестирование
### Проверка DNS
- С клиента:
![DNS_check](https://github.com/notforhealth/Network-engineering/blob/main/VM_office_corp_2/images/dns_check.png)

### Проверка DHCP
- Перезагрузим клиента, после загрузки он должен получить IP-адрес
![DHCP_check](https://github.com/notforhealth/Network-engineering/blob/main/VM_office_corp_2/images/dhcp_check.png)

### Проверка веб сервера
- Из браузера клиента: http://192.168.30.10
![web_check](https://github.com/notforhealth/Network-engineering/blob/main/VM_office_corp_2/images/web_server_site.png)

### Проверка мониторинга
- Из браузера клиента: http://192.168.20.100:3000 для Grafana, http://192.168.20.100:9090 для Prometheus.

![grafana_check](https://github.com/notforhealth/Network-engineering/blob/main/VM_office_corp_2/images/grafana_check.png)
![prometheus_check](https://github.com/notforhealth/Network-engineering/blob/main/VM_office_corp_2/images/prometheus_check.png)

### Проверка фаервола
- С клиента зайти на http://192.168.30.10 - должно работать
- С webserver с помощью curl подключиться к http://192.168.10.1 - не должно работать


## Итоги
### Что работает в системе:
- Сетевая сегментация
- Фаервол с правилами
- DNS - внутренний домен office.corp
- DHCP
- NTP
- Веб-сервер - работающий сайт
- Мониторинг - Prometheus+Grafana
- Резервное копирование - автоматические бэкапы



