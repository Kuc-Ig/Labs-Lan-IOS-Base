# Организация сети офиса с двумя ЦОД

![](./1.jpeg)

С целью реализации сети офиса с двумя ЦОД необходимо запланировать ip адресацию с разделением по VLAN, далее ниже приведена таблица подсетей с указанием VLAN. 
### VLAN и подсети
![](./2.jpeg)

### Стыковочные сети между сетевыми оборудованиями (узлами)
![](./3.jpeg)
Далее необходимо назначить ip адреса для парка серверов. Ниже приведена таблица с адресацией по серверам.
### SERVER NETWORK
![](./4.jpeg)

DNS можно указать 192.168.50.10, 8.8.8.8

### Пояснение:
ADM – отдел системных администраторов;
IB – отдел информационной безопасности (далее ИБ); 
BUH – бухгалтерия; 
SERVER – сеть для парка серверов;
MGMT – отдельная подсеть для администрирования; 
USERS – остальные пользователи или гостевая сеть.
INTERNET – сервер выхода в интернет, может выступать как сервер так и маршрутизатор;
IB-SV-1 – сервер отдела информационной безопасности в ЦОД – 2;
IB-SV-2 – сервер отдела информационной безопасности в ЦОД – 1;
ADM-SV-1 – сервер отдела системных администраторов;
User-SV-1 – сервер для остальных пользователей (или гостевых), который может служить для хранения временных файлом и иных сервисов компании;
«Config:» - конфигурация настроек;
«Ход выполнения настроек:» - процесс конфигурирования оборудования и хода выполнения работы.
Схема организации сети будет выглядеть следующем образом:

![](./5.jpeg)



### Сеть должна отвечать следующим требованиям:

1. User 1 и User 2 должны иметь доступ только Server-PT INTERNET и Server-PT User-SV-1
2. ADM-1, ADM-2, ADM-3 должны иметь доступ ко всем системам кроме  IB-SV-1 (2)
3. IB-1, IB-2, IB-3 должны иметь доступ  только к IB-SV-1 (2)
4. На всех сетевых устройствах должен быть установлен логин - adm и пароль - gfhjkm и открыть доступ по ssh для ADM-1, ADM-2, ADM-3 для остальных доступ по ssh должен быть закрыт
5. На всех сетевых устройствах неиспользуемые порты должны быть выключены
6. Между свитчами SW-1, SW-2, SW-3, должен быть настроен протокол osfp
7. BUH-1, BUH-2 должны иметь доступ только на User-SV-1, а BUH-3 только на User-SV-1 и Server-INTERNET только протоколу 80
8. User-1, User-2 не должны иметь сетевого взаимодействия с BUH-1, BUH-2 и BUH-3 но ADM-1, ADM-2, ADM-3, IB-1, IB-2, IB-3 должны иметь сетевое взаимодействие с User-1, User-2, BUH-1, BUH-2 и BUH-3.
9. Роутер R1-Office - является dhcp сервером для User-1, User-2, ADM-1, ADM-2, ADM-3, IB-1, IB-2, IB-3, BUH-1, BUH-2, BUH-3

Первым делом необходимо настроить сеть офиса, назначить на коммутаторе  VLAN:



### Настройка SW Office
## Создание VLAN

