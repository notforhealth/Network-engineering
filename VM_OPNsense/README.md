# Создание маршрутизатора и программного межсетевого экрана с помощью OPNsense
- OPNsense - операционная система на базе FreeBSD, заточенная под задачи маршрутизатора и МЭ. В данном проекте OPNsense также будет выполнять роль DHCP сервера для локальной сети.
- Программный межсетевой экран - ПО, которое защищает устройство или сеть, фильтруя трафик по заданным правилам.

## Содержание
- [Используемые средства](#используемые-средства)
- [Логическая схема](#логическая-схема)
- [Установка всех необходимых средств](#установка-всех-необходимых-средств)
- [Настройка сетевых адаптеров](#настройка-сетевых-адаптеров)
- [Настройка OPNsense](#настройка-opnsense)
- [Настройка Debian](#настройка-debian)
- [Настройка DHCP](#настройка-dhcp)
- [Настройка МЭ](#настройка-мэ)
- [Тестирование](#тестирование)

## Используемые средства
- Персональный компьютер
- Операционная система [OPNsense](https://opnsense.org/download)
- Операционная система [Debian](https://www.debian.org/distrib/index.ru.html)
- [Oracle VirtualBox](https://www.virtualbox.org)


## Логическая схема
![Логическая схема](https://github.com/notforhealth/Network-engineering/blob/main/VM_OPNsense/images/logical_scheme.png)


## Установка всех необходимых средств
### VirtualBox
- Для установки VB необходимо перейти по ссылке https://www.virtualbox.org и произвести базовую установку приложения.
### OPNsense
- Для установки OPNsense переходим по ссылке https://opnsense.org/download и выбираем amd64, dvd, OPNsense. Далее необходимо импортировать iso файл в VirtualBox. Нажимаем создать вписываем название, выбираем образ iso, выбираем необходимое количество оперативной памяти и число ядер процессора.
### Debian
- Для установки Debian скачиваем образ Debian 13.1.0 с сайта https://www.debian.org/distrib/index.ru.html. Импортируем также как и OPNsense, но указываем логин, пароль и имя хоста.
- В итоге получаем две виртуальные машины в VirtualBox:

- ![Виртуальные машины](https://github.com/notforhealth/Network-engineering/blob/main/VM_OPNsense/images/VM.png)

## Настройка сетевых адаптеров
### Для OPNsense
- Необходимо добавить второй сетевой адаптер с внутренней сетью:
![Внутренняя сеть](https://github.com/notforhealth/Network-engineering/blob/main/VM_OPNsense/images/adapter_opnsense_first.png)
### Для Debian
- Необходимо переключить с NAT на внутреннюю сеть:
![Внутренняя сеть](https://github.com/notforhealth/Network-engineering/blob/main/VM_OPNsense/images/adapter_debian.png)


## Настройка OPNsense
### Установка
- Запускаем ВМ OPNsense и в момент загрузки система предложит настроить интерфейсы, соглашаемся и настраиваем em0 как WAN, а em1 как LAN:
![Настройка интерфейсов](https://github.com/notforhealth/Network-engineering/blob/main/VM_OPNsense/images/OPNsense/interfaces.png)
- Далее нужно продолжить настройку. Для этого заходим на аккаунт installer с паролем opnsense, после чего появится окно настроек OPNsense. Здесь необходимо выбрать UFS (файловая система, используемая в OPNsense) и подтвердить запись изменений на диск. В конце меняем пароль от пользователя root и выключаем ВМ. Отключаем iso файл из носителей OPNsense, чтобы при запуске ВМ не начала настраиваться заново.
- Запускаем ВМ и ждем загрузки, заходим под пользователем root.
- ![login](https://github.com/notforhealth/Network-engineering/blob/main/VM_OPNsense/images/OPNsense/login.png)
## Настройка Debian
- В ВМ Debian необходимо поменять IP-адрес на статический, для того чтобы попасть на веб версию OPNsense:
- ![ipv4](https://github.com/notforhealth/Network-engineering/blob/main/VM_OPNsense/images/debian/debian_changing_IP.png)
- Заходим через firefox в OPNsense:
![web site](https://github.com/notforhealth/Network-engineering/blob/main/VM_OPNsense/images/debian/debian_connection.png)
![web opnsense](https://github.com/notforhealth/Network-engineering/blob/main/VM_OPNsense/images/debian/login_OPNsense_web.png)


## Настройка DHCP
- В ВМ OPNsense в меню выбираем пункт 2 для включения DHCP. Выбираем LAN и вписываем тот же IP адрес с той же маской, что были и раньше. На пункте "DHCP server on LAN" выбираем да и записываем пул IP-адресов для этой подсети:
![dhcp](https://github.com/notforhealth/Network-engineering/blob/main/VM_OPNsense/images/OPNsense/OPNsense_changin_LAN_IP.png)
![dhcp](https://github.com/notforhealth/Network-engineering/blob/main/VM_OPNsense/images/OPNsense/OPNsense_changin_LAN_IP_2.png)
![dhcp](https://github.com/notforhealth/Network-engineering/blob/main/VM_OPNsense/images/OPNsense/OPNsense_changin_LAN_IP_3.png)
- После этого меняем на клиенте выдачу IP-адресов с помощью DHCP:
![dhcp](https://github.com/notforhealth/Network-engineering/blob/main/VM_OPNsense/images/debian/debian_changing_IP_DHCP.png)
![dhcp](https://github.com/notforhealth/Network-engineering/blob/main/VM_OPNsense/images/debian/debian_netrwork.png)
## Настройка МЭ
- Переходим в Firewall - Rules - LAN. Удаляем или выключаем работающие правила и добавляем новые правила.
- Создаем правило для HTTP: Action - pass, Protocol - TCP/UDP, Source - LAN net, Destination - any, Port Range: From - 80; to - 80.
- Такие же правила создаем для HTTPS(443 порт), DNS(53 порт).
- По итогу получим такую картину:
![firewall](https://github.com/notforhealth/Network-engineering/blob/main/VM_OPNsense/images/debian/firewall_LAN_rules.png)
## Тестирование
- Пробуем в браузере зайти на любой сайт:
![http](https://github.com/notforhealth/Network-engineering/blob/main/VM_OPNsense/images/debian/rules_test.png)
- Пробуем сделать пинг до внешнего адреса:
![ping](https://github.com/notforhealth/Network-engineering/blob/main/VM_OPNsense/images/debian/icmp_test.png)
- Можно сделать вывод, что правила работают, т.к. на сайт зайти получилось, то есть трафик по протоколам 80 и 443 проходит, а ICMP трафик не проходит, т.к. утилита ping не выдала ответа.













