![](./2.png)


| Устройство           |   Интерфейс     | 		IP-адрес        |   Маска подсети             | 	Шлюз по умолчанию     |
|:---------------------|:---------------:|:---------------------|:---------------------------:|:--------------------------|
|R1                    |G0/0/1           | 192.168.1.1          |  255.255.255.0              | -                         |
|S1                    |VLAN 1           |   192.168.1.11       |  255.255.255.0              | 192.168.1.1               |
|PC-A                  |NIC              |   192.168.1.3        |  255.255.255.0              | 192.168.1.1               |


### Задачи

+ Часть 1. Настройка основных параметров устройства
+ Часть 2. Настройка маршрутизатора для доступа по протоколу SSH
+ Часть 3. Настройка коммутатора для доступа по протоколу SSH
+ Часть 4. SSH через интерфейс командной строки (CLI) коммутатора

Первое что необходимо сделать - это настроить ip адресацию каждого утсройства, задать пароль для привилигированного пользователя и скрыть его встроенным алгоритмом шифрования (эдакая защита от подглядывания из-за спины)
настройка будет производится согласно таблице представленной выше. Необходимо будет так же прописать баннер (The device is the property of the company, any unauthorized change to the configuration is punishable by law.) 
Отключите поиск DNS, чтобы предотвратить попытки маршрутизатора неверно преобразовывать введенные команды таким образом, как будто они являются именами узлов.
d.	Назначьте class в качестве зашифрованного пароля привилегированного режима EXEC.
e.	Назначьте cisco в качестве пароля консоли и включите вход в систему по паролю.
f.	Назначьте cisco в качестве пароля VTY и включите вход в систему по паролю.
После чего необходимо будет настроить доступ по ssh ver.2.0, длинну ключа необходимо будет использовать не менее 2048 бит.
Задать длинну пороля не менее 14 символов.

 

Switch(config)#interface range fastEthernet 0/24
Switch(config-if-range)#no shu
Switch(config-if-range)#no shutdown 
Switch(config-if-range)#exit
Switch(config)#int
Switch(config)#interface vla
Switch(config)#interface vlan 1
Switch(config-if)#ip add
Switch(config-if)#ip address 192.168.1.11 255.255.255.0
Switch(config-if)#exit
Switch(config)#hostname S1
S1(config)#int vlan 1
S1(config-if)#no shu
S1(config-if)#no shutdown 
S1(config-if)#exit
S1(config)#banne
S1(config)#banner ?
  motd  Set Message of the Day banner
S1(config)#banner m
S1(config)#banner motd ^The device is the property of the company, any unauthorized change to the configuration is punishable by law.^
S1(config)#ip domain-name otus.ru
S1(config)#no ip domain-lookup
S1(config)#enable secret class
S1(config)#username cisco secret class
S1(config)#service password-encryption 
S1(config)#crypto key gen
S1(config)#crypto key generate ?
  rsa  Generate RSA keys
S1(config)#crypto key generate rs
S1(config)#crypto key generate rsa 
The name for the keys will be: S1.otus.ru
Choose the size of the key modulus in the range of 360 to 2048 for your
  General Purpose Keys. Choosing a key modulus greater than 512 may take
  a few minutes.

How many bits in the modulus [512]: 2048
% Generating 2048 bit RSA keys, keys will be non-exportable...[OK]
S1(config)#ip ssh
S1(config)#ip ssh ver
S1(config)#ip ssh version ?
  <1-2>  Protocol version
S1(config)#ip ssh version 2
S1(config)#sh
S1(config)#do sh ip ssh
SSH Enabled - version 2.0
Authentication timeout: 120 secs; Authentication retries: 3
S1(config)#
S1(config)#username admin pri
S1(config)#username admin privilege 15 ?
  password  Specify the password for the user
  secret    Specify the secret for the user
  <cr>
S1(config)#username admin privilege 15 se
S1(config)#username admin privilege 15 secret cisco
S1(config)#line vty 0
S1(config-line)#lo
S1(config-line)#logging synchronous
S1(config-line)#exit
S1(config)#lin
S1(config)#line vt
S1(config)#line vty 0 4 
S1(config-line)#login ?
  authentication  authenticate using aaa method list
  local           Local password checking
  <cr>
S1(config-line)#login local
S1(config-line)#tra
S1(config-line)#transport in
S1(config-line)#transport input ssh
S1(config-line)#exit
S1(config)#line
S1(config)#line vt
S1(config)#line vty 5 15
S1(config-line)#logi
S1(config-line)#login local
S1(config-line)#tra
S1(config-line)#transport int
S1(config-line)#transport in
S1(config-line)#transport input ssh
S1(config-line)#exit
S1(config)#ip def
S1(config)#ip default-gateway 192.168.1.1

Далее пробуем подключиться по ssh c PC:


По аналогии необходимо настроить маршрутизатор, сам процесс настройки я не буду приводить, а просто поочередно приведу набор команд, которые сожно будет только скопировать и вставить в CLI, конфиг будет отличаться только ip адресом и задания длинны пароля не менее 14 символов (но сам пароль будет стандартный, при следующем изменении пароля необходимо будет уже соблюдать требование).
Сам перечень набора команд:
en
conf t
interface GigabitEthernet0/0/1
ip addr 192.168.1.1 255.255.255.0
no shutdown
exit
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

После чего нужно попробовать подключиться по ssh и проверить версию протокола:

скрин

Попробовать подключиться к R1 через S1

скрин