## Config:
```
configure terminal
hostname SW-Office
vlan 10
name USERS
vlan 20
name ADM
vlan 30
name IB
vlan 40
name BUH
vlan 50
name SERVERS
vlan 99
name MGMT
```
## Ход выполнения настроек:
```
Switch(config)#host
Switch(config)#hostname SW-Office
SW-Office(config)#vla
SW-Office(config)#vlan 10
SW-Office(config-vlan)#na
SW-Office(config-vlan)#name USERS
SW-Office(config-vlan)#exit
SW-Office(config)#vla
SW-Office(config)#vlan 20
SW-Office(config-vlan)#na
SW-Office(config-vlan)#name ADM
SW-Office(config-vlan)#exit
SW-Office(config)#vla
SW-Office(config)#vlan 30
SW-Office(config-vlan)#name IB
SW-Office(config-vlan)#exit
SW-Office(config)#vla
SW-Office(config)#vlan 40
SW-Office(config-vlan)#BUH
^
% Invalid input detected at '^' marker.
SW-Office(config-vlan)#name BUH
SW-Office(config-vlan)#exit
SW-Office(config)#vlan 50
SW-Office(config-vlan)#name SERVERS
SW-Office(config-vlan)#exit
SW-Office(config)#vlan
SW-Office(config)#vlan 99
SW-Office(config-vlan)#name MGMT
SW-Office(config-vlan)#exit
SW-Office(config)#
```


### Назначение портов
## Config:
### USERS
```
interface range fa0/1-2
switchport mode access
switchport access vlan 10
spanning-tree portfast
```

## Ход выполнения настроек:
```
SW-Office#conf t
Enter configuration commands, one per line. End with CNTL/Z.
SW-Office(config)#int
SW-Office(config)#interface ra
SW-Office(config)#interface range fa0/1-2
SW-Office(config-if-range)#swi
SW-Office(config-if-range)#switchport mo
SW-Office(config-if-range)#switchport mode acc
SW-Office(config-if-range)#switchport mode access 
SW-Office(config-if-range)#swi
SW-Office(config-if-range)#switchport acc
SW-Office(config-if-range)#switchport access vlan 10
SW-Office(config-if-range)#sp
SW-Office(config-if-range)#spa
SW-Office(config-if-range)#spanning-tree po
SW-Office(config-if-range)#spanning-tree portfast 
%Warning: portfast should only be enabled on ports connected to a single
host. Connecting hubs, concentrators, switches, bridges, etc... to this
interface when portfast is enabled, can cause temporary bridging loops.
Use with CAUTION

%Portfast has been configured on FastEthernet0/1 but will only
have effect when the interface is in a non-trunking mode.
%Warning: portfast should only be enabled on ports connected to a single
host. Connecting hubs, concentrators, switches, bridges, etc... to this
interface when portfast is enabled, can cause temporary bridging loops.
Use with CAUTION

%Portfast has been configured on FastEthernet0/2 but will only
have effect when the interface is in a non-trunking mode.
```
## Config
### ADM
```
interface range fa0/3-5
switchport mode access
switchport access vlan 20
spanning-tree portfast
```
## Ход выполнения настроек:
```
SW-Office#conf t
Enter configuration commands, one per line. End with CNTL/Z.
SW-Office(config)#int
SW-Office(config)#interface ra
SW-Office(config)#interface range fa0/3-5
SW-Office(config-if-range)#swi
SW-Office(config-if-range)#switchport mo
SW-Office(config-if-range)#switchport mode acc
SW-Office(config-if-range)#switchport mode access 
SW-Office(config-if-range)#swi
SW-Office(config-if-range)#switchport acc
SW-Office(config-if-range)#switchport access vl
SW-Office(config-if-range)#switchport access vlan 20
SW-Office(config-if-range)#sp
SW-Office(config-if-range)#spa
SW-Office(config-if-range)#spanning-tree port
SW-Office(config-if-range)#spanning-tree portfast 
%Warning: portfast should only be enabled on ports connected to a single
host. Connecting hubs, concentrators, switches, bridges, etc... to this
interface when portfast is enabled, can cause temporary bridging loops.
Use with CAUTION

%Portfast has been configured on FastEthernet0/3 but will only
have effect when the interface is in a non-trunking mode.
%Warning: portfast should only be enabled on ports connected to a single
host. Connecting hubs, concentrators, switches, bridges, etc... to this
interface when portfast is enabled, can cause temporary bridging loops.
Use with CAUTION

%Portfast has been configured on FastEthernet0/4 but will only
have effect when the interface is in a non-trunking mode.
%Warning: portfast should only be enabled on ports connected to a single
host. Connecting hubs, concentrators, switches, bridges, etc... to this
interface when portfast is enabled, can cause temporary bridging loops.
Use with CAUTION

%Portfast has been configured on FastEthernet0/5 but will only
have effect when the interface is in a non-trunking mode.
```
## Config:
### IB

