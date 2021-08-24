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


##### Задание:
Изучить межвлановую маршрутизацию (InterVLAN Routing).
Присвоить оборудованию адреса, согласно таблице T1.jpg
Смоделировать коммутацию согласно схеме SH1.jpg
Выполнить простейшую настройку коммутаторов и роутера.

##### Решение:
Выполнена настройка доступа с авторизацией.
На коммутаторах S1 и S2 магистральные порты переведены в режи мработы Trunk
Для распространения VLANов использую VTP.
На шлюзе настроена маршрутизация.
Лабораторный стенд - LL1.jpg

