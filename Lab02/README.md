# Развертывание коммутируемой сети с резервными каналами
### Топология
![alt-текст](https://github.com/udalovsa/otus-ccna/blob/main/Lab02/Capture.jpg "12345")
### Адресация коммутаторов
![alt-текст](https://github.com/udalovsa/otus-ccna/blob/main/Lab02/Capture1.jpg "12345")
### Цели работы
![alt-текст](https://github.com/udalovsa/otus-ccna/blob/main/Lab02/Capture2.jpg "12345")




#### Базовая настройка коммутаторов на примере коммутатора S1

Отключение DNS поиска
```
Switch(config)#no ip domain-lookup
```
Присваиваю имя
```
Switch(config)#hostname S1
```
Пароль на привилегированный режим
```
S1(config)#enable password class
```
Установка пароля на консоль и терминал
```
S1(config)#line con 0
S1(config-line)#password cisco
S1(config-line)#login
S1(config-line)#exit

S1(config)#line vty 0 1500

S1(config-line)#password cisco
S1(config-line)#login
S1(config-line)#exit
```
Так же добавляю в консоль настройку для возврата курсора при вводе команд при сообщениях ОС
```
S1(config-line)#logging synchronous
```
Добавляю банер
```
S1(config)#banner motd c
Enter TEXT message.  End with the character 'c'.
12345*************************************************************
c
S1(config)#
```
#### Назначение адресов
Задаю адрес коммутатора в vlan 1
```
S1(config)#interface vlan 1
*Aug 25 21:39:20.144: %LINEPROTO-5-UPDOWN: Line protocol on Interface Vlan1, changed state to down
S1(config-if)#ip address 192.168.1.1 255.255.255.0
S1(config-if)#no shutdown
```
Сохранаю настройки в стартовый конфиг файл
```
S1#copy running-config startup-config
Destination filename [startup-config]?
Building configuration...
```

#### Конфигурация STP.
Проверка доступности коммутаторов
```
S1#ping 192.168.1.2
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.1.2, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 4/5/9 ms
S1#ping 192.168.1.3
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.1.3, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 4/4/7 ms
```


Отключаю порты на коммутаторах
```
S1(config)#interface range GigabitEthernet 0/0 - 3
S1(config-if-range)#shutdown
```
Проверю отключение
```
S1#sh interfaces description
Interface                      Status         Protocol Description
Gi0/0                          admin down     down
Gi0/1                          admin down     down
Gi0/2                          admin down     down
Gi0/3                          admin down     down
Vl1                            up             up
```
Переключаю порты в режим транка
```
S1(config)#interface range GigabitEthernet 0/0 - 3
S1(config-if-range)#switchport trunk encapsulation dot1q
S1(config-if-range)#switchport mode trunk
```
Включаю интерфейсы 1 и 3
```
S2(config)#interface Gig 0/1
S2(config-if)#no shu
S2(config-if)#no shutdown
S2(config-if)#ex
S2(config-if)#exit
S2(config)#
*Aug 25 22:04:43.574: %LINK-3-UPDOWN: Interface GigabitEthernet0/1, changed state to up
S2(config)#
*Aug 25 22:04:44.575: %LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet0/1, changed state to up
S2(config)#inte Gig 0/3
S2(config-if)#no shu
S2(config-if)#ex
S2(config-if)#exit
```
Проверяю режим работы интерфейсов
```
S2#sh inte trunk

Port        Mode             Encapsulation  Status        Native vlan
Gi0/1       on               802.1q         trunking      1
Gi0/3       on               802.1q         trunking      1

Port        Vlans allowed on trunk
Gi0/1       1-4094
Gi0/3       1-4094

Port        Vlans allowed and active in management domain
Gi0/1       1
Gi0/3       1

Port        Vlans in spanning tree forwarding state and not pruned
Gi0/1       1
Gi0/3       1
```
##### Результат работы STP
```
S1#show spanning-tree

VLAN0001
  Spanning tree enabled protocol rstp
  Root ID    Priority    32769
             Address     5000.0001.0000
             This bridge is the root
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     5000.0001.0000
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Gi0/1               Desg FWD 4         128.2    Shr
Gi0/3               Desg FWD 4         128.4    Shr


S2#show spanning-tree

VLAN0001
  Spanning tree enabled protocol rstp
  Root ID    Priority    32769
             Address     5000.0001.0000
             Cost        4
             Port        2 (GigabitEthernet0/1)
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     5000.0002.0000
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Gi0/1               Root FWD 4         128.2    Shr
Gi0/3               Desg FWD 4         128.4    Shr



S3#show spanning-tree

VLAN0001
  Spanning tree enabled protocol rstp
  Root ID    Priority    32769
             Address     5000.0001.0000
             Cost        4
             Port        4 (GigabitEthernet0/3)
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     5000.0003.0000
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Gi0/1               Altn BLK 4         128.2    Shr
Gi0/3               Root FWD 4         128.4    Shr
```

В настроенной конфигурации корневой коммутатор - S1. Этот коммутаторй имеет самый меньший МАС, приоритеты у коммутаторов одинаковые, поэтому параметр адреса сыграл решающую роль в выборе рута. У коммутатора все порты назначенные, т.к. он является корневым. Заблокирован порт на коммутаторе S3 (альтернативный) в сторону S2, т.к. вес маршрута до руда больше, чем и второго порта.

Меняю стоимость маршрута на портуй коммутатора S3 для изменения root port
```
S3(config)#inte Gig 0/1
S3(config-if)#spanning-tree cost 1
S3(config-if)#do show spann

VLAN0001
  Spanning tree enabled protocol rstp
  Root ID    Priority    32769
             Address     5000.0001.0000
             Cost        4
             Port        4 (GigabitEthernet0/3)
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     5000.0003.0000
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Gi0/1               Altn BLK 1         128.2    Shr
Gi0/3               Root FWD 4         128.4    Shr

```
root port не изменился, т.к. стоимость маршрута через рут порт - 4, а через альтернативный стала 5 (была  8). 
тогда верну стоимость обратно и изменю стоимость порта 0/3
```
S3(config)#interface Gig 0/3
S3(config-if)#spanning-tree cost 20
S3(config-if)#do sho spa

VLAN0001
  Spanning tree enabled protocol rstp
  Root ID    Priority    32769
             Address     5000.0001.0000
             Cost        8
             Port        2 (GigabitEthernet0/1)
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     5000.0003.0000
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Gi0/1               Root FWD 4         128.2    Shr
Gi0/3               Altn BLK 20        128.4    Shr

```
после увеличения стоимости STP порта 0/3 произошла смена root порта. (4->8)
верну значение стоимости
```
S3(config-if)#no spanning-tree cost
```

##### Выбор порта при одинаковых значениях веса и приоритета:
Включаю все порты на некорвевых коммутаторах
```
S2(config)#inter range Gig 0/0 - 3
S2(config-if-range)#no shu
S2(config-if-range)#no shutdown

```
```
S2(config-if-range)#do show spann

VLAN0001
  Spanning tree enabled protocol rstp
  Root ID    Priority    32769
             Address     5000.0001.0000
             Cost        4
             Port        2 (GigabitEthernet0/1)
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     5000.0002.0000
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Gi0/0               Desg FWD 4         128.1    Shr
Gi0/1               Root FWD 4         128.2    Shr
Gi0/2               Desg FWD 4         128.3    Shr
Gi0/3               Desg FWD 4         128.4    Shr


S3(config-if-range)#do show spann

VLAN0001
  Spanning tree enabled protocol rstp
  Root ID    Priority    32769
             Address     5000.0001.0000
             Cost        4
             Port        4 (GigabitEthernet0/3)
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     5000.0003.0000
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Gi0/0               Altn BLK 4         128.1    Shr
Gi0/1               Altn BLK 4         128.2    Shr
Gi0/2               Desg FWD 4         128.3    Shr
Gi0/3               Root FWD 4         128.4    Shr
```
Результат абсолютно противоположенный относительно методички.

#### Контрольные вопросы
![alt-текст](https://github.com/udalovsa/otus-ccna/blob/main/Lab02/Capture3.JPG "12345")

После выбора корневого коммутатора протокол стп использует стоимость до рута при выборе порта.
Если стоимость одинакова, то сравнивает приоритет порта.
Если приоритет одинаковый, то сравнивает номер порта.