```
interface range fa0/6-8
switchport mode access
switchport access vlan 30
spanning-tree portfast
```
## Ход выполнения настроек:
```
SW-Office#
SW-Office#conf t
Enter configuration commands, one per line. End with CNTL/Z.
SW-Office(config)#int
SW-Office(config)#interface ra
SW-Office(config)#interface range fa0/6-8
SW-Office(config-if-range)#
SW-Office(config-if-range)#swi
SW-Office(config-if-range)#switchport mo
SW-Office(config-if-range)#switchport mode acc
SW-Office(config-if-range)#switchport mode access 
SW-Office(config-if-range)#swi
SW-Office(config-if-range)#switchport acc
SW-Office(config-if-range)#switchport access vl
SW-Office(config-if-range)#switchport access vlan 30
SW-Office(config-if-range)#spa
SW-Office(config-if-range)#spanning-tree po
SW-Office(config-if-range)#spanning-tree portfast 
%Warning: portfast should only be enabled on ports connected to a single
host. Connecting hubs, concentrators, switches, bridges, etc... to this
interface when portfast is enabled, can cause temporary bridging loops.
Use with CAUTION

%Portfast has been configured on FastEthernet0/6 but will only
have effect when the interface is in a non-trunking mode.
%Warning: portfast should only be enabled on ports connected to a single
host. Connecting hubs, concentrators, switches, bridges, etc... to this
interface when portfast is enabled, can cause temporary bridging loops.
Use with CAUTION

%Portfast has been configured on FastEthernet0/7 but will only
have effect when the interface is in a non-trunking mode.
%Warning: portfast should only be enabled on ports connected to a single
host. Connecting hubs, concentrators, switches, bridges, etc... to this
interface when portfast is enabled, can cause temporary bridging loops.
Use with CAUTION

%Portfast has been configured on FastEthernet0/8 but will only
have effect when the interface is in a non-trunking mode.
```
## Config:
## BUH
```
interface range fa0/9-11
switchport mode access
switchport access vlan 40
spanning-tree portfast

```
Ход выполнения настроек:

```
SW-Office(config)#int
SW-Office(config)#interface ra
SW-Office(config)#interface range fa0/9-11
SW-Office(config-if-range)#swi
SW-Office(config-if-range)#switchport m
SW-Office(config-if-range)#switchport mode acc
SW-Office(config-if-range)#switchport mode access 
SW-Office(config-if-range)#swi
SW-Office(config-if-range)#switchport acc
SW-Office(config-if-range)#switchport access vl
SW-Office(config-if-range)#switchport access vlan 40
SW-Office(config-if-range)#spa
SW-Office(config-if-range)#spanning-tree po
SW-Office(config-if-range)#spanning-tree portfast 
%Warning: portfast should only be enabled on ports connected to a single
host. Connecting hubs, concentrators, switches, bridges, etc... to this
interface when portfast is enabled, can cause temporary bridging loops.
Use with CAUTION

%Portfast has been configured on FastEthernet0/9 but will only
have effect when the interface is in a non-trunking mode.
%Warning: portfast should only be enabled on ports connected to a single
host. Connecting hubs, concentrators, switches, bridges, etc... to this
interface when portfast is enabled, can cause temporary bridging loops.
Use with CAUTION

%Portfast has been configured on FastEthernet0/10 but will only
have effect when the interface is in a non-trunking mode.
%Warning: portfast should only be enabled on ports connected to a single
host. Connecting hubs, concentrators, switches, bridges, etc... to this
interface when portfast is enabled, can cause temporary bridging loops.
Use with CAUTION

%Portfast has been configured on FastEthernet0/11 but will only
have effect when the interface is in a non-trunking mode.
SW-Office(config-if-range)#
```
## Транк к R1-Office
## Config:

