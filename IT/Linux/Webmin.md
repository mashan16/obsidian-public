---
tags:
  - public
  - linux
title: Панель управления сервером Linux
---

**Автоматический setup-repos.shскрипт для настройки репозиториев в ваших производных системах RHEL или Debian**
1. Ставим `curl` если нету
```
apt install curl -y
```
1. Скачиваем скрипт
```
curl -o setup-repos.sh https://raw.githubusercontent.com/webmin/webmin/master/setup-repos.sh
```
3. Запускаем и устанавливаем
```
sh setup-repos.sh && apt install webmin -yy
```
*Либо*
1. Нужно добавить репозиторий Webmin, для этого редактируем файл с репозиториями:
`sudo nano /etc/apt/sources.list`
 2) В конец добавляйте эти 2 строки и сохраняете через CTRL+X:
```
deb http://download.webmin.com/download/repository sarge contrib

deb [signed-by=/usr/share/keyrings/debian-webmin-developers.gpg] https://download.webmin.com/download/newkey/repository stable contrib
```
3. Панель будет висеть на порту `1000`

**Сменить пароль Webmin root**
```sh
/usr/share/webmin/changepass.pl /etc/webmin root XXXXXX
```
где XXXXXX – новый пароль