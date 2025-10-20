# Локальная сеть с использованием безопасности сети на уровне доступа
В данном проекте реализована локальная сеть с настройкой Port Security, ACL, DHCP Snooping и 802.1Х аутентификацией.

## Содержание
- [Технологии](#технологии)
- [Используемые аппаратные средства](#используемые-аппаратные-средства)
- [Топология сети](#топология-сети)
- [Распределение ip-адресов](#распределение-ip-адресов)
- [Настройка маршрутизации](#настройка-маршрутизации)
- [Настройка серверов](#настройка-серверов)
- [Настройка Port Security](#настройка-port-security)
- [Настройка DHCP Snooping](#настройка-dhcp-snooping)
- [Настройка ACL](#настройка-acl)
- [Настройка 802.1X](#настройка-802.1x)
- [Проверка работы](#проверка-работы)

## Технологии
- [Cisco Packet Tracer](https://www.netacad.com/cisco-packet-tracer)

## Используемые аппаратные средства
- Маршрутизатор Cisco 2811
- [Коммутатор Cisco Catalyst WS-C2960-24TT-L](https://cisco-russia.ru/cisco-ws-c2960-24tt-l)
- Точка доступа
- Персональные компьютеры
- Ноутбук
- Сматрфон
- Copper Straight-Through - прямой медный кабель витой пары для соединения устройств


## Топология сети
![Топология сети](https://github.com/notforhealth/Network-engineering/blob/main/CPT_project_security/images/Network_topology.png)
- В данном проекте реализуется локальная сеть офиса с сетью для сотрудников и гостевой сетью. Для реализации гостевой сети применяется точка доступа как переход на беспроводную сеть. Точка доступа подключается к интерфейсу G0/1, чтобы использоваться только как переход, без образования отдельной локальной сети.

## Распределение ip-адресов
| Device | Interface | IP-address/mask | Default gateway |
| --- | --- | --- | --- |
| Router0 | F0/0 | 192.168.10.1/24 ||
|| F0/1 | 192.168.20.1/24 ||
|| E0/3/0 | 10.0.1.1/24 ||
| RADIUS Server| F0 | 10.0.1.10/24 | 10.0.1.1 |
| DHCP Server | F0 | 10.0.1.11/24 | 10.0.1.1 |
| PC0 | F0 | выдача по DHCP | 192.168.10.1 |
| PC1 | F0 | выдача по DHCP | 192.168.10.1 |
| Laptop0 | F0 | выдача по DHCP | 192.168.20.1 |
| Smartphone0 | F0 | выдача по DHCP | 192.168.20.1 |


- 192.168.x.x - стандарт для домашних/офисных сетей

## Настройка маршрутизации
### Router0
``````
en
conf t
int e0/3/0
ip address 10.0.1.1 255.255.255.0
no sh
ex

int f0/0
ip address 192.168.10.1 255.255.255.0
no sh
ex

int f0/1
ip address 192.168.20.1 255.255.255.0
no sh
ex

ip routing
ex
w
``````

## Настройка серверов
### RADIUS-сервер
- После назначения IP-адреса, маски и шлюза, переходим во вкладку Services, после в AAA. Здесь создаем пользователей под PC0 и PC1: user1/password1, user2/password2. Также создаем клиента RADIUS (им является коммутатор сети сотрудников): Client IP: 192.168.10.2, secret: secretkey.

### DHCP-сервер
- После назначения IP-адреса, маски и шлюза, переходим во вкладку Services, после в DHCP. Здесь создаем пул-адресов, который будет выдаваться конечным устройствам: шлюз: 192.168.10.1, start IP: 192.168.10.10, маска: 255.255.255.0, max users: 50.

## Настройка Port Security
### Switch1
``````
en
conf t
int range f0/1-2
switchport mode access
switchport port-security
switchport port-security maximum 3 !максимальное количество mac-адресов для порта
switchport port-security violation shutdown !при количества mac-адресов больше заданного выключить порт
switchport port-security mac-address sticky !фиксирует первый подключенный mac-адрес
no sh
ex
``````

### Switch2
``````
en
conf t
int f0/1
switchport mode access
switchport port-security
switchport port-security maximum 3 !максимальное количество mac-адресов для порта
switchport port-security violation shutdown !при количества mac-адресов больше заданного выключить порт
switchport port-security mac-address sticky !фиксирует первый подключенный mac-адрес
no sh
ex
``````

## Настройка DHCP Snooping
### Switch1
``````
en
conf t
ip dhcp snooping
int g0/1 !интерфейс, идущий на маршрутизатор
ip dhcp snooping trust !доверенный интерфейс
ex

int range f0/1-2
no ip dhcp snooping trust
ex
ex
w
``````

### Switch2
``````
en
conf t
ip dhcp snooping
int g0/1 !интерфейс, идущий на маршрутизатор
ip dhcp snooping trust !доверенный интерфейс
ex

int f0/1
no ip dhcp snooping trust
ex
ex
w
``````

## Настройка ACL

### Router0
``````
en
conf t
ip access-list extended BLOCK-GUESTS
deny ip 192.168.20.0 0.0.0.255 10.0.1.0 0.0.0.255 !запрет на доступ к серверной сети
deny ip 192.168.20.0 0.0.0.255 192.168.10.0 0.0.0.255 !запрет на доступ к сети сотрудников
permit ip any any !разрешаем весь остальной трафик
ex

int f0/1
ip access-group BLOCK-GUESTS in !применяем ACL к интерфейсу гостевой сети (фильтрация входящего трафика)
ex
ex
w
``````


## Настройка 802.1X
### Switch1
``````
en
conf t
dot1x system-auth-control !включаем 802.1X
aaa new-model 
aaa authentication dot1x default group radius !создаем метод аутентификации через radius
radius server RADIUS !настройка radius сервера
address ipv4 10.0.1.10 auth-port 1645
key secretkey
ex

int range f0/1-2
dot1x pae authenticator
dot1x port-control auto !режим порта, требующий аутентификации(в cisco packet tracer реализован не полностью, поэтому не работает)
ex
ex
w
``````

### Настройка клиента для 802.1X
- В окне IP-configuration вводим данные пользователя от 802.1X и включаем запрос IP-адреса по DHCP.

## Проверка работы
### Port Security
![Port Security](https://github.com/notforhealth/Network-engineering/blob/main/CPT_project_security/images/port-security.png)

### DHCP Snooping
![DHCP Snooping](https://github.com/notforhealth/Network-engineering/blob/main/CPT_project_security/images/dhcp_snooping.png)
### ACL
![ACL1](https://github.com/notforhealth/Network-engineering/blob/main/CPT_project_security/images/acl.png)
![ACL2](https://github.com/notforhealth/Network-engineering/blob/main/CPT_project_security/images/ping_acl.png)