```
interface giga0/24

switchport mode trunk
switchport trunk allowed vlan 10,20,30,40,50,99
```
Ход выполнения настроек:
```
SW-Office(config)#
SW-Office(config)#int
SW-Office(config)#interface giga0/1
SW-Office(config-if)#swi
SW-Office(config-if)#switchport tr
SW-Office(config-if)#switchport trunk ?
allowed Set allowed VLAN characteristics when interface is in trunking mode
native Set trunking native characteristics when interface is in trunking
mode
SW-Office(config-if)#switchport mode trunk 
SW-Office(config-if)#switchport trunk all
SW-Office(config-if)#switchport trunk allowed vl
SW-Office(config-if)#switchport trunk allowed vlan 10?
WORD 
SW-Office(config-if)#switchport trunk allowed vlan 10,20,30,40,50,99
SW-Office(config-if)#
```
## Management IP
## Config:
```
interface vlan 99
ip address 192.168.99.10 255.255.255.0
no shutdown
ip default-gateway 192.168.99.1
```

Ход выполнения настроек:
```
SW-Office#
SW-Office#conf t
Enter configuration commands, one per line. End with CNTL/Z.
SW-Office(config)#int
SW-Office(config)#interface vl
SW-Office(config)#interface vlan 99
SW-Office(config-if)#
%LINK-5-CHANGED: Interface Vlan99, changed state to up

SW-Office(config-if)#ip add
SW-Office(config-if)#ip address 192.168.99.10 255.255.255.0
SW-Office(config-if)#no shu
SW-Office(config-if)#no shutdown 
SW-Office(config-if)#ip de
SW-Office(config-if)#ip dee
SW-Office(config-if)#ip def
SW-Office(config-if)#exit
SW-Office(config)#ip def
SW-Office(config)#ip default-gateway 192.168.99.1
SW-Office(config)#exit
SW-Office#
```
## SSH 
## Config:
```
ip domain-name corp.local
crypto key generate rsa
1024

username adm privilege 15 secret gfhjkm

line vty 0 15
login local
transport input ssh
access-class 10 in

ip access-list standard 10
permit 192.168.20.0 0.0.0.255
```
Ход выполнения настроек:
```
SW-Office#
SW-Office#conf t
Enter configuration commands, one per line. End with CNTL/Z.
SW-Office(config)#ip do
SW-Office(config)#ip dom
SW-Office(config)#ip ?
access-list Named access-list
arp IP ARP global configuration
default-gateway Specify default gateway (if not routing IP)
dhcp Configure DHCP server and relay parameters
domain IP DNS Resolver
domain-lookup Enable IP Domain Name System hostname translation
domain-name Define the default domain name
ftp FTP configuration commands
host Add an entry to the ip hostname table
name-server Specify address of name server to use
scp Scp commands
ssh Configure ssh options
SW-Office(config)#ip domain-name corp.local
SW-Office(config)#cry
SW-Office(config)#crypto k
SW-Office(config)#crypto key get
SW-Office(config)#crypto key gen
SW-Office(config)#crypto key generate rsa
The name for the keys will be: SW-Office.corp.local
Choose the size of the key modulus in the range of 360 to 2048 for your
General Purpose Keys. Choosing a key modulus greater than 512 may take
a few minutes.

How many bits in the modulus [512]: 1024
% Generating 1024 bit RSA keys, keys will be non-exportable...[OK]

SW-Office(config)#user
*Mar 1 1:14:33.646: %SSH-5-ENABLED: SSH 1.99 has been enabled
SW-Office(config)#username adm pri
SW-Office(config)#username adm privilege 15 sec
SW-Office(config)#username adm privilege 15 secret gfhjkm
SW-Office(config)#lin
SW-Office(config)#line vty 0 15
SW-Office(config-line)#lo
SW-Office(config-line)#log
SW-Office(config-line)#login local
SW-Office(config-line)#tr
SW-Office(config-line)#transport in
SW-Office(config-line)#transport input ssh
SW-Office(config-line)#acc
SW-Office(config-line)#acce
SW-Office(config-line)#access-class 10 in
SW-Office(config-line)#exit
SW-Office(config)#ip acc
SW-Office(config)#ip access-list sta
SW-Office(config)#ip access-list standard 10
SW-Office(config-std-nacl)#rerm
SW-Office(config-std-nacl)#per
SW-Office(config-std-nacl)#permit 192.168.20.0 0.0.0.255
SW-Office(config-std-nacl)#exit
SW-Office(config)#do wr mem
Building configuration...
[OK]
SW-Office(config)#
```
## NTP SERVER
## Config:
```
ntp master 1

```
Ход выполнения настроек:
```
SW-Office(config)#nt
SW-Office(config)#ntp ma
SW-Office(config)#ntp master 1
SW-Office(config)#
```

