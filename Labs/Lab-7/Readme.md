### Лабораторная работа. Развертывание коммутируемой сети с резервными каналами

![](./1.png)

### Таблица адресации
| Устройство           |   Интерфейс     | 		IP-адрес        |   Маска подсети               |
|:---------------------|:---------------:|:---------------------|:---------------------------:|
|S1                    |VLAN 1           | 192.168.1.1          |  255.255.255.0              |
|S2                    |VLAN 1           | 192.168.1.2          |255.255.255.0                |
|S3                    |VLAN 1           | 192.168.1.3          |255.255.255.0                | 

### Задачи
+ Часть 1. Создание сети и настройка основных параметров устройства
+ Часть 2. Выбор корневого моста
+ Часть 3. Наблюдение за процессом выбора протоколом STP порта, исходя из стоимости портов
+ Часть 4. Наблюдение за процессом выбора протоколом STP порта, исходя из приоритета портов
### Общие сведения/сценарий
Избыточность позволяет увеличить доступность устройств в топологии сети за счёт устранения единой точки отказа. Избыточность в коммутируемой сети обеспечивается посредством использования нескольких коммутаторов или нескольких каналов между коммутаторами. Когда в проекте сети используется физическая избыточность, возможно возникновение петель и дублирование кадров.
Протокол spanning-tree (STP) был разработан как механизм предотвращения возникновения петель на 2-м уровне для избыточных каналов коммутируемой сети. Протокол STP обеспечивает наличие только одного логического пути между всеми узлами назначения в сети путем намеренного блокирования резервных путей, которые могли бы вызвать петлю.
В этой лабораторной работе команда show spanning-tree используется для наблюдения за процессом выбора протоколом STP корневого моста. Также вы будете наблюдать за процессом выбора портов с учетом стоимости и приоритета.
Примечание. Используются коммутаторы Cisco Catalyst 2960s с Cisco IOS версии 15.0(2) (образ lanbasek9). Допускается использование других моделей коммутаторов и других версий Cisco IOS. В зависимости от модели устройства и версии Cisco IOS доступные команды и результаты их выполнения могут отличаться от тех, которые показаны в лабораторных работах. 
Примечание. Убедитесь, что все настройки коммутатора удалены и загрузочная конфигурация отсутствует. Если вы не уверены, обратитесь к инструктору.
	Необходимые ресурсы
•	3 коммутатора (Cisco 2960 с операционной системой Cisco IOS 15.0(2) (образ lanbasek9) или аналогичная модель)
•	Консольные кабели для настройки устройств Cisco IOS через консольные порты
•	Кабели Ethernet, расположенные в соответствии с топологией


### При первом подключении к устройствам, необходимо провести первоначальную настройку:

+ Задание паролей пользователя
+ Настройка ssh для подключения
+ Задания баннера


Сам перечень набора команд для S3:

```
en
conf t
hostname S3
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
Далее необходимо задать ip адрес для устройства

```
conf t
interface vlan 1
ip address 192.168.1.3 255.255.255.0
no shutdown 
end
```
По аналогии необходимо настроить два других коммутатора согласно таблице.
После чего, необходимо проверить доступность всех коммутаторов:
![](./2.png)

После чего необходимо определить корневой мост. Для этого необходимо 

Корневой мост служит точкой привязки для всех расчётов протокола spanning-tree, позволяя определить избыточные пути, которые следует заблокировать.
Процесс выбора определяет, какой из коммутаторов станет корневым мостом. Коммутатор с наименьшим значением идентификатора моста (BID) становится корневым мостом. Идентификатор BID состоит из значения приоритета моста, расширенного идентификатора системы и MAC-адреса коммутатора. Значение приоритета может находиться в диапазоне от 0 до 65535 с шагом 4096. По умолчанию используется значение 32768.
Первым делом необходимо отключить все порты на коммутаторах, для этого поочередно на каждом коммутаторе необходимо выполнить команду:

```
S2(config)#int ran
S2(config)#int range fa
S2(config)#int range fastEthernet 0/1-24
S2(config-if-range)#shu
S2(config-if-range)#shutdown 
```
![](./3.png)

Далее необходимо назначить используемые порты в качестве транковых:

```
S1(config)#int
S1(config)#interface fas
S1(config)#interface ran
S1(config)#interface range fa
S1(config)#interface range fastEthernet 0/3-4
S1(config-if-range)#?
  authentication    Auth Manager Interface Configuration Commands
  cdp               Global CDP configuration subcommands
  channel-group     Etherchannel/port bundling configuration
  channel-protocol  Select the channel protocol (LACP, PAgP)
  description       Interface specific description
  dot1x             Interface Config Commands for IEEE 802.1X
  duplex            Configure duplex operation.
  exit              Exit from interface configuration mode
  ip                Interface Internet Protocol config commands
  lldp              LLDP interface subcommands
  mdix              Set Media Dependent Interface with Crossover
  mls               mls interface commands
  no                Negate a command or set its defaults
  shutdown          Shutdown the selected interface
  spanning-tree     Spanning Tree Subsystem
  speed             Configure speed operation.
  storm-control     storm configuration
  switchport        Set switching mode characteristics
  tx-ring-limit     Configure PA level transmit ring limit
S1(config-if-range)#
S1(config-if-range)#swi
S1(config-if-range)#switchport mo
S1(config-if-range)#switchport mode tru
S1(config-if-range)#switchport mode trunk 
S1(config-if-range)#end
```
Далее необходимо включить порты F2 и F4 на всех коммутаторах:

```
S1#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
S1(config)#int
S1(config)#interface fads
S1(config)#interface fas
S1(config)#interface fastEthernet 0/4
S1(config-if)#no shu
S1(config-if)#no shutdown 

%LINK-5-CHANGED: Interface FastEthernet0/4, changed state to down
S1(config-if)#exit
S1(config)#interface fastEthernet 0/2
S1(config-if)#no shu
S1(config-if)#no shutdown 

%LINK-5-CHANGED: Interface FastEthernet0/2, changed state to down
S1(config-if)#exit
S1(config)#
```
После чего проверить данные протокола spanning-tree каждого коммутатора:

![](./4.png)

```
S1#show spanning-tree 
VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     0001.C728.95DB
             Cost        19
             Port        1(FastEthernet0/1)
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     0003.E4B0.46C0
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  20

Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Fa0/1            Root FWD 19        128.1    P2p
Fa0/2            Altn BLK 19        128.2    P2p
Fa0/3            Desg FWD 19        128.3    P2p
Fa0/4            Desg FWD 19        128.4    P2p
```
```
S3#show sp
S3#show spanning-tree 
VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     0001.C728.95DB
             Cost        19
             Port        1(FastEthernet0/1)
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     000A.41B9.8B78
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  20

Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Fa0/1            Root FWD 19        128.1    P2p
Fa0/2            Altn BLK 19        128.2    P2p
Fa0/3            Altn BLK 19        128.3    P2p
Fa0/4            Altn BLK 19        128.4    P2p

```
```
S2#show sp
S2#show spanning-tree 
VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     0001.C728.95DB
             This bridge is the root
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     0001.C728.95DB
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  20

Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Fa0/1            Desg FWD 19        128.1    P2p
Fa0/2            Desg FWD 19        128.2    P2p
Fa0/3            Desg FWD 19        128.3    P2p
Fa0/4            Desg FWD 19        128.4    P2p

```
Как видно корневым коммутатором является S2

![](./5.png)

С учетом выходных данных, поступающих с коммутаторов, ответьте на следующие вопросы.
Какой коммутатор является корневым мостом? 
Ответ:
S2

Почему этот коммутатор был выбран протоколом spanning-tree в качестве корневого моста?
Ответ:
Потому что у него наименьшее значение идентификатора моста (BID) 
Какие порты на коммутаторе являются корневыми портами? 
Ответ:
S1-Fast 0/1, S3-fast 0/1
Какие порты на коммутаторе являются назначенными портами? 
Ответ:
В качестве действующего 
S1:fa0/1
S2:fa0/4
Какой порт отображается в качестве альтернативного и в настоящее время заблокирован? 
Ответ:
Да.
S1: fa0/3, fa0/4,fa0/2
S3: fa0/3.
Почему протокол spanning-tree выбрал этот порт в качестве невыделенного (заблокированного) порта?
Ответ:
Потому что на каждом этапе сравнения он оказался "хуже" своего соседа.


































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
Далее необходимо назначить VLAN на коммутаторы S1 и S2 согласно таблице.
Рассмотрим пример создания VLAN на примере S1, настройка на коммутаторе S2 производится аналогично:

```
S1#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
S1(config)#vlan 10
S1(config-vlan)#na
S1(config-vlan)#name vlan-10
S1(config-vlan)#exit
S1(config)#interface fastEthernet 0/1
S1(config-if)#swi
S1(config-if)#switchport acc
S1(config-if)#switchport mod
S1(config-if)#switchport mode acc
S1(config-if)#switchport mode access ?
  <cr>
