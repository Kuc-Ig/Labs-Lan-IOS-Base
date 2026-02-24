![](./1.png)

### Таблица адресации
| Устройство           |   Интерфейс     | 		IP-адрес        |   Маска подсети             | 	Шлюз по умолчанию     |
|:---------------------|:---------------:|:---------------------|:---------------------------:|:--------------------------|
|R1                    |G0/0/0           | 10.0.0.1         |  255.255.255.0              | -                         |
|                      |G0/0/1           | 192.168.20.1         |255.255.255.0                |                           |
|                      |G0/0/1.100       | 192.168.30.1         |255.255.255.0                |                           |
|                      |G0/0/1.200       | ---                  |                             |                           |
|                      |G0/0/1.1000      |
|S1                    |VLAN 10          |   192.168.10.11      |  255.255.255.0              | 192.168.10.1              |
|S2                    |VLAN 10          |   192.168.10.12      |  255.255.255.0              | 192.168.10.1              |
|PC-B                  |NIC              |   192.168.20.3       |  255.255.255.0              | 192.168.20.1              |
|PC-A                  |NIC              |   192.168.20.3       |  255.255.255.0              | 192.168.30.1              |


### Таблица VLAN
| VLAN                 |   Имя           | 		Назначенный интерфейс        |
|:---------------------|:---------------:|:----------------------------------|
|10                    |Управление       | S1: VLAN 10; S2: VLAN 10          |
|20                    |Sales            | S1: F0/6                          |
|30                    |Operations       | S2: F0/18                         |
|999                   |Parking_Lot      | С1: F0/2-4, F0/7-24, G0/1-2       |
|                      |                 | С2: F0/2-17, F0/19-24, G0/1-2     |

### Задачи
+ Часть 1. Создание сети и настройка основных параметров устройства
+ Часть 2. Настройка и проверка двух серверов DHCPv4 на R1
+ Часть 3. Настройка и проверка DHCP-ретрансляции на R2


### При первом подключении к устройствам, необходимо провести первоначальную настройку:

+ Задание паролей пользователя
+ Настройка ssh для подключения
+ Задания баннера


Сам перечень набора команд для R1:

```
en
conf t
hostname R1
banner motd ^The device is the property of the company, any unauthorized change to the configuration is punishable by law.^
ip domain-name otus.ru
no ip domain-lookup
enable secret class
username cisco secret class
service password-encryption 
crypto key generate rsa 
2048
ip ssh version 2
username admin privilege 15 secret Adm1nP@55
line vty 0
logging synchronous
exit
line vty 0 4 
login local
transport input ssh
exit
line vty 5 15
login local
transport input ssh
exit
security password min-length 14
exit
wr mem
```

Сам перечень набора команд для R2:

```
en
conf t
hostname R2
banner motd ^The device is the property of the company, any unauthorized change to the configuration is punishable by law.^
ip domain-name otus.ru
no ip domain-lookup
enable secret class
username cisco secret class
service password-encryption 
crypto key generate rsa 
2048
ip ssh version 2
username admin privilege 15 secret Adm1nP@55
line vty 0
logging synchronous
exit
line vty 0 4 
login local
transport input ssh
exit
line vty 5 15
login local
transport input ssh
exit
security password min-length 14
exit
wr mem
```

Сам перечень набора команд для S1:

```
en
conf t
hostname S1
banner motd ^The device is the property of the company, any unauthorized change to the configuration is punishable by law.^
ip domain-name otus.ru
no ip domain-lookup
enable secret class
username cisco secret class
service password-encryption 
crypto key generate rsa 
2048
ip ssh version 2
username admin privilege 15 secret Adm1nP@55
line vty 0
logging synchronous
exit
line vty 0 4 
login local
transport input ssh
exit
line vty 5 15
login local
transport input ssh
exit
security password min-length 14
exit
wr mem
```

Сам перечень набора команд для S2:

```
en
conf t
hostname S2
banner motd ^The device is the property of the company, any unauthorized change to the configuration is punishable by law.^
ip domain-name otus.ru
no ip domain-lookup
enable secret class
username cisco secret class
service password-encryption 
crypto key generate rsa 
2048
ip ssh version 2
username admin privilege 15 secret Adm1nP@55
line vty 0
logging synchronous
exit
line vty 0 4 
login local
transport input ssh
exit
line vty 5 15
login local
transport input ssh
exit
security password min-length 14
exit
wr mem
```

Далее необходимо расчитать сеть и маску для подсетей А (клиентская), В (для управления-MGM) и С (клиентская):

### Расчет подсетей

Подсеть A (58 хостов) - Клиентская VLAN на маршрутизаторе R1:
Одна подсеть «Подсеть A», поддерживающая 58 хостов (клиентская VLAN на R1).
Подсеть A:
Требуется: 58 хостов 
Маска: /26 (255.255.255.192) = 64 адреса
Сеть: 192.168.1.0/26
Первый IP: 192.168.1.1/26 (R1 VLAN G0/0/1.100)
Диапазон: 192.168.1.1 - 192.168.1.62

Подсеть B - Управление на R1
Требуется: 28 хостов
Маска: /27 (255.255.255.224)
Сеть: 192.168.1.64/27
Первый IP: 192.168.1.65/27 (R1 G0/0/1.200)
Второй IP: 192.168.1.66/27 (S1 VLAN 200)
Шлюз по умолчанию: 192.168.1.65

Подсеть C (12 узлов) - Клиентская сеть на R2
Требуется: 12 хостов 
Маска: /28 (255.255.255.240)
Сеть: 192.168.1.96/28
Первый IP: 192.168.1.97/28 (R2 G0/0/1)


### Настройка маршрутизации между VLAN на R1

```
R1#
R1#
R1#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
R1(config)#int
R1(config)#interface gig
R1(config)#interface gigabitEthernet 0/0/1
R1(config-if)#no shu
R1(config-if)#no shutdown 

R1(config-if)#
%LINK-5-CHANGED: Interface GigabitEthernet0/0/1, changed state to up

%LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet0/0/1, changed state to up

R1(config-if)#exit
R1(config)#int
R1(config)#interface gig
R1(config)#interface gigabitEthernet 0/0/1.100
R1(config-subif)#
%LINK-5-CHANGED: Interface GigabitEthernet0/0/1.100, changed state to up

%LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet0/0/1.100, changed state to up

R1(config-subif)#des
R1(config-subif)#description CLIENTS
R1(config-subif)#en
R1(config-subif)#encapsulation d
R1(config-subif)#encapsulation dot1Q 100
R1(config-subif)#ip add
R1(config-subif)#ip address 192.168.1.1 255.255.255.192
R1(config-subif)#exit
R1(config)#int
R1(config)#interface gig
R1(config)#interface gigabitEthernet 0/0/1.200
R1(config-subif)#
%LINK-5-CHANGED: Interface GigabitEthernet0/0/1.200, changed state to up

%LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet0/0/1.200, changed state to up

R1(config-subif)#des
R1(config-subif)#description MANAGMENT(MGM)
R1(config-subif)#en
R1(config-subif)#encapsulation d
R1(config-subif)#encapsulation dot1Q 200
R1(config-subif)#ip add
R1(config-subif)#ip address 192.168.1.65 255.255.255.224
R1(config-subif)#exit
R1(config)#int
R1(config)#interface gig
R1(config)#interface gigabitEthernet 0/0/1.1000
R1(config-subif)#
%LINK-5-CHANGED: Interface GigabitEthernet0/0/1.1000, changed state to up

%LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet0/0/1.1000, changed state to up

R1(config-subif)#en
R1(config-subif)#encapsulation d
R1(config-subif)#encapsulation dot1Q 1000 ?
  native  Make this as native vlan
  <cr>
R1(config-subif)#encapsulation dot1Q 1000 n
R1(config-subif)#encapsulation dot1Q 1000 native 
R1(config-subif)#des
R1(config-subif)#description NATIVE
R1(config-subif)#no i
R1(config-subif)#no ip
R1(config-subif)#exit
R1(config)#int
R1(config)#interface gi
R1(config)#interface gigabitEthernet 0/0/0
R1(config-if)#ip add
R1(config-if)#ip address 10.0.0.1 255.255.255.252
R1(config-if)#no shu
R1(config-if)#no shutdown 

R1(config-if)#
%LINK-5-CHANGED: Interface GigabitEthernet0/0/0, changed state to up

R1(config-if)#do wr  mem
Building configuration...
[OK]
R1(config-if)#
```
















