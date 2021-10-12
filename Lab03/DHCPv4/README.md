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