S1(config-if)#switchport mode access 
S1(config-if)#swi
S1(config-if)#switchport acc
S1(config-if)#switchport access vl
S1(config-if)#switchport access vlan 10
S1(config-if)#
```
Убедимся что vlan назначен на нужный порт

![](./2.png)

Далее по аналогии назначаем соответствующие vlan на интерфейсы коммутатора согласно таблице:

```
S1(config-if)#exit
S1(config)#vlan 20
S1(config-vlan)#na
S1(config-vlan)#name vlan-20
S1(config-vlan)#exit
S1(config)#int fa0/6
S1(config-if)#switchport mode access
S1(config-if)#switchport access vlan 20
S1(config-if)#exit
S1(config)#int
S1(config)#interface ra
S1(config)#interface range fa
S1(config)#interface range fastEthernet 0/2-5
S1(config-if-range)#swi
S1(config-if-range)#switchport mo
S1(config-if-range)#switchport mode acc
S1(config-if-range)#switchport mode access 
S1(config-if-range)#swi
S1(config-if-range)#switchport 
%CDP-4-NATIVE_VLAN_MISMATCH: Native VLAN mismatch discovered on FastEthernet0/1 (10), with S1 FastEthernet0/1 (1).
acc
S1(config-if-range)#switchport access vla
S1(config-if-range)#switchport access vlan 999
S1(config-if-range)#
S1(config-if-range)#exit
S1(config)#int
S1(config)#interface ra
S1(config)#interface range gig
S1(config)#interface range gigabitEthernet 0/1-2
S1(config-if-range)#swi
S1(config-if-range)#switchport mo
S1(config-if-range)#switchport mode acc
S1(config-if-range)#switchport mode access 
%CDP-4-NATIVE_VLAN_MISMATCH: Native VLAN mismatch discovered on FastEthernet0/1 (10), with S1 FastEthernet0/1 (1).

S1(config-if-range)#swi
S1(config-if-range)#switchport acc
S1(config-if-range)#switchport access vla
S1(config-if-range)#switchport access vlan 999
S1(config-if-range)#
```
Проверяем таблицу VLAN

![](./3.png)

После чего необходимо задать режим trunk для магистральных портов коммутатора:
```
S1(config-if)#switchport trunk native vlan 999
S1(config-if)#switchport trunk allowed vlan 10,20,30,999
S1(config-if)#switchport mode trunk
```
таким образом между коммутаторами разрешаем прохождения трафика для соответствующих Vlan
Проверим транки:

![](./4.png)

Так, у нас похоже ошибка, между интерфейсом fa0/1 и fa0/5 нет транка vlan 10,20,30,999, так же нет нативного влана на fa0/5.
После просмотра конфига, оказалось что отсутсвует vlan 30 на S1. Конечные настройки на S1 выглядят так:

![](./5.png)
![](./6.png)


Аналогично проводим настройки согласно таблице и для коммутатора S2.
Результат должен быть аналогичный:
![](./7.png)
После чего создаем подынтерфейсы на маршрутизаторе c 10,20,30,999 на каждом интерфейсе прописываем инкапсуляцию на соответсвующие vlan 10,20,30 и 999, так же не забываем прописывать ip address на каждом из них согласно таблице, конечные настройки будут выглядеть следующим образом:
![](./8.png)
 После проверяем доступность до PC-A и PC-B%

 ![](./9.png)

 Проверяем сетевую связанность со всеми сетевыми устройствами с РС-А и РС-В:

  ![](./10.png)

