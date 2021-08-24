## ЛЛ №1
#### Работа с VLAN. Межвлановая мршрутизация.

###### Схема сети
![alt-текст](https://github.com/udalovsa/otus-ccna/blob/main/lab01/LL1.JPG "12345")


###### Базовая настройка коммутатора

Подключаюсь к коммутатору. Активирую режим привилегий.

``` S2>enable ```

Запускаю режим конфигурации.

``` S2#conf t ```

Задаю има коммутатора S1.

``` hostname S1 ```

Для коммутатора S2:

``` hostname S2 ```

Отключаю обработку DNS:

``` no ip domain-lookup ```

Устанавливаю пароль на режим привилегий :

``` enable password class ```

Устанавливаю пароль на консольный вход на устройство:

``` 
S2(config)#line console 0
S2(config-line)#password cisco
S2(config-line)#login
S2(config-line)#exit
```
Устанавливаю пароль на вход через терминал:

```
S2(config)#line vty 0 1500
S2(config-line)#password cisco
S2(config-line)#login
S2(config-line)#exit
```
Шифрую пароли, чтобы они были в нейвном виде при выводе конфигурации:
```
S2(config)#service password-encryption
```
Вывожу предупреждение при подключении к коммутатору:
```
S2(config)#banner motd c
Enter TEXT message.  End with the character 'c'.
TEST messege
c
S2(config)#
```
Устанавливаю время на коммутаторе:
Вручную:
```
S2#clock set 22:22:22 11 september 2025
```
Обновление по NTP:
```
S2(config)#clock timezone UTC + 3
S2(config)#ntp server 85.21.78.91
```
Скопировать конфиг в стартовый:
``` 
S2#copy running-config sta
S2#copy running-config startup-config
Destination filename [startup-config]?
```
###### Базовая настройка роутера

Базовая настройка роутера аналогична настроки коммутаторов.
Я приведу команды без пояснения.
```
hostname R2

no ip domain-lookup

line console 0
password cisco
login
exit

line vty 0 924
password cisco
login
exit

service password-encryption

banner motd c
Enter TEXT message.  End with the character 'c'.
TEST messege
c

clock timezone UTC + 3
ntp server 85.21.78.91

copy running-config startup-config
```
##### Создание вланов и настройка маршрутизации:
![alt-текст](https://github.com/udalovsa/otus-ccna/blob/main/lab01/T1.JPG "ТЗ")

Для распространения вланов на коммутаторы я использовал протокол VTP.
Коммутаторй S2 является vtp сервером.
```
vtp mode server
vtp domain otus
vtp version 2
vtp password 12345
```
Коммутатор 1 является клиентом.
```
vtp mode client
```

На коммутаторе S2 создаю вланы согласно таблице ТЗ
``` 
vlan N
```
где N - номер влана.
Проверяю вланы на S2 и S1

```S2#show vlan brief

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active
3    Management                       active
4    Operations                       active    Gi0/0
7    ParkingLot                       active    Gi0/1, Gi0/2
8    Native                           active
1002 fddi-default                     act/unsup
1003 trcrf-default                    act/unsup
1004 fddinet-default                  act/unsup
1005 trbrf-default                    act/unsup
```
```

S1#sh vtp status
VTP Version capable             : 1 to 3
VTP version running             : 2
VTP Domain Name                 : otus
VTP Pruning Mode                : Disabled
VTP Traps Generation            : Disabled
Device ID                       : 5000.0002.0000
Configuration last modified by 0.0.0.0 at 8-16-21 21:32:52

Feature VLAN:
--------------
VTP Operating Mode                : Client
Maximum VLANs supported locally   : 1005
Number of existing VLANs          : 9
Configuration Revision            : 5
MD5 digest                        : 0xF1 0x96 0x7D 0x50 0x75 0xA9 0xF9 0x46
                                    0x22 0xD5 0xDF 0x7E 0x0E 0x01 0x8F 0xFE
S1#sh vla
S1#sh vlan bri
S1#sh vlan brief

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Gi0/1
3    Management                       active    Gi0/0
4    Operations                       active
7    ParkingLot                       active
8    Native                           active
1002 fddi-default                     act/unsup
1003 trcrf-default                    act/unsup
1004 fddinet-default                  act/unsup
1005 trbrf-default                    act/unsup
```
Настраиваю интерфейсы согласно ТЗ: магистральные порты, порты доступа в указанном влане, присваиваю адрес коммутатору в валне 3, указываю шлюз.
```
interface GigabitEthernet0/0
 switchport access vlan 4
 switchport mode access
 media-type rj45
 negotiation auto
!
interface GigabitEthernet0/1
 switchport access vlan 7
 switchport mode access
 media-type rj45
 negotiation auto
!
interface GigabitEthernet0/2
 switchport access vlan 7
 switchport mode access
 media-type rj45
 negotiation auto
!
interface GigabitEthernet0/3
 switchport trunk encapsulation dot1q
 switchport trunk native vlan 8
 switchport mode trunk
 media-type rj45
 no negotiation auto
!
interface Vlan3
 ip address 192.168.3.12 255.255.255.0
!
ip default-gateway 192.168.3.1
```
Коммутатор S1
```
interface GigabitEthernet0/0
 switchport access vlan 3
 switchport mode access
 media-type rj45
 negotiation auto
!
interface GigabitEthernet0/1
 switchport mode access
 media-type rj45
 negotiation auto
!
interface GigabitEthernet0/2
 description R1
 switchport trunk allowed vlan 3,4,8
 switchport trunk encapsulation dot1q
 switchport trunk native vlan 8
 switchport mode trunk
 media-type rj45
 no negotiation auto
!
interface GigabitEthernet0/3
 switchport trunk allowed vlan 3,4,8
 switchport trunk encapsulation dot1q
 switchport trunk native vlan 8
 switchport mode trunk
 media-type rj45
 no negotiation auto
!
interface Vlan3
 ip address 192.168.3.11 255.255.255.0
!
ip default-gateway 192.168.3.1
```

##### Настройка шлюза и межвлановой маршрутизации.

Для реализации концепции RoS использую 1 интерфейс маршрутизатора. Пример настройки:
```
interface GigabitEthernet0/2.3
 encapsulation dot1Q 3
 ip address 192.168.3.1 255.255.255.0
!
interface GigabitEthernet0/2.4
 encapsulation dot1Q 4
 ip address 192.168.4.1 255.255.255.0
!
interface GigabitEthernet0/2.8
 encapsulation dot1Q 8 native
```
1. использую виртуальные интерфейсы для тегированного трафика вланов 3.4.8
2. указываю адреса интерфейсов в указанных вланах.


##### проверяю с рабочих станций маршрутизацию.

![alt-текст](https://github.com/udalovsa/otus-ccna/blob/main/lab01/PCA.jpg "PCA")
![alt-текст](https://github.com/udalovsa/otus-ccna/blob/main/lab01/PCB.jpg "PCB")

