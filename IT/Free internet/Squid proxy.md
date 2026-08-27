---
share:
tags:
  - "#public"
  - proxy
  - network
  - linux
  - debian
  - free_internet
---
HTTP прокси
- Устанавливаем расширение для браузера [Proxy SwitchyOmega](https://chromewebstore.google.com/detail/proxy-switchyomega/padekgcemlokbadohgkifijomclgjgif) или [# Proxy SwitchyOmega 3 (ZeroOmega)](https://chromewebstore.google.com/detail/proxy-switchyomega-3-zero/pfnededegaaopdmhkdmcofjmoldfiped)
- Устанавливаем http прокси squid [GitHub](https://github.com/squid-cache/squid)
- Указываем параметры для подключения прокси в расширении ProxySwitchyOmega

## Debian установка и настройка прокси сервера
1. Обновляем пакеты
2. Устанавливаем прокси squid и пакет с утилитами Apache (включая `htpasswd`)
3. Добавляем squid в автозагрузку
4. Открываем конфиг прокси
```bash
apt update && apt upgrade -y
apt install squid apache2-utils -y
systemctl enable squid
nano /etc/squid/squid.conf
```
5. Копируем туда конфиг
```ini
# Настраивает аутентификацию через NCSA-файл с паролями
auth_param basic program /usr/lib/squid/basic_ncsa_auth /etc/squid/passwd
# Задаёт название области аутентификации (отображается в браузере)
auth_param basic realm Proxy
# Требующий аутентификации для доступа
acl authenticated proxy_auth REQUIRED
# Разрешает HTTP-доступ только аутентифицированным пользователям
http_access allow authenticated

# Указываем на каком порту работает прокси
http_port 3128 

# Разрешим доступ из всех IP (опционально, можно ограничить по IP)
acl all src 0.0.0.0/0
http_access allow all
```
6. Создаем файл где будут хранится УД от подключения к proxy, с пользователем `proxyuser` , и указываем пароль 2 раза
7. Рестарт службы!
```bash
htpasswd -c /etc/squid/passwd proxyuser
systemctl restart squid.service
```


Убедимся что порт `3128` у нас открыт на нашем сервере, сделать это можно в windows через `powershell` 
```powershell
Test-NetConnection my_server.ru -port 3128
```
Если `TcpTestSucceeded : True` то ОК если `False` то порт не открыт!
## Настройка расширения Proxy SwitchyOmega и добавление сайтов
Устанавливаем отсюда: [Магазин Chrome](https://chromewebstore.google.com/detail/proxy-switchyomega/padekgcemlokbadohgkifijomclgjgif?hl=ru)
В Profiles - Proxy Указываем настройки подключения к прокси 
![[Pasted image 20250630212022.png]]
Выбираем режим auto swith, добавляем правило для сайта
![[Pasted image 20251028130600.png]] .
Все добавленные правила будут отображаться в профиле в настройках расширения
![[image-фывфывфывфыв.png]]