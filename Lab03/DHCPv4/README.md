Настройка DHCPv4 на роутере Cisco

Базовая настройка роутера на примере R2

```
Router(config)#hostname R2
```
Отключение обработки DNS запросов
```
no ip domain-lookup
```

установка пароля для входа в режим привилегий

```
R2(config)#enable password class
```
установка пароля при использовании консоли и терминала:
```
R2(config)#line console 0
R2(config-line)#password cisco
R2(config-line)#login
R2(config-line)#exi

R2(config)#line vty 0 4
R2(config-line)#password cisco
R2(config-line)#login
R2(config-line)#exit

```
скрыть пароль в конфиг. файле

```
R2(config)#service password-encryption
```

Баннер - предупреждение
```
R2(config)#banner motd c
Enter TEXT message.  End with the character 'c'.
###
###Create the banner that warns anyone thai he is a cock. ###
```

Сохраняю
```
R2#write
Building configuration...
[OK]
R2#
*Oct 12 11:22:59.266: %GRUB-5-CONFIG_WRITING: GRUB configuration is being updated on disk. Please wait...
*Oct 12 11:23:00.520: %GRUB-5-CONFIG_WRITTEN: GRUB configuration was written to disk successfully.
R2#cop
R2#copy run
R2#copy running-config sta
R2#copy running-config startup-config
Destination filename [startup-config]?
Building configuration...
[OK]
R2#
*Oct 12 11:23:14.563: %GRUB-5-CONFIG_WRITING: GRUB configuration is being updated on disk. Please wait...
*Oct 12 11:23:15.824: %GRUB-5-CONFIG_WRITTEN: GRUB configuration was written to disk successfully.

```
установка времени и даты
```
R2#clock set 4:25:00 12 oct 2021
```

Конфигурирую интерфейс на R1:
```
interface GigabitEthernet0/0.100
 description Clients
 encapsulation dot1Q 100
!
interface GigabitEthernet0/0.200
 description Management
 encapsulation dot1Q 200
!
interface GigabitEthernet0/0.1000
 description Native
 encapsulation dot1Q 1000
```

Конфигурация адресов на интерфейсах R1
```
R1(config)#interface Gi 0/1
R1(config-if)#ip address 10.0.0.1 255.255.255.252
```
конфигурация DHCP сервера в ВЛАНе 100
```
R1#sh runn | begin dhcp
ip dhcp excluded-address 192.168.1.0 192.168.1.100
ip dhcp excluded-address 192.168.1.158 192.168.1.255
!
ip dhcp pool Clients
 network 192.168.1.0 255.255.255.0
 default-router 192.168.1.1
 dns-server 192.168.1.1
 domain-name home.local
```


```
