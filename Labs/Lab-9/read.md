![](./1.png)

### Таблица адресации
| Устройство           |   Интерфейс     | 		IPv6-адрес      |   
|:---------------------|:---------------:|:---------------------|
|R1                    |G0/0/0           | 2001:db8:acad:2::1/64|  
|                      |                 | fe80::1              |
|                      |G0/0/1           | 2001:db8:acad:1::1/64|
|                      |                 | fe80::1              |
|R2                    |G0/0/0           | 2001:db8:acad:2::2/64|                                                
|                      |                 |fe80::2               |
|                      |G0/0/1           |2001:db8:acad:3::1/64 |  
|                      |                 |fe80::1               | 
|PC-B                  |NIC              |DHCP                  |  
|PC-A                  |NIC              |DHCP                  |  


### Задачи
+ Часть 1. Создание сети и настройка основных параметров устройства
+ Часть 2. Проверка назначения адреса SLAAC от R1
+ Часть 3. Настройка и проверка сервера DHCPv6 без гражданства на R1
+ Часть 4. Настройка и проверка состояния DHCPv6 сервера на R1
+ Часть 5. Настройка и проверка DHCPv6 Relay на R2

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
Далее необходимо настроить на маршрутизаторе R1 согласно таблице ip6 адрес на интерфейсе и задать маршрут по умолчению:

```
R1#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
R1(config)#ipv6 ?
  access-list      Configure access lists
  cef              Cisco Express Forwarding
  dhcp             Configure Ipv6 DHCP
  general-prefix   Configure a general IPv6 prefix
  host             Configure static hostnames
  inspect          Context-based Access Control Engine
  local            Specify local options
  nat              NAT-PT Configuration commands
  neighbor         Neighbor
  route            Configure static routes
  router           Enable an IPV6 routing process
  unicast-routing  Enable unicast routing
R1(config)#ipv6 un
R1(config)#ipv6 unicast-routing 
R1(config)#int giga0/0/0
R1(config-if)#ipv6 add
R1(config-if)#ipv6 address 2001:db8:acad:2::1/64
R1(config-if)#ipv6 add
R1(config-if)#ipv6 address fe80::1 link-local
R1(config-if)#no shu
R1(config-if)#no shutdown 

R1(config-if)#
%LINK-5-CHANGED: Interface GigabitEthernet0/0/0, changed state to up

R1(config-if)#exit
R1(config)#int
R1(config)#interface gi
R1(config)#interface gigabitEthernet 0/0/1
R1(config-if)#ipv6 add
R1(config-if)#ipv6 address 2001:db8:acad:1::1/64
R1(config-if)#ipv6 address fe80::1 link-local
R1(config-if)#no shu
R1(config-if)#no shutdown 

R1(config-if)#
%LINK-5-CHANGED: Interface GigabitEthernet0/0/1, changed state to up

%LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet0/0/1, changed state to up

R1(config-if)#exit
R1(config)#ipv6 ?
  access-list      Configure access lists
  cef              Cisco Express Forwarding
  dhcp             Configure Ipv6 DHCP
  general-prefix   Configure a general IPv6 prefix
  host             Configure static hostnames
  inspect          Context-based Access Control Engine
  local            Specify local options
  nat              NAT-PT Configuration commands
  neighbor         Neighbor
  route            Configure static routes
  router           Enable an IPV6 routing process
  unicast-routing  Enable unicast routing
R1(config)#ipv6 r
R1(config)#ipv6 ro
R1(config)#ipv6 rou
R1(config)#ipv6 route ::/0 2001:db8:acad:2::2
R1(config)#do wr mem
Building configuration...
[OK]
```

По аналогии настраиваем маршрутизатор R2:

```
ipv6 unicast-routing
interface g0/0/0
ipv6 address 2001:db8:acad:2::2/64
ipv6 address fe80::2 link-local
no shutdown
exit
interface g0/0/1
ipv6 address 2001:db8:acad:3::1/64
ipv6 address fe80::1 link-local
no shutdown
exit
ipv6 route ::/0 2001:db8:acad:2::1
```