Согласно таблице необходимо создать сети и настроить основные параметры устройства:







### Для R1:
```
R1#
R1#
R1#
R1#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
R1(config)#int
R1(config)#interface gigi
R1(config)#interface giga
R1(config)#interface gigabitEthernet 0/0/0
R1(config-if)#ip add
R1(config-if)#ip address 10.0.0.1 255.255.255.252
R1(config-if)#no shu
R1(config-if)#no shutdown 

R1(config-if)#
%LINK-5-CHANGED: Interface GigabitEthernet0/0/0, changed state to up

R1(config-if)#exit
R1(config)#int
R1(config)#interface gi
R1(config)#interface gigabitEthernet 0/0/1
R1(config-if)#no shu
R1(config-if)#no shutdown 

R1(config-if)#
%LINK-5-CHANGED: Interface GigabitEthernet0/0/1, changed state to up

%LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet0/0/1, changed state to up

R1(config-if)#exit
R1(config)#nt
R1(config)#int
R1(config)#interface gi
R1(config)#interface gigabitEthernet 0/0/1.100
R1(config-subif)#
%LINK-5-CHANGED: Interface GigabitEthernet0/0/1.100, changed state to up

%LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet0/0/1.100, changed state to up

R1(config-subif)#en
R1(config-subif)#encapsulation do
R1(config-subif)#encapsulation dot1Q 100
R1(config-subif)#ip add
R1(config-subif)#ip address 192.168.1.1 255.255.255.0
R1(config-subif)#des
R1(config-subif)#description CLIENTS
R1(config-subif)#
R1(config-subif)#exit
R1(config)#int
R1(config)#interface giga
R1(config)#interface gigabitEthernet 0/0/1.200
R1(config-subif)#
%LINK-5-CHANGED: Interface GigabitEthernet0/0/1.200, changed state to up

%LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet0/0/1.200, changed state to up

R1(config-subif)#en
R1(config-subif)#encapsulation do
R1(config-subif)#encapsulation dot1Q 200
R1(config-subif)#ip add
R1(config-subif)#ip address 192.168.2.1 255.255.255.0
R1(config-subif)#des
R1(config-subif)#description MANAGEMENT(MGM)
R1(config-subif)#exit
R1(config)#int
R1(config)#interface giga
R1(config)#interface gigabitEthernet 0/0/1.1000
R1(config-subif)#
%LINK-5-CHANGED: Interface GigabitEthernet0/0/1.1000, changed state to up

%LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet0/0/1.1000, changed state to up

R1(config-subif)#en
R1(config-subif)#encapsulation do
R1(config-subif)#encapsulation dot1Q 1000 ?
  native  Make this as native vlan
  <cr>
R1(config-subif)#encapsulation dot1Q 1000 na
R1(config-subif)#encapsulation dot1Q 1000 native 
R1(config-subif)#des
R1(config-subif)#description NATIVE
R1(config-subif)#ip add
R1(config-subif)#ip address 192.168.3.1 255.255.255.252
R1(config-subif)#end
R1#
%SYS-5-CONFIG_I: Configured from console by console

R1#
```







![PTK файл enable password: class](./stp.pkt)

