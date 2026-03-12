---
tags:
  - public
  - windows
  - wg
  - vpn
title: Разворачиваем Wireguard сервера на Windows
---

- ~~[Сайт](https://www.wireguardconfig.com/) Cгенирировать конфигурации для wg~~ 
- ~~Обязательно удалять в конфигах клиентам строчку `ListenPort =`~~
- ~~+ Добавлять строчку с рекламой в конфигурации~~

- [dbca-wa](https://dbca-wa.github.io/wg-webcfg/wg-webcfg.html) Лучше использовать этот или генерировать вручную.
![[Pasted image 20251029132500.png]]
- `Server` = Публичный (Внешний) IP Адрес сервера WG (Endopoint для клиентов)
- `Port` = Порт сервера WG
- `Net` = Подсеть VPN WG я обычно ставлю `10.10.0.0`
- `1stIP` = С какого IP адреса будут создаваться клиенты (обычно ставлю `2`)
- `Clinet` = Кол-во конфигураций для клиентов
- `ServerPrivatKey` и `ServerPublicKey` тут я думаю понятно приватный и публичный ключ сервера


### Клиенты wireguard
**Для всех клиентов требуются права администратора!**
- Windows С [офф сайта](https://www.wireguard.com/install)
- Windows  [TunSafe](https://tunsafe.com/download) (Расширенные возможности статистики VPN соединения) 
  _Если не устанавливается TAP адаптер качаем отдельно установщик TunSafe-TAP.exe_
- Windows Wiresock [офф сайт](https://www.wiresock.net/) (Много функций)
---
- macOS [скачать](https://itunes.apple.com/us/app/wireguard/id1451685025?ls=1&mt=12) 
- iOS [скачать](https://itunes.apple.com/us/app/wireguard/id1441195209?ls=1&mt=8)
- Android [скачать](https://play.google.com/store/apps/details?id=com.wireguard.android)
### Параметров конфигурации (Описание)
**Сервер**
```
[Interface]
`PrivateKey =` Приватный ключ сервера
`ListenPort =`  Порт которые слушает WG для подключений
`Address =` 10.0.0.1/24 IP Адрес сервера
[Peer]
`PublicKey =` Публичный ключ клиента 1
`AllowedIPs =` 10.0.0.2/32 IP Адрес клиента / 32 маска для маршрутизации
[Peer]
`PublicKey =` Публичный ключ клиента 2
`AllowedIPs =` 10.0.0.3/32 IP Адрес клиента / 32 маска для маршрутизации
```

**Клиент**
[Interface]
`PrivateKey =` Приватный ключ клиента
`Address =` 10.0.0.2/24 IP Адрес клиента
`DNS =` 8.8.8.8 IP Адреса DNS серверов (Если требуется указать свои)
[Peer]
`PublicKey =` Публичный ключ сервера
`AllowedIPs =` 10.0.0.0/28 Разрешенные подсети
`Endpoint =` 195.206.54.195:2024 Публичный IP адрес сервера
`PersistentKeepalive =` 30 Время проверки клиентом актуальности подключения

## Примеры конфигураций (Описание)
Сервер
```
[Interface]
PrivateKey = Private_key_server
ListenPort = 1194 Port_server
Address = 10.10.0.1/24 Network_VPN

[Peer] # client 2
PublicKey = Public_key_client2
AllowedIPs = 10.10.0.2/32 Для_формирование_маршрута_к_клиенту

[Peer] # client 3
PublicKey = Public_key_client3
AllowedIPs = 10.10.0.3/32 Для_формирование_маршрута_к_клиенту

[Peer] # client 4
PublicKey = Public_key_client4
AllowedIPs = 10.10.0.4/32 Для_формирование_маршрута_к_клиенту

```
Клиент
```
[Interface]
PrivateKey = Private_key_client
Address = 10.10.0.3/24 Ip_adress_client

[Peer]
PublicKey = Public_key_server
AllowedIPs = 10.10.0.0/24 Разрешенные_подсети
Endpoint = IP_and_port_Server
PersistentKeepalive = 30 Проверка_состояния_сессии
```

## Шаблоны Сервер - Клиент
Для сервера и 2х клиентов
- Изменить `PublicKey` на ключи своих клиентов
```
ListenPort = 1194
Address = 10.10.0.1/24

[Peer] Сlient_2 
PublicKey = Public_key_client2
AllowedIPs = 10.10.0.2/32

[Peer] Сlient_3
PublicKey = Public_key_client3
AllowedIPs = 10.10.0.3/32
```
Для клиента
- 1 раз изменить `PublicKey` на ключ своего сервера
- 1раз изменить `Endpoint` На IP своего сервера
-  При копировании для каждого клиента менять `Address` на 0.3, 0.4, 0.5, и т.д *иначе получится клиент 3 с ip 0.2 и так далее *
```
Address = 10.10.0.2/24 IP_adress_client

[Peer]
PublicKey = Public_key_server
AllowedIPs = 10.10.0.0/24
Endpoint = 11.111.11:1194
PersistentKeepalive = 30
```

## Черновик

Для сервера и 2х клиентов
- Изменить `PublicKey` на ключи своих клиентов
```
[Interface]
PrivateKey = IM/zk1Dk0Tk1rDBMWvcvdX0wZT/V4WXaTUEUjoAcdFg=
ListenPort = 1194
Address = 10.10.0.1/24

[Peer]
PublicKey = ihIci3klLhv0JOGvIxd5IB6FBoCm1ROP7KYBNjUx7DY=
AllowedIPs = 10.10.0.2/32
[Peer]
PublicKey = t2ERFc1nuDXTKcwl0R6ybC4viXn3FWxLmShC4lJQ1CU=
AllowedIPs = 10.10.0.3/32
[Peer]
PublicKey = HBhqgL1qI6eHxtGJegvXpXaSPQi0NhQSHQDklpivzwg=
AllowedIPs = 10.10.0.4/32
[Peer]
PublicKey = 7NygBYDEDrnO83ZtPd42aiIrgp1UceYIOorUBBHRI1s=
AllowedIPs = 10.10.0.5/32
[Peer]
PublicKey = B/MYDH5+gFhN1j0I/s/FBp+skuOEF9WwmcwiwRoVxGk=
AllowedIPs = 10.10.0.6/32
[Peer]
PublicKey = fZK7z8L9DDTK2g7n1rZzhCDhl5/0Ohd/HRwUPhfljWE=
AllowedIPs = 10.10.0.7/32
```

Clients
```
[Interface]
PrivateKey = kCkbY5ugnKRlw3FaW35juXfxjICHorQXQDwsLJYjDU4=
Address = 10.10.0.2/24
[Peer]
PublicKey = gi/0Cz6VhjfJfqpNk3/Y9spNUtHh0eFiRYrySo2AQHo=
AllowedIPs = 10.10.0.0/24
Endpoint = 90.188.227.46:1194
PersistentKeepalive = 30
```
3
```
[Interface]
PrivateKey = oF6KqNFzKqj9w/2BT3kzUQbnBf/NjoBAjGUhRtArUFo=
Address = 10.10.0.3/24
[Peer]
PublicKey = gi/0Cz6VhjfJfqpNk3/Y9spNUtHh0eFiRYrySo2AQHo=
AllowedIPs = 10.10.0.0/24
Endpoint = 90.188.227.46:1194
PersistentKeepalive = 30
```
4
```
[Interface]
PrivateKey = MKNF3evpUJ16z4i4YTZb4uIKeoYhElwKuqDy68NdCXU=
Address = 10.10.0.4/24
[Peer]
PublicKey = gi/0Cz6VhjfJfqpNk3/Y9spNUtHh0eFiRYrySo2AQHo=
AllowedIPs = 10.10.0.0/24
Endpoint = 90.188.227.46:1194
PersistentKeepalive = 30
```
5
```
[Interface]
PrivateKey = ED0BQFS7n8/W6sY8i4gs+yTQp38k39/hdTwiugAfmUM=
Address = 10.10.0.5/24
[Peer]
PublicKey = gi/0Cz6VhjfJfqpNk3/Y9spNUtHh0eFiRYrySo2AQHo=
AllowedIPs = 10.10.0.0/24
Endpoint = 90.188.227.46:1194
PersistentKeepalive = 30
```
6
```
[Interface]
PrivateKey = 6PNGu7ExwKiuhzSO4B0j2HpVjVcSRpTTecYfnkNvyXs=
Address = 10.10.0.6/24
[Peer]
PublicKey = gi/0Cz6VhjfJfqpNk3/Y9spNUtHh0eFiRYrySo2AQHo=
AllowedIPs = 10.10.0.0/24
Endpoint = 90.188.227.46:1194
PersistentKeepalive = 30
```
7
```
[Interface]
PrivateKey = iFL/awK2cgFbWj39tCJn7HEY+vKPqMgIygczGIwLe2E=
Address = 10.10.0.7/24
[Peer]
PublicKey = gi/0Cz6VhjfJfqpNk3/Y9spNUtHh0eFiRYrySo2AQHo=
AllowedIPs = 10.10.0.0/24
Endpoint = 90.188.227.46:1194
PersistentKeepalive = 30
```