Проверяем сетевую связанность с R1:

![](./2.png)


Проверяем SLAAC:

![](./3.png)

Как видно на PC-A был назначен ipv6 адрес используя информацию, полученную от маршрутизатора R1

### Шаг 2. Настройте R1 для предоставления DHCPv6 без состояния для PC-A

Для этого необходимо задать имя пула DHCP и прописать доменное имя и диапазон адресов исходя из маски:
Настройки будем проводить на R1:

```
R1#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
R1(config)#ipv6
R1(config)#ipv6 dh
R1(config)#ipv6 dhcp poo
R1(config)#ipv6 dhcp pool R1-STATELESS
R1(config-dhcpv6)#?
  address            IPv6 address allocation
  dns-server         DNS servers
  domain-name        Domain name to complete unqualified host names
  exit               Exit from DHCPv6 configuration mode
  no                 Negate a command or set its defaults
  prefix-delegation  IPv6 prefix delegation
R1(config-dhcpv6)#dn
R1(config-dhcpv6)#dns-server 2001:db8:acad::254
R1(config-dhcpv6)#dom
R1(config-dhcpv6)#domain-name STATELESS.com
R1(config-dhcpv6)#exit
R1(config)#int
R1(config)#interface gig
R1(config)#interface gigabitEthernet 0/0/1
R1(config-if)#ipv6 ?
  address          Configure IPv6 address on interface
  authentication   authentication subcommands
  crypto           Encryption/Decryption commands
  dhcp             IPv6 DHCP interface subcommands
  eigrp            Configure EIGRP IPv6 on interface
  enable           Enable IPv6 on interface
  flow             NetFlow Related commands
  hello-interval   Configures IP-EIGRP hello interval
  inspect          Apply inspect name
  mtu              Set IPv6 Maximum Transmission Unit
  nat              Enable IPv6 NAT on interface
  nd               IPv6 interface Neighbor Discovery subcommands
  ospf             OSPF interface commands
  rip              Configure RIP routing protocol
  summary-address  Summary prefix
  traffic-filter   Access control list for packets
  unnumbered       Preferred interface for source address selection
R1(config-if)#ipv6 n
R1(config-if)#ipv6 nd
R1(config-if)#ipv6 nd ?
  managed-config-flag  Hosts should use DHCP for address config
  other-config-flag    Other stateful configuration flag
  ra                   Router Advertisement control
R1(config-if)#ipv6 nd ot
R1(config-if)#ipv6 nd other-config-flag 
R1(config-if)#ipv6 dh
R1(config-if)#ipv6 ?
  address          Configure IPv6 address on interface
  authentication   authentication subcommands
  crypto           Encryption/Decryption commands
  dhcp             IPv6 DHCP interface subcommands
  eigrp            Configure EIGRP IPv6 on interface
  enable           Enable IPv6 on interface
  flow             NetFlow Related commands
  hello-interval   Configures IP-EIGRP hello interval
  inspect          Apply inspect name
  mtu              Set IPv6 Maximum Transmission Unit
  nat              Enable IPv6 NAT on interface
  nd               IPv6 interface Neighbor Discovery subcommands
  ospf             OSPF interface commands
  rip              Configure RIP routing protocol
  summary-address  Summary prefix
  traffic-filter   Access control list for packets
  unnumbered       Preferred interface for source address selection
R1(config-if)#ipv6 dh
R1(config-if)#ipv6 dhcp ?
  client  Act as an IPv6 DHCP client
  server  Act as an IPv6 DHCP server
R1(config-if)#ipv6 dhcp se
R1(config-if)#ipv6 dhcp server ?
  WORD  Name of IPv6 DHCP pool
R1(config-if)#ipv6 dhcp server R1-STATELESS
R1(config-if)#do wr mem
Building configuration...
[OK]
R1(config-if)#end
R1#
```




============================================================================






============================================================================














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

Проверяем настройки:

```
R1#show ip int
R1#show ip interface ?
  Dialer            Dialer interface
  Dot11Radio        Dot11 interface
  Ethernet          IEEE 802.3
  FastEthernet      FastEthernet IEEE 802.3
  GigabitEthernet   GigabitEthernet IEEE 802.3z
  Loopback          Loopback interface
  Serial            Serial
  Tunnel            Tunnel interface
  Virtual-Access    Virtual Access interface
  Virtual-Template  Virtual Template interface
  Vlan              Catalyst Vlans
  brief             Brief summary of IP status and configuration
  |                 Output Modifiers
  <cr>
R1#show ip interface br
R1#show ip interface brief 
Interface              IP-Address      OK? Method Status                Protocol 
GigabitEthernet0/0/0   10.0.0.1        YES manual up                    down 
GigabitEthernet0/0/1   unassigned      YES unset  up                    up 
GigabitEthernet0/0/1.100192.168.1.1     YES manual up                    up 
GigabitEthernet0/0/1.200192.168.1.65    YES manual up                    up 
GigabitEthernet0/0/1.1000unassigned      YES unset  up                    up 
GigabitEthernet0/0/2   unassigned      YES unset  administratively down down 
Vlan1                  unassigned      YES unset  administratively down down
```

### Далее необходимо прописать соответсвующие VLAN на коммутаторе S1:

```
S1#
S1#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
S1(config)#vla
S1(config)#vlan 100
S1(config-vlan)#n
S1(config-vlan)#na
S1(config-vlan)#name CLIENTS
S1(config-vlan)#exit
S1(config)#vla
S1(config)#vlan 200
S1(config-vlan)#na
S1(config-vlan)#name MGM
S1(config-vlan)#exit
S1(config)#vla
S1(config)#vlan 999
S1(config-vlan)#nam
S1(config-vlan)#name Parking-Lot
S1(config-vlan)#exit
S1(config)#vla
S1(config)#vlan 1000
S1(config-vlan)#na
S1(config-vlan)#name Native
S1(config-vlan)#exit
S1(config)#
```
Проверяем созданные VLAN:

```
S1#sh
S1#show vl
S1#show vlan 

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Fa0/1, Fa0/2, Fa0/3, Fa0/4
                                                Fa0/5, Fa0/6, Fa0/7, Fa0/8
                                                Fa0/9, Fa0/10, Fa0/11, Fa0/12
                                                Fa0/13, Fa0/14, Fa0/15, Fa0/16
                                                Fa0/17, Fa0/18, Fa0/19, Fa0/20
                                                Fa0/21, Fa0/22, Fa0/23, Fa0/24
                                                Gig0/1, Gig0/2
100  CLIENTS                          active    
200  MGM                              active    
999  Parking-Lot                      active    
1000 Native                           active    
1002 fddi-default                     active    
1003 token-ring-default               active    
1004 fddinet-default                  active    
1005 trnet-default                    active    

VLAN Type  SAID       MTU   Parent RingNo BridgeNo Stp  BrdgMode Trans1 Trans2
---- ----- ---------- ----- ------ ------ -------- ---- -------- ------ ------
1    enet  100001     1500  -      -      -        -    -        0      0
100  enet  100100     1500  -      -      -        -    -        0      0
200  enet  100200     1500  -      -      -        -    -        0      0
999  enet  100999     1500  -      -      -        -    -        0      0
1000 enet  101000     1500  -      -      -        -    -        0      0
1002 fddi  101002     1500  -      -      -        -    -        0      0   
1003 tr    101003     1500  -      -      -        -    -        0      0   
1004 fdnet 101004     1500  -      -      -        ieee -        0      0   
1005 trnet 101005     1500  -      -      -        ibm  -        0      0   

VLAN Type  SAID       MTU   Parent RingNo BridgeNo Stp  BrdgMode Trans1 Trans2
---- ----- ---------- ----- ------ ------ -------- ---- -------- ------ ------

Remote SPAN VLANs
------------------------------------------------------------------------------

Primary Secondary Type              Ports
------- --------- ----------------- ------------------------------------------
S1#
```
Далее необходимо прописать VLAn 999 для портов:

```
S1#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
S1(config)#int
S1(config)#interface vl
S1(config)#interface vlan 200
S1(config-if)#
%LINK-5-CHANGED: Interface Vlan200, changed state to up

S1(config-if)#ip add
S1(config-if)#ip address 192.168.1.66 255.255.255.224
S1(config-if)#no sh
S1(config-if)#no shutdown 
S1(config-if)#ip def
S1(config-if)#ip ?
  address         Set the IP address of an interface
  helper-address  Specify a destination address for UDP broadcasts
S1(config-if)#exit
S1(config)#int
S1(config)#interface ra
S1(config)#interface range fa
S1(config)#interface range fastEthernet 0/1-4, f0/7-24, g0/1-2
S1(config-if-range)#sw
S1(config-if-range)#switchport m
S1(config-if-range)#switchport mode acc
S1(config-if-range)#switchport mode access 
S1(config-if-range)#swi
S1(config-if-range)#switchport acc
S1(config-if-range)#switchport access vl
S1(config-if-range)#switchport access vlan 999
S1(config-if-range)#no sh
S1(config-if-range)#no shutdown 
S1(config-if-range)#end
S1#
%SYS-5-CONFIG_I: Configured from console by console

S1#
```
Проверяем внесенные изменения:

```
S1#show vl
S1#show vlan 

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Fa0/5, Fa0/6
100  CLIENTS                          active    
200  MGM                              active    
999  Parking-Lot                      active    Fa0/1, Fa0/2, Fa0/3, Fa0/4
                                                Fa0/7, Fa0/8, Fa0/9, Fa0/10
                                                Fa0/11, Fa0/12, Fa0/13, Fa0/14
                                                Fa0/15, Fa0/16, Fa0/17, Fa0/18
                                                Fa0/19, Fa0/20, Fa0/21, Fa0/22
                                                Fa0/23, Fa0/24, Gig0/1, Gig0/2
1000 Native                           active    
1002 fddi-default                     active    
1003 token-ring-default               active    
1004 fddinet-default                  active    
1005 trnet-default                    active    

VLAN Type  SAID       MTU   Parent RingNo BridgeNo Stp  BrdgMode Trans1 Trans2
---- ----- ---------- ----- ------ ------ -------- ---- -------- ------ ------
1    enet  100001     1500  -      -      -        -    -        0      0
100  enet  100100     1500  -      -      -        -    -        0      0
200  enet  100200     1500  -      -      -        -    -        0      0
999  enet  100999     1500  -      -      -        -    -        0      0
1000 enet  101000     1500  -      -      -        -    -        0      0
1002 fddi  101002     1500  -      -      -        -    -        0      0   
1003 tr    101003     1500  -      -      -        -    -        0      0   
1004 fdnet 101004     1500  -      -      -        ieee -        0      0   
1005 trnet 101005     1500  -      -      -        ibm  -        0      0   

VLAN Type  SAID       MTU   Parent RingNo BridgeNo Stp  BrdgMode Trans1 Trans2
---- ----- ---------- ----- ------ ------ -------- ---- -------- ------ ------

Remote SPAN VLANs
------------------------------------------------------------------------------

Primary Secondary Type              Ports
------- --------- ----------------- ------------------------------------------
S1#
```

Произведем настройку маршрутизатора R2:

```
R2>
R2>
R2>en
Password: 
R2#int
R2#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
R2(config)#int
R2(config)#interface gig
R2(config)#interface gigabitEthernet 0/0/1
R2(config-if)#ip add
R2(config-if)#ip address 192.168.1.97 255.255.255.240
R2(config-if)#no shu
R2(config-if)#no shutdown 

R2(config-if)#
%LINK-5-CHANGED: Interface GigabitEthernet0/0/1, changed state to up

%LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet0/0/1, changed state to up

R2(config-if)#exit
R2(config)#int
R2(config)#interface gig
R2(config)#interface gigabitEthernet 0/0/0
R2(config-if)#ip add
R2(config-if)#ip address 10.0.0.2 255.255.255.252
R2(config-if)#no shu
R2(config-if)#no shutdown 

R2(config-if)#
%LINK-5-CHANGED: Interface GigabitEthernet0/0/0, changed state to up

%LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet0/0/0, changed state to up

R2(config-if)#exit
R2(config)#ip rou
R2(config)#ip ro
R2(config)#?
Configure commands:
  aaa                Authentication, Authorization and Accounting.
  access-list        Add an access list entry
  banner             Define a login banner
  bba-group          Configure BBA Group
  boot               Modify system boot parameters
  cdp                Global CDP configuration subcommands
  class-map          Configure Class Map
  clock              Configure time-of-day clock
  config-register    Define the configuration register
  crypto             Encryption module
  default            Set a command to its defaults
  do                 To run exec commands in config mode
  dot11              IEEE 802.11 config commands
  enable             Modify enable password parameters
  end                Exit from configure mode
  exit               Exit from configure mode
  flow               Global Flow configuration subcommands
  hostname           Set system's network name
  interface          Select an interface to configure
  ip                 Global IP configuration subcommands
  ipv6               Global IPv6 configuration commands
  key                Key management
  license            Configure license features
  line               Configure a terminal line
  lldp               Global LLDP configuration subcommands
  logging            Modify message logging facilities
  login              Enable secure login checking
  mac-address-table  Configure the MAC address table
  no                 Negate a command or set its defaults
  ntp                Configure NTP
  parameter-map      parameter map
  parser             Configure parser
  policy-map         Configure QoS Policy Map
  port-channel       EtherChannel configuration
  priority-list      Build a priority list
  privilege          Command privilege parameters
  queue-list         Build a custom queue list
  router             Enable a routing process
  secure             Secure image and configuration archival commands
  security           Infra Security CLIs
  service            Modify use of network based services
  snmp-server        Modify SNMP engine parameters
  spanning-tree      Spanning Tree Subsystem
  tacacs-server      Modify TACACS query parameters
  username           Establish User Name Authentication
  vpdn               Virtual Private Dialup Network
  vpdn-group         VPDN group configuration
  zone               FW with zoning
  zone-pair          Zone pair command
R2(config)#ip
R2(config)#ip 
R2(config)#ip ?
  access-list       Named access-list
  cef               Cisco Express Forwarding
  default-gateway   Specify default gateway (if not routing IP)
  default-network   Flags networks as candidates for default routes
  dhcp              Configure DHCP server and relay parameters
  domain            IP DNS Resolver
  domain-lookup     Enable IP Domain Name System hostname translation
  domain-name       Define the default domain name
  flow-export       Specify host/port to send flow statistics
  forward-protocol  Controls forwarding of physical and directed IP broadcasts
  ftp               FTP configuration commands
  host              Add an entry to the ip hostname table
  inspect           Context-based Access Control Engine
  ips               Intrusion Prevention System
  local             Specify local options
  name-server       Specify address of name server to use
  nat               NAT configuration commands
  route             Establish static routes
  routing           Enable IP routing
  scp               Scp commands
  ssh               Configure ssh options
  tcp               Global TCP parameters
R2(config)#ip rou
R2(config)#ip route 0.0.0.0 0.0.0.0 10.0.0.1
R2(config)#do ping 10.0.0.1

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.0.0.1, timeout is 2 seconds:
.!!!!
Success rate is 80 percent (4/5), round-trip min/avg/max = 0/0/0 ms

R2(config)#do wr mem
Building configuration...
[OK]
R2(config)#
```

Для сетевой связанности нужно добавить маршрут на R1 по умолчанию на соседний маршрутизатор R2:

```
R1>
R1>
R1>en
Password: 
R1#ip ro
R1#ip route
R1#ip route 0.0.0.0 0.0.0.0 10.0.0.2
      ^
% Invalid input detected at '^' marker.
	
R1#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
R1(config)#ip rou
R1(config)#ip route 0.0.0.0 0.0.0.0 10.0.0.2
R1(config)#do ping 192.168.1.97

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.1.97, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 0/0/0 ms

R1(config)#
```


