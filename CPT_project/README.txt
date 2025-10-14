# Локальная сеть
В данном проекте реализована локальная сеть с "выходом в интернет" в виде Loopback. 

## Содержание
- [Технологии](#технологии)
- [Топология сети](#топология-сети)
- [Распределение ip-адресов](#распределение-ip-адресов)
- [Настройка маршрутизаторов и PC](#настройка-маршрутизаторов-и-pc)
- [Тестирование](#тестирование)

## Технологии
- [Cisco Packet Tracer](https://www.netacad.com/cisco-packet-tracer)

## Топология сети
[Топология сети](https://github.com/notforhealth/Network-engineering/blob/main/CPT_project/images/Network_topology.png)

## Распределение ip-адресов
| Device | Interface | IP-address/mask | Default gateway |
| --- | --- | --- | --- |
| ISP_Router | Loopback0(Internet) | 8.8.8.8/32 ||
|| G0/0(to Router0) | 100.64.0.1/30 ||
|| G0/1(to Router1) | 100.64.0.5/30 ||
| Router0 | G0/0(to ISP_Router) | 100.64.0.2/30 ||
|| G0/1(to switch) | 192.168.1.1/24 ||
| Router1 | G0/0(to ISP_Router) | 100.64.0.6/30 ||
|| G0/1(to switch) | 192.168.2.1/24 ||
| PC0 | F0 | 192.168.1.3/24 | 192.168.1.1 |
| PC1 | F0 | 192.168.1.4/24 | 192.168.1.1 |
| PC2 | F0 | 192.168.1.5/24 | 192.168.1.1 |
| PC3 | F0 | 192.168.2.3/24 | 192.168.1.1 |
| PC4 | F0 | 192.168.2.4/24 | 192.168.1.1 |

По RFC 6598 адреса для соединений между маршрутизаторами используется диапазон ip-адресов: 100.64.0.0 - 100.127.255.255 - специальные "Shared Address Space" адреса.
192.168.x.x - стандарт для домашних/офисных сетей
8.8.8.8 - адрес dns.google.com - как пример для выхода в сеть

## Настройка маршрутизаторов и PC
Настройка ISP_Router:
``````
enable
configure terminal

hostname ISP_Router

int g0/0
ip address 100.64.0.1 255.255.255.252
no shutdown
exit

int g0/1
ip address 100.64.0.1 255.255.255.252
no shutdown
exit

int Loopback0
ip address 8.8.8.8 255.255.255.255
exit

ip route 192.168.1.0 255.255.255.0 100.64.0.2
ip route 192.168.2.0 255.255.255.0 100.64.0.6

``````

Настройка Router0:
``````
enable
configure terminal

hostname Router0

int g0/0
ip address 100.64.0.2 255.255.255.252
no shutdown
exit

int g0/1
ip address 192.168.1.1 255.255.255.0
no shutdown
exit

ip route 0.0.0.0 0.0.0.0 100.64.0.1

int g0/0
ip nat inside
exit

int g0/1
ip nat outside
exit

access-list 1 permit 192.168.1.0 0.0.0.255
ip nat inside source list 1 int g0/1 overload

``````

Настройка Router1:
``````
enable
configure terminal

hostname Router1

int g0/0
ip address 100.64.0.6 255.255.255.252
no shutdown
exit

int g0/1
ip address 192.168.2.1 255.255.255.0
no shutdown
exit

ip route 0.0.0.0 0.0.0.0 100.64.0.5
int g0/0
ip nat inside
exit

int g0/1
ip nat outside
exit

access-list 1 permit 192.168.2.0 0.0.0.255
ip nat inside source list 1 int g0/1 overload

``````

Все PC настраиваются через IP Configuration





