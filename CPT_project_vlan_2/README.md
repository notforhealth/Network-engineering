# Локальная сеть с vlan
В данном проекте реализована локальная сеть офиса компании с 3 рабочими кабинетами и основным помещением работников. Имитация выхода в интернет реализована с помощью маршрутизатора провайдера и сервера.

## Содержание
- [Технологии](#технологии)
- [Используемые аппаратные средства](#используемые-аппаратные-средства)
- [Топология сети](#топология-сети)
- [Распределение ip-адресов](#распределение-ip-адресов)
- [Настройка коммутаторов](#настройка-коммутаторов)
- [Настройка маршрутизаторов, ПК и серверов](#настройка-маршрутизаторов-пк-и-серверов)
- [Настройка NAT](#настройка-nat)
- [Тестирование](#тестирование)

## Технологии
- [Cisco Packet Tracer](https://www.netacad.com/cisco-packet-tracer)

## Используемые аппаратные средства
- [Маршрутизатор Cisco CISCO2901-V/K9](https://cisco-russia.ru/cisco-cisco2901-v-k9)
- [Коммутатор Cisco Catalyst WS-C2960-24TT-L](https://cisco-russia.ru/cisco-ws-c2960-24tt-l)
- Персональные компьютеры
- Серверы
- Copper Straight-Through - прямой медный кабель витой пары для соединения устройств


## Топология сети
![Топология сети](https://github.com/notforhealth/Network-engineering/blob/main/CPT_project_vlan_2/images/Network_topology.png)

## Распределение ip-адресов
| Device | Interface | IP-address/mask | Default gateway |
| --- | --- | --- | --- |
| Router0 | G0/0.2(vlan 2) | 192.168.2.1/24 ||
|| G0/0.3(vlan 3) | 192.168.3.1/24 ||
|| G0/0.4(vlan 4) | 192.168.4.1/24 ||
|| G0/0.5(vlan 5) | 192.168.5.1/24 ||
| Router2 | G0/0(to Router0) | 100.64.0.2/30 ||
|| G0/1(to Server1) | 100.64.1.1/24 ||
| Server1 | F0(to Router2) | 100.64.1.2/24 | 100.64.1.1 |
| PC0 | F0 | 192.168.2.3/24 | 192.168.2.1 |
| PC1 | F0 | 192.168.2.4/24 | 192.168.2.1 |
| PC2 | F0 | 192.168.3.3/24 | 192.168.3.1 |
| PC3 | F0 | 192.168.3.4/24 | 192.168.3.1 |
| PC4 | F0 | 192.168.4.3/24 | 192.168.4.1 |
| PC5 | F0 | 192.168.4.4/24 | 192.168.4.1 |
| PC6-PC20 | F0 | 192.168.5.3/24-192.168.5.17/24 | 192.168.5.1 |
| Server0 | F0 | 192.168.5.2/24 | 192.168.5.1 |


- По RFC 6598 адреса для соединений между маршрутизаторами используется диапазон ip-адресов: 100.64.0.0 - 100.127.255.255 - специальные "Shared Address Space" адреса.
- 192.168.x.x - стандарт для домашних/офисных сетей

## Настройка коммутаторов
### Настройка Switch1:
``````
SW1#conf t
SW1(config)#vlan 2
SW1(config-vlan)#ex
SW1(config)#int range f0/1-2
SW1(config-if-range)#switchport mode access
SW1(config-if-range)#switchport access vlan 2
SW1(config-if-range)#ex
SW1(config)#int g0/1
SW1(config-if)#switchport mode trunk
SW1(config-if)#switchport trunk allowed vlan 2
SW1(config-if)#ex
``````
По такому же принципу настраиваются Switch2, Switch3 и Switch4

### Настройка SwitchMain:
``````
SWmain#conf t
SWmain(config)#vlan 2
SWmain(config-vlan)#ex
SWmain(config)#vlan 3
SWmain(config-vlan)#ex
SWmain(config)#vlan 4
SWmain(config-vlan)#ex
SWmain(config)#vlan 5
SWmain(config-vlan)#ex
SWmain(config)#int range f0/1-4
SWmain(config-if-range)#switchport mode trunk
SWmain(config-if-range)#switchport trunk allowed vlan 2,3,4,5
``````

## Настройка маршрутизаторов, ПК и серверов

### Настройка Router0:
``````
R0#conf t
R0(config)#int f0/0.2
R0(config-if-subif)#encapsulation dot1Q 2
R0(config-if-subif)#ip address 192.168.2.1 255.255.255.0
R0(config-if-subif)#ex
R0(config)#int f0/0.3
R0(config-if-subif)#encapsulation dot1Q 3
R0(config-if-subif)#ip address 192.168.3.1 255.255.255.0
R0(config-if-subif)#ex
R0(config)#int f0/0.4
R0(config-if-subif)#encapsulation dot1Q 4
R0(config-if-subif)#ip address 192.168.4.1 255.255.255.0
R0(config-if-subif)#ex
R0(config)#int f0/0.5
R0(config-if-subif)#encapsulation dot1Q 5
R0(config-if-subif)#ip address 192.168.5.1 255.255.255.0
R0(config-if-subif)#ex

!Для настройки DHCP на ПК в vlan4:
R0(config)#int g0/0.5
R0(config-if-subif)#ip helper-address 192.168.5.2
R0(config-if-subif)#ex
``````

### Настройка Router2:
``````
Router#conf t
Router(config)#int g0/0
Router(config-if)#ip address 100.64.0.2 255.255.255.252
Router(config-if)#no sh
Router(config-if)#ex
Router(config)#int g0/1
Router(config-if)#ip address 100.64.1.1 255.255.255.252
Router(config-if)#no sh
Router(config-if)#ex
``````

### Настройка Server0
- IP-адрес: 192.168.5.2/24
- Основной шлюз: 192.168.5.1
- Во вкладке Services выбираем DHCP и настраиваем его под значения данной подсети:
Start IP Address: 192.168.5.3, Subnet Mask: 255.255.255.0, Maximum Numbers of Users: 253.

### Настройка Server1
- Через настройку IP Configuration задаем IP-адрес 100.64.1.2/24 с шлюзом 100.64.1.1

### Настройка ПК
- Все PC настраиваются через IP Configuration. Для vlan5 выбираем DHCP, остальные настраиваются вручную или через консоль.

## Настройка NAT
-Для настройки NAT на Router0 откроем CLI:
``````
R0(config)#int g0/1
R0(config-if)#ip address 100.64.0.1 255.255.255.252
R0(config-if)#no sh
R0(config-if)#ex
R0(config)#ip route 0.0.0.0 0.0.0.0 100.64.0.2
R0(config)#int g0/1
R0(config-if)#ip nat outside
R0(config-if)#ex
R0(config)#int g0/0.2
R0(config-if-subif)#ip nat inside
R0(config-if-subif)#ex
R0(config)#int g0/0.3
R0(config-if-subif)#ip nat inside
R0(config-if-subif)#ex
R0(config)#int g0/0.4
R0(config-if-subif)#ip nat inside
R0(config-if-subif)#ex
R0(config)#int g0/0.5
R0(config-if-subif)#ip nat inside
R0(config-if-subif)#ex
R0(config)#ip access list standard FOR-NAT
R0(config-std-nacl)#permit 192.168.2.0 0.0.0.255
R0(config-std-nacl)#permit 192.168.3.0 0.0.0.255
R0(config-std-nacl)#permit 192.168.4.0 0.0.0.255
R0(config-std-nacl)#permit 192.168.5.0 0.0.0.255
R0(config)#ip nat inside source list FOR-NAT interface g0/1 overload
``````

## Тестирование

- Для проверки работы можно использовать утилиту ping:

- ping с PC2 на PC13 и обратно(проверка работы vlan между друг другом)
![Проверка с помощью ping](https://github.com/notforhealth/Network-engineering/blob/main/CPT_project_vlan_2/images/ping_first.png)
- ping с PC1, PC16 на Server1 и обратно(проверка работы безопасности подключения к интернету)
![Проверка с помощью ping](https://github.com/notforhealth/Network-engineering/blob/main/CPT_project_vlan_2/images/ping_second.png)
- В данном случае увидим, что с Server1 не удается подключиться к компьютерам локальной сети, но с компьютеров к Server1 ping проходит (то есть доступ в интернет есть), а значит все работает правильно.