Согласно схеме на коммутаторе S1 порт 6 соединен с РСА, настроим соответсвующий VLAN:

```
S1>
S1>
S1>
S1>en
Password: 
S1#conf
S1#configure in
S1#configure int
S1#configure t
Enter configuration commands, one per line.  End with CNTL/Z.
S1(config)#int
S1(config)#interface fa
S1(config)#interface fastEthernet 0/6
S1(config-if)#swi
S1(config-if)#switchport m
S1(config-if)#switchport mode acc
S1(config-if)#switchport mode access 
S1(config-if)#swi
S1(config-if)#switchport acc
S1(config-if)#switchport access vl
S1(config-if)#switchport access vlan 100
S1(config-if)#do sh vlan br

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Fa0/5
100  CLIENTS                          active    Fa0/6
200  MGM                              active    
999  Parking-Lot                      active    Fa0/1, Fa0/2, Fa0/3, Fa0/4
                                                Fa0/7, Fa0/8, Fa0/9, Fa0/10
                                                Fa0/11, Fa0/12, Fa0/13, Fa0/14
                                                Fa0/15, Fa0/16, Fa0/17, Fa0/18
                                                Fa0/19, Fa0/20, Fa0/21, Fa0/22
                                                Fa0/23, Fa0/24, Gig0/1, Gig0/2
1000 Native                           active    
1002 fddi-default                     active    
1003 token-ring-default               active    
1004 fddinet-default                  active    
1005 trnet-default                    active    
```
А на порту 5 настроим нативный влан 1000 и транк на вланы 100 200 и 1000 соответсвенно:

```
S1(config)#
S1(config)#
S1(config)#
S1(config)#int
S1(config)#interface fa
S1(config)#interface fastEthernet 0/5
S1(config-if)#swi
S1(config-if)#switchport m
S1(config-if)#switchport mode tr
S1(config-if)#switchport mode trunk 

S1(config-if)#
%LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/5, changed state to down

%LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/5, changed state to up

%LINEPROTO-5-UPDOWN: Line protocol on Interface Vlan200, changed state to up

S1(config-if)#sw
S1(config-if)#switchport tr
S1(config-if)#switchport trunk na
S1(config-if)#switchport trunk native vl
S1(config-if)#switchport trunk native vlan 1000
S1(config-if)#swi
S1(config-if)#switchport tr
S1(config-if)#switchport trunk all
S1(config-if)#switchport trunk allowed vl
S1(config-if)#switchport trunk allowed vlan 100,200,1000
S1(config-if)#do sh int trunk
Port        Mode         Encapsulation  Status        Native vlan
Fa0/5       on           802.1q         trunking      1000

Port        Vlans allowed on trunk
Fa0/5       100,200,1000

Port        Vlans allowed and active in management domain
Fa0/5       100,200,1000

Port        Vlans in spanning tree forwarding state and not pruned
Fa0/5       none

S1(config-if)#
```

настроим DHCP