## Отключение неиспользуемых портов
## Config:
```
interface range fa0/12-23,g0/1-2
shutdown
```
Ход выполнения настроек:

```
SW-Office(config)#
SW-Office(config)#int
SW-Office(config)#interface ra
SW-Office(config)#interface range fa0/12-23, g0/1-2
SW-Office(config-if-range)#shu
SW-Office(config-if-range)#shutdown 

%LINK-5-CHANGED: Interface FastEthernet0/12, changed state to administratively down

%LINK-5-CHANGED: Interface FastEthernet0/13, changed state to administratively down

%LINK-5-CHANGED: Interface FastEthernet0/14, changed state to administratively down

%LINK-5-CHANGED: Interface FastEthernet0/15, changed state to administratively down

%LINK-5-CHANGED: Interface FastEthernet0/16, changed state to administratively down

%LINK-5-CHANGED: Interface FastEthernet0/17, changed state to administratively down

%LINK-5-CHANGED: Interface FastEthernet0/18, changed state to administratively down

%LINK-5-CHANGED: Interface FastEthernet0/19, changed state to administratively down

%LINK-5-CHANGED: Interface FastEthernet0/20, changed state to administratively down

%LINK-5-CHANGED: Interface FastEthernet0/21, changed state to administratively down

%LINK-5-CHANGED: Interface FastEthernet0/22, changed state to administratively down

%LINK-5-CHANGED: Interface FastEthernet0/23, changed state to administratively down

%LINK-5-CHANGED: Interface GigabitEthernet0/1, changed state to administratively down

%LINK-5-CHANGED: Interface GigabitEthernet0/2, changed state to administratively down
SW-Office(config-if-range)#do wr mem
Building configuration...
[OK]
SW-Office(config-if-range)#
```
## Настройка R1-Office
## Config:
```
hostname R1-Office

ip domain-name corp.local
crypto key generate rsa
1024

username adm privilege 15 secret gfhjkm

line vty 0 15
login local
transport input ssh
access-class 10 in

ip access-list standard 10
permit 192.168.20.0 0.0.0.255
```
Ход выполнения настроек:
```
Router>en
Router#conf t
Enter configuration commands, one per line. End with CNTL/Z.
Router(config)#hos
Router(config)#hostname R1-Office
R1-Office(config)#cr
R1-Office(config)#ip do
R1-Office(config)#ip domain-name corp.local
R1-Office(config)#cry
R1-Office(config)#crypto k
R1-Office(config)#crypto key gen
R1-Office(config)#crypto key generate rsa
The name for the keys will be: R1-Office.corp.local
Choose the size of the key modulus in the range of 360 to 2048 for your
General Purpose Keys. Choosing a key modulus greater than 512 may take
a few minutes.

How many bits in the modulus [512]: 1024
% Generating 1024 bit RSA keys, keys will be non-exportable...[OK]

R1-Office(config)#usern
*Mar 1 2:5:36.163: %SSH-5-ENABLED: SSH 1.99 has been enabled
R1-Office(config)#username adm pri
R1-Office(config)#username adm privilege 15 se
R1-Office(config)#username adm privilege 15 secret gfhjkm
R1-Office(config)#lin
R1-Office(config)#line vty 0 15
R1-Office(config-line)#login local
R1-Office(config-line)#tra
R1-Office(config-line)#transport in
R1-Office(config-line)#transport input ssh
R1-Office(config-line)#acc
R1-Office(config-line)#acce
R1-Office(config-line)#access-class 10 in
R1-Office(config-line)#exit
R1-Office(config)#ip acc
R1-Office(config)#ip access-list sta
R1-Office(config)#ip access-list standard 10
R1-Office(config-std-nacl)#perm
R1-Office(config-std-nacl)#permit 192.168.20.0 0.0.0.255
R1-Office(config-std-nacl)#do wr mem
Building configuration...
[OK]
R1-Office(config-std-nacl)#
```

