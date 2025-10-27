# Создание IPSec туннеля
В данном проекте реализован IPSec туннель с использованием маршрутизатора провайдера.
IPSec - набор протоколов, обеспечивающих безопасную передачу данных по IP-сетям путем шифрования, аутентификации и обеспечения целостности пакетов. Работает на сетевом уровне и широко используется для создания VPN-соединений, защищая данные от перехвата и постороннего вмешательства.

## Содержание
- [Технологии](#технологии)
- [Используемые аппаратные средства](#используемые-аппаратные-средства)
- [Топология сети](#топология-сети)
- [Распределение ip-адресов](#распределение-ip-адресов)
- [Настройка маршрутизаторов и ПК](#настройка-маршрутизаторов-и-пк)
- [Настройка IPSec](#настройка-ipsec)
- [Тестирование](#тестирование)

## Технологии
- [Cisco Packet Tracer](https://www.netacad.com/cisco-packet-tracer)

## Используемые аппаратные средства
- [Маршрутизатор Cisco CISCO2901-V/K9](https://cisco-russia.ru/cisco-cisco2901-v-k9)
- [Коммутатор Cisco Catalyst WS-C2960-24TT-L](https://cisco-russia.ru/cisco-ws-c2960-24tt-l)
- Персональные компьютеры
- Copper Straight-Through - прямой медный кабель витой пары для соединения устройств


## Топология сети
![Топология сети](https://github.com/notforhealth/Network-engineering/blob/main/CPT_project_vpn_5/images/network_topology.png)

## Распределение ip-адресов
| Device | Interface | IP-address/mask | Default gateway |
| --- | --- | --- | --- |
| Router0 | G0/0 | 10.1.0.2/30 ||
|| G0/1 | 10.2.0.2/30 ||
| Router1 | G0/0 | 192.168.10.1/24 ||
|| G0/1 | 10.1.0.1/30 ||
| Router2 | G0/0 | 192.168.11.1/24 ||
|| G0/1 | 10.2.0.1/30 ||
| PC0 | F0 | 192.168.10.11/24 | 192.168.10.1 |
| PC1 | F0 | 192.168.11.11/24 | 192.168.11.1 |

- 192.168.x.x - стандарт для домашних/офисных сетей


## Настройка маршрутизаторов и ПК

### Настройка Router0:
``````
R0#conf t
R0(config)#int g0/0 
R0(config-if)#ip address 10.1.0.2 255.255.255.252
R0(config-if)##no sh
R0(config-if)#ex
R0(config)#int g0/1
R0(config-if)#ip address 10.2.0.2 255.255.255.252
R0(config-if)##no sh
R0(config-if)#ex
``````

### Настройка Router1:
``````
R1#conf t
R1(config)#int g0/0
R1(config-if)#ip address 192.168.10.1 255.255.255.0
R1(config-if)#no sh
R1(config-if)#ex
R1(config)#int g0/1
R1(config-if)#ip address 10.1.0.1 255.255.255.252
R1(config-if)#no sh
R1(config-if)#ex
R1(config)#ip route 0.0.0.0 0.0.0.0 10.1.0.2

``````

### Настройка Router2:
``````
R2#conf t
R2(config)#int g0/1
R2(config-if)#ip address 192.168.11.1 255.255.255.0
R2(config-if)#no sh
R2(config-if)#ex
R2(config)#int g0/0
R2(config-if)#ip address 10.2.0.1 255.255.255.252
R2(config-if)#no sh
R2(config-if)#ex
R2(config)#ip route 0.0.0.0 0.0.0.0 10.2.0.2
``````

### Настройка ПК
- Все PC настраиваются через IP Configuration.

## Настройка IPSec
### Настройка IPSec на Router1:
- Для начала необходимо подключить модуль безопасности Router1 и Router2, после чего перезагрузить маршрутизатор:
``````
R1(config)#license boot module c2900 technology-package securityk9 
R1(config)#ex
R1#reload

``````
- Аналогично для второго маршрутизатора
- Далее нужно создать ACL для управления трафиком IPSec туннеля:
``````
R1(config)#access-list 100 permit ip 192.168.10.0 0.0.0.255 192.168.11.0 0.0.0.255
``````
- Настраиваем протоколы для IPSec туннеля:
``````
R1(config)#crypto isakmp policy 10 !приоритет политики защиты передаваемых данных
R1(config-isakmp)#authentication pre-share !метод аутентификации PSK
R1(config-isakmp)#encryption aes 256 !метод шифрования AES с длинной ключа 256
R1(config-isakmp)#group 5 !использование алгоритма Диффи-Хеллмана
``````

- PSK (Pre-Shared Key) - метод аутентификации, использующий общий ключ для подключения к сети VPN/Wi-Fi.
- AES (Advanced Encryption Standart) - симметричный блочный алгоритм шифрования, использующий ключи длиной 128, 192 ил 256 для шифрования блоков данных размеров 128 бит.
- Алгоритм Диффи-Хеллмана используют для создания общего секретного ключа двумя сторонами, используя незащищенный от прослушивания канал связи. Данный ключ нужен для шифрования дальнейшего обмена с помощью алгоритмов симметричного шифрования.


- Создание ключа для шифрования (password - ключ для примера):
``````
R1(config)#crypto isakmp key password address 10.2.0.1 !адрес указывается для понимания, с кем будет организован информационный обмен
``````

- Создание туннеля IPSec:
``````
R1(config)#crypto ipsec transform-set R1-R2 esp-aes 256 esp-sha-hmac
!transform-set - набор преобразований, комбинация протоколов и алгоритмов безопасности
!esp-aes - протокол, использующий AES для шифрования данных. Обеспечивает конфиденциальность, целостность и аутентификацию данных пакета
!esp-sha-hmac - протокол, использующий хэш-функцию SHA и код проверки подлинности HMAC, для защиты целостности пакета
``````
- Создание криптокарты IPSec:
``````
R1(config)#crypto map CrMap 10 ipsec-isakmp 
R1(config-crypto-map)#set peer 10.2.0.1 !с каким узлом будет осуществляться взаимодействие
R1(config-crypto-map)#set pfs group5 !запуск алгоритма Диффи-Хеллмана
R1(config-crypto-map)#set security-association lifetime seconds 86400 !время жизни туннеля
R1(config-crypto-map)#set transform-set R1-R2 !указание набора преобразований
R1(config-crypto-map)#match address 100 !источник пакетов для туннеля с указанием ACL
R1(config-crypto-map)#ex
R1(config)#int g0/1
R1(config-if)#crypto map CrMap !указание криптокарты для интерфейса
``````

### Настройка IPSec на Router2:
``````
R2(config)#access-list 100 permit ip 192.168.11.0 0.0.0.255 192.168.10.0 0.0.0.255
R2(config)#crypto isakmp policy 10
R2(config-isakmp)#authentication pre-share 
R2(config-isakmp)#encryption aes 256
R2(config-isakmp)#group 5
R2(config-isakmp)#ex
R2(config)#crypto isakmp key password address 10.1.0.1
R2(config)#crypto ipsec transform-set R2-R1 esp-aes 256 esp-sha-hmac 
R2(config)#crypto map CrMap 10 ipsec-isakmp 
R2(config-crypto-map)#set peer 10.1.0.1
R2(config-crypto-map)#set pfs group5
R2(config-crypto-map)#set security-association lifetime seconds 86400
R2(config-crypto-map)#set transform-set R2-R1
R2(config-crypto-map)#match address 100
R2(config-crypto-map)#ex
R2(config)#int g0/0
R2(config-if)#crypto map CrMap
``````


## Тестирование

- Для проверки работы напишем в CLI Router2 команду "show crypto ipsec sa":
![Router2 show_first](https://github.com/notforhealth/Network-engineering/blob/main/CPT_project_vpn_5/images/show_ipsec_first.png)

- Увидим количество инкапсулированных пакетов (pkts encaps) равным 14.

- Отправим эхо-запрос с PC0 на PC1:
![Эхо-запрос](https://github.com/notforhealth/Network-engineering/blob/main/CPT_project_vpn_5/images/ping.png)

- Теперь заново посмотрим в CLI Router2 количество инкапсулированных пакетов:
![Router2 show_second](https://github.com/notforhealth/Network-engineering/blob/main/CPT_project_vpn_5/images/show_ipsec_second.png)

- Увидим, что количество инкапсулированных пакетов увеличилось. Это означает, что IPSec туннель работает исправно.











