# Локальная сеть с резервированием и отказоустойчивостью
В данном проекте реализована локальная сеть с резервными каналами и протоколами для обеспечения надежности.

## Содержание
- [Технологии](#технологии)
- [Используемые аппаратные средства](#используемые-аппаратные-средства)
- [Топология сети](#топология-сети)
- [Распределение ip-адресов](#распределение-ip-адресов)
- [Настройка EtherChannel](#настройка-etherchannel)
- [Настройка HSRP](#настройка-hsrp)
- [Настройка STP](#настройка-stp)
- [Настройка маршрутизации и PC](#настройка-маршрутизации-и-pc)
- [Тестирование](#тестирование)

## Технологии
- [Cisco Packet Tracer](https://www.netacad.com/cisco-packet-tracer)

## Используемые аппаратные средства
- [Маршрутизатор Cisco CISCO2901-V/K9](https://cisco-russia.ru/cisco-cisco2901-v-k9)
- [Коммутатор Cisco Catalyst WS-C2960-24TT-L](https://cisco-russia.ru/cisco-ws-c2960-24tt-l)
- [Коммутатор Cisco Catalyst WS-C3650-24PS-L](https://cisco-russia.ru/cisco-ws-c3650-24ps-l)
- Персональные компьютеры
- Copper Straight-Through - прямой медный кабель витой пары для соединения устройств


## Топология сети
![Топология сети](https://github.com/notforhealth/Network-engineering/blob/main/CPT_project_reservation/images/Network_topology.png)
- В подключении между Multilayer Switch 0 и 1 используется EtherChannel (технология агрегации каналов, позволяющая объединить нескольких портов в единый логический интерфейс). Для этого используется протокол LACP(Link Aggregation Control Protocol). Он автоматизирует процесс объединение портов, позволяя сетевым устройствам договариваться о формировании агрегированного канала. При отказе одного из каналов трафик автоматически перенаправляется через оставшиеся, обеспечивая бесперебойную работу сети.

## Распределение ip-адресов
| Device | Interface | IP-address/mask | Default gateway |
| --- | --- | --- | --- |
| Router0 | G0/0 | 192.168.10.2/24 ||
| Router1 | G0/0 | 192.168.10.3/24 ||
| Mult.Switch0 | vlan 10 | 192.168.10.4/24 ||
| Mult.Switch1 | vlan 10 | 192.168.10.5/24 ||
| PC0 | F0 | 192.168.10.10/24 | 192.168.10.1 |
| PC1 | F0 | 192.168.10.11/24 | 192.168.10.1 |

- 192.168.x.x - стандарт для домашних/офисных сетей

## Настройка EtherChannel
### Настройка Mult.Switch0:
``````
enable
conf t
int Port-channel 1
description LACP To Core-SW1
switchport mode trunk
ex
int range g1/0/1-4
description To Core-SW1
switchport mode trunk
channel-group 1 mode active
no sh
ex
w
``````

### Настройка Mult.Switch1:
``````
enable
conf t
int Port-channel 1
description LACP To Core-SW0
switchport mode trunk
ex
int range g1/0/1-4
description To Core-SW0
switchport mode trunk
channel-group 1 mode active
no sh
ex
w
``````

## Настройка HSRP

- HSRP(Hot Standby Router Protocol) - протокол компании Cisco для обеспечения отказоустойчивости шлюза по умолчанию в сети. Объединяет несколько маршрутизаторов в группу, которая предоставляет виртуальный IP и MAC-адреса, используемый устройствами как единый шлюз. Один маршрутизатор активен и обрабатывает трафик, второй находится в резерве и готов к работе при сбое активного маршрутизатора.

### Настройка Router0:
``````
en
conf t
int g0/0
ip address 192.168.10.2 255.255.255.0
standby version 2
standby 10 ip 192.168.10.1
standby 10 priority 150 !Высший приоритет
standby 10 preempt !Возможность захватывать роль Active
no sh
ex
w
``````

### Настройка Router1:
``````
en
conf t
int g0/0
ip address 192.168.10.3 255.255.255.0
standby version 2
standby 10 ip 192.168.10.1
standby 10 priority 100 !Приоритет ниже
standby 10 preempt
no sh
ex
w
``````

## Настройка STP
- STP(Spanning Tree Protocol) - протокол, который предотвращает образование петель в сети с избыточными соединениями, чтобы избежать потери данных и зацикливания. Петли - ситуация, когда сетевой трафик бесконечно циркулирует по замкнутому маршруту, что приводит к перегрузке сети и ее неработоспособности.

### Настройка Mult.Switch0:
``````
enable
conf t
spanning-tree mode rapid-pvst
spanning-tree vlan 1-4094 priority 0 !Этот коммутатор будет являться корневым мостом
int g0/10 !Интерфейс к Switch0
switcport mode trunk
ex
w
``````
### Настройка Mult.Switch1:
``````
enable
conf t
spanning-tree mode rapid-pvst
spanning-tree vlan 1-4094 priority 4096 !Резервный корневой мост
int g0/10 
switcport mode trunk
ex
w
``````
### Настройка Switch0:
``````
enable
conf t
int range g0/1-2
switchport mode trunk
spanning-tree portfast trunk !Ускоряет поднятие линка
ex
int range f0/1-24
switchport mode access
spanning-tree portfast !Отключает STP на портах к конечным устройствам
ex
w
``````

## Настройка маршрутизации и PC
### Настройка Mult.Switch0:
``````
enable
conf t
int vlan 10
ip address 192.168.10.4 255.255.255.0
no sh
ex
ip routing
``````

### Настройка Mult.Switch1:
``````
enable
conf t
int vlan 10
ip address 192.168.10.5 255.255.255.0
no sh
ex
ip routing
``````

### Настройка ПК
- Все PC настраиваются через IP Configuration.

## Тестирование

### Ping с PC0 на PC1
![Проверка с помощью ping](https://github.com/notforhealth/Network-engineering/blob/main/CPT_project_reservation/images/ping_PC.png)
### Ping с PC0 на PC1 с выключением интерейса на Router0
- Выполним ping -t 192.168.10.11 на PC0. Во время работы ping отключим интерфейс на Router0.
![Проверка с помощью ping](https://github.com/notforhealth/Network-engineering/blob/main/CPT_project_reservation/images/ping_PC.png)
- В данном случае увидим, что в момент выключения и включения повышается время передачи пакета, а Router1 переводится в режим Active и Standby соответственно.
### Тест EtherChannel
- Выполним ping -t 192.168.10.11 на PC0. Во время работы ping удалим 2 кабеля между Mult.Switch 0 и 1.
![Проверка с помощью ping](https://github.com/notforhealth/Network-engineering/blob/main/CPT_project_reservation/images/ping_PC.png)
- В данном случае увидим, что из-за отключения кабелей несколько пакетов передалось с задержкой, но потом соединение стабилизировалось.
### Тест STP
- Выключим порт на Mult.Switch0, ведущий к Switch0. Из-за этого STP пересчитает дерево, и Switch0 теперь будет использовать путь через Mult.Switch1.









