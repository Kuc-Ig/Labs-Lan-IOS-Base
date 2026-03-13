![](./1.png)

### Таблица адресации
| Устройство           |   Интерфейс     | 		IP-адрес/Маска  |   
|:---------------------|:---------------:|:---------------------|
|R1                    |G0/0/1           | 192.168.10.1/24      |  
|                      |Loopback 0       | 10.10.1.1/24         |
|                      |                 |                      |
|                      |                 |                      |
|S1                    |VLAN 10          | 192.168.10.201/24    | 
|S2                    |VLAN 10          | 192.168.10.202/24    |
|                      |                 |                      |  
|                      |                 |                      | 
|PC-А                  |NIC              |DHCP                  |  
|PC-В                  |NIC              |DHCP                  |  


### Задачи
Цели
Часть 1. Настройка основного сетевого устройства
+ Создайте сеть.
+ Настройте маршрутизатор R1.
+ Настройка и проверка основных параметров коммутатора


### При первом подключении к устройствам, необходимо провести первоначальную настройку на всех сетевых устройствах:

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