```
R1>en
Password: 
R1#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
R1(config)#ip dh
R1(config)#ip dhcp ex
R1(config)#ip dhcp excluded-address 192.168.1.1 192.168.1.5
R1(config)#ip dhcp excluded-address 192.168.1.97 192.168.1.101
R1(config)#ip dh
R1(config)#ip dhcp poo
R1(config)#ip dhcp pool CLIENTS
R1(dhcp-config)#net
R1(dhcp-config)#network 192.168.1.0 255.255.255.192
R1(dhcp-config)#dom
R1(dhcp-config)#domain-name CCNA-lab.com
R1(dhcp-config)#def
R1(dhcp-config)#de
R1(dhcp-config)#default-router ?
  A.B.C.D  Router's IP address
R1(dhcp-config)#default-router 192.168.1.1
R1(dhcp-config)#?
  default-router  Default routers
  dns-server      Set name server
  domain-name     Domain name
  exit            Exit from DHCP pool configuration mode
  network         Network number and mask
  no              Negate a command or set its defaults
  option          Raw DHCP options
R1(dhcp-config)#l
R1(dhcp-config)#le
R1(dhcp-config)#exit
R1(config)#ip
R1(config)#ip dh
R1(config)#ip dhcp poo
R1(config)#ip dhcp pool R2-Clients
R1(dhcp-config)#net
R1(dhcp-config)#network 192.168.1.96 255.255.255.240
R1(dhcp-config)#dom
R1(dhcp-config)#domain-name CCNA-lab.com
R1(dhcp-config)#def
R1(dhcp-config)#default-router 192.168.1.97
R1(dhcp-config)#lease ?
% Unrecognized command
R1(dhcp-config)#do wr mem
Building configuration...
[OK]
R1(dhcp-config)#
```
Так же можно задать время аренда для ip адреса например на на 2 дня 12 часов и 30 минут, делается это командой lease 2 12 30.
Однако в нашем случае данный функцианал в эмуляторе отсутствут:
```
R1(dhcp-config)#?
  default-router  Default routers
  dns-server      Set name server
  domain-name     Domain name
  exit            Exit from DHCP pool configuration mode
  network         Network number and mask
  no              Negate a command or set its defaults
  option          Raw DHCP options
R1(dhcp-config)#
```

После чего проверим конфигурацию DHCP:

![](./2.png)

Как видно РС-А получил соответсвующий адрес из пула.

Проверим сетевую связанность на РС-А:

![](./3.png)

Далее необходимо настроить ретрянсляцию DHCP на R2:

![](./4.png)

Согласно справки команда  "helper-address"  служит для указания адреса назначения для широковещательных UDP-запросов.

```
R2>
R2>
R2>en
Password: 
R2#conf  t
Enter configuration commands, one per line.  End with CNTL/Z.
R2(config)#int
R2(config)#interface gig
R2(config)#interface gigabitEthernet 0/0/1
R2(config-if)#ip ?
  access-group     Specify access control for packets
  address          Set the IP address of an interface
  authentication   authentication subcommands
  flow             NetFlow Related commands
  hello-interval   Configures IP-EIGRP hello interval
  helper-address   Specify a destination address for UDP broadcasts
  inspect          Apply inspect name
  ips              Create IPS rule
  mtu              Set IP Maximum Transmission Unit
  nat              NAT interface commands
  ospf             OSPF interface commands
  proxy-arp        Enable proxy ARP
  split-horizon    Perform split horizon
  summary-address  Perform address summarization
R2(config-if)#ip he
R2(config-if)#ip hel
R2(config-if)#ip helper-address 10.0.0.1
R2(config-if)#do sh run
Building configuration...

Current configuration : 1230 bytes
!
version 15.4
no service timestamps log datetime msec
no service timestamps debug datetime msec
service password-encryption
security passwords min-length 14
!
hostname R2
!
!
!
enable secret 5 $1$mERr$9cTjUIEqNGurQiFU.ZeCi1
!
!
!
!
!
!
ip cef

R2(config-if)#
R2#
%SYS-5-CONFIG_I: Configured from console by console

R2#do wr mem
      ^
% Invalid input detected at '^' marker.
	
R2#

```
Проверяем сетевую связанность от РС-В с РС-А:

![](./5.png)

Проверяем статистику DHCP на R1:

```
R1#sh
R1#show ip dh
R1#show ip dhcp ?
  binding   DHCP address bindings
  conflict  DHCP address conflicts
  pool      DHCP pools information
  relay     Miscellaneous DHCP relay information
R1#show ip dhcp bi
R1#show ip dhcp binding 
IP address       Client-ID/              Lease expiration        Type
                 Hardware address
192.168.1.6      0004.9A41.5EC3           --                     Automatic
192.168.1.102    00E0.B051.0E1E           --                     Automatic
R1#
```

![PTK файл enable password: class](./DHCPPv4.pkt)