## Далее настраивам VLAN
```
interface g0/0/0
no shutdown

```
Ход выполнения
```
R1-Office(config)#int
R1-Office(config)#interface g0/0/0
R1-Office(config-if)#no shu
R1-Office(config-if)#no shutdown 

R1-Office(config-if)#
%LINK-5-CHANGED: Interface GigabitEthernet0/0/0, changed state to up

%LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet0/0/0, changed state to up

R1-Office(config-if)#
```
### VLAN 10 USERS
```
interface g0/0/0.10
encapsulation dot1Q 10
ip address 192.168.10.1 255.255.255.0
```
Ход работы
```
R1-Office(config)#
R1-Office(config)#int
R1-Office(config)#interface g0/0/0.10
R1-Office(config-subif)#
%LINK-5-CHANGED: Interface GigabitEthernet0/0/0.10, changed state to up

%LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet0/0/0.10, changed state to up

R1-Office(config-subif)#en
R1-Office(config-subif)#encapsulation do
R1-Office(config-subif)#encapsulation dot1Q 10
R1-Office(config-subif)#ipadd
R1-Office(config-subif)#ip add
R1-Office(config-subif)#ip address 192.168.10.1 255.255.255.0
R1-Office(config-subif)#exit
R1-Office(config)#
```
##  VLAN 20 ADM
```
interface g0/0/0.20
encapsulation dot1Q 20
ip address 192.168.20.1 255.255.255.0
```
```
R1-Office(config)#
R1-Office(config)#interface g0/0/0.20
R1-Office(config-subif)#encapsulation dot1Q 20
R1-Office(config-subif)#ip address 192.168.20.1 255.255.255.0
%LINK-5-CHANGED: Interface GigabitEthernet0/0/0.20, changed state to up

%LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet0/0/0.20, changed state to up

R1-Office(config-subif)#
R1-Office(config-subif)#exit
R1-Office(config)#
```
## VLAN 30 IB
```
interface g0/0/0.30
encapsulation dot1Q 30
ip address 192.168.30.1 255.255.255.0
```
```
R1-Office(config)#
R1-Office(config)#interface g0/0/0.30
R1-Office(config-subif)#
R1-Office(config-subif)#encapsulation dot1Q 30
R1-Office(config-subif)#
R1-Office(config-subif)#ip address 192.168.30.1 255.255.255.0
R1-Office(config-subif)#
R1-Office(config-subif)#
%LINK-5-CHANGED: Interface GigabitEthernet0/0/0.30, changed state to up

%LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet0/0/0.30, changed state to up

R1-Office(config-subif)#exit
R1-Office(config)#
```































