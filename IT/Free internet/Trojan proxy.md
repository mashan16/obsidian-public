---
tags:
  - free_internet
  - public
  - linux
---
## Назначение и принцип работы 
**Что:** [Trojan](https://github.com/p4gefau1t/trojan-go) — протокол прокси-сервера, маскирующий трафик под обычный HTTPS.

**Зачем:** ТСПУ (DPI-система блокировок РФ) детектирует прокси по паттернам — открытый HTTP CONNECT (Squid/3proxy), нетипичные TLS-фингерпринты, кастомные заголовки. Trojan от этого уходит: снаружи выглядит как заход на легитимный HTTPS-сайт.

**Как работает:**
1. Клиент подключается к серверу по TLS 1.3 на 443 порт — как при обычном заходе на сайт.
2. Внутри TLS-сессии клиент передаёт пароль (хеш SHA224) + команду (куда проксировать).
3. Если пароль верный — сервер открывает туннель и гонит трафик дальше в интернет.
4. Если пароля нет или он неверный (например, ТСПУ или сканер тыкается без авторизации) — сервер отдаёт **fallback**: реальную статичную страницу через nginx. Для стороннего наблюдателя сервер — обычный сайт, ничего больше.

**Ключевое отличие от Squid/3proxy:** там прокси-протокол виден снаружи (CONNECT-паттерн, иногда plaintext заголовки). У Trojan снаружи только TLS-handshake к серверу с действительным сертификатом — отличить от настоящего HTTPS-сайта нечем, а активное зондирование натыкается на fallback.

## Требуемые ресурсы
**Сервер:**
- VPS с ОС Debian
- Root-доступ по SSH
- Открытый порт 443/tcp снаружи (если что-то уже занимает — освободить или развести)
**Домен:**
- Если есть используем свой, если нет бесплатные
**Бесплатные DDNS сервисы:**
- **DuckDNS** (`*.duckdns.org`) — самый простой, поддерживает A-запись на твой IP, бесплатно навсегда
- No-IP — до 3 хостов бесплатно, но требует подтверждения каждые 30 дней (иначе домен отбирают)
- FreeDNS (afraid.org) — 5 общих хостнеймов бесплатно, без ограничения по доменам на аккаунт
**Сертификат:**
- Let's Encrypt через certbot (бесплатно, автообновление) — нужен только открытый 80 или 443 на момент выпуска
**Fallback-сайт:**
- Любая простая статика — можно использовать существующий шаблон или одну HTML-страницу
- nginx для отдачи fallback (можно облегчённый, без лишних модулей)
**Софт:**
- trojan-go (бинарник, ставится без компиляции) — основной выбор
- systemd для автозапуска
**На клиентской стороне (Windows):**
- NekoRay/NekoBox или аналог — лёгкий локальный клиент
- Расширение Proxy SwitchyOmega в браузере
**Данные, которые понадобятся по ходу:**
- Пароль(и) для Trojan-аутентификации (придумать заранее)
- Email для Let's Encrypt (для уведомлений об истечении серта)

## Описание шагов инструкции (обзор)
Общая последовательность, без деталей команд (детали — в блоке 4):
1. **DNS** — поддомен (`proxy.yan38.ru` или похожий) → A-запись на IP впски
2. **Подготовка сервера** — обновление системы, открытие порта 443
3. **Сертификат** — выпуск Let's Encrypt через certbot на поддомен
4. **Fallback-сайт** — nginx с заглушкой на локальном порту (не 443)
5. **Установка trojan-go** — скачивание бинарника, размещение, права
6. **Конфигурация Trojan** — JSON: порт 443, путь к серту, пароль(и), адрес fallback
7. **Разведение портов** — Trojan слушает 443 снаружи, nginx-fallback доступен только локально/внутренне
8. **Systemd unit** — автозапуск и автостарт при перезагрузке
9. **Firewall** — открыть 443/tcp, закрыть всё лишнее
10. **Проверка сервера** — fallback отдаётся без пароля, туннель работает с паролем
11. **Клиент на Windows** — установка NekoRay/аналога, конфиг с адресом сервера и паролем
12. **SwitchyOmega** — настройка SOCKS5 на локальный порт клиента
13. **Финальная проверка** — IP меняется, ТСПУ не режет, fallback не выдаёт прокси

## Установка, настройка и проверка серверной части
### 1. Добавляем  DNS запись у регистратора домена
Нужно создать **A-запись**, указывающую поддомен на IP-адрес VPS.
**Пример:**

| Тип записи | Имя (Host) | Public IP VPS | TTL  |
| ---------- | ---------- | ------------- | ---- |
| A          | proxy      | 203.0.113.10  | 3600 |

Это создаст `proxy.example.com → 203.0.113.10`.
- В поле имени/хоста — только поддомен (`proxy`), без `.example.com` — большинство панелей дописывают домен сами
- IP — публичный IPv4-адрес сервера (узнать командой `curl ifconfig.me` на самом сервере)
**Проверка после прописки** 
С сервера
```bash
dig +short proxy.example.com
```
С компа на винде 
```powershell
Resolve-DnsName proxy.example.com -Server 8.8.8.8
```

Должен вернуться тот же IP. Если пусто или старый IP — DNS ещё не обновился (может занять от нескольких минут до пары часов, зависит от TTL и регистратора).
### 2. Подготовка системы
```
apt update && apt upgrade -y
apt install -y curl nginx certbot
```
### 3. Сертификат Let's Encrypt
Временно нужен открытый 80 порт для HTTP-01 валидации:
```
certbot certonly --standalone -d <DOMAIN> --agree-tos -m <твой_email> --no-eff-email
```
Сертификат окажется в `/etc/letsencrypt/live/<DOMAIN>/fullchain.pem` и `privkey.pem`.
### 4. Fallback-сайт
nginx слушает локально, не на 443:
```
cat > /etc/nginx/sites-available/fallback <<'EOF'
server {
    listen 127.0.0.1:8080;
    server_name <DOMAIN>;
    root /var/www/html;
    index index.html;
}
EOF
ln -s /etc/nginx/sites-available/fallback /etc/nginx/sites-enabled/
nginx -t && systemctl restart nginx
```
В `/var/www/html/index.html` — любая статика (заглушка, страница-визитка, что угодно нейтральное).
### 5. Установка trojan-go
```
curl -L -o trojan-go.zip https://github.com/p4gefau1t/trojan-go/releases/latest/download/trojan-go-linux-amd64.zip
apt install -y unzip
unzip trojan-go.zip -d /opt/trojan-go
```
### 6. Конфиг Trojan
```
cat > /opt/trojan-go/config.json <<EOF
{
    "run_type": "server",
    "local_addr": "0.0.0.0",
    "local_port": 443,
    "remote_addr": "127.0.0.1",
    "remote_port": 8080,
    "password": ["<твой_пароль>"],
    "ssl": {
        "cert": "/etc/letsencrypt/live/<DOMAIN>/fullchain.pem",
        "key": "/etc/letsencrypt/live/<DOMAIN>/privkey.pem",
        "sni": "<DOMAIN>"
    }
}
EOF
```
`remote_addr/remote_port` — это и есть fallback (nginx на 8080).
### 7. Systemd unit
```
cat > /etc/systemd/system/trojan-go.service <<'EOF'
[Unit]
Description=trojan-go
After=network.target

[Service]
ExecStart=/opt/trojan-go/trojan-go -config /opt/trojan-go/config.json
Restart=on-failure

[Install]
WantedBy=multi-user.target
EOF
systemctl daemon-reload
systemctl enable --now trojan-go
```

### 8. Firewall
```
ufw allow 443/tcp
ufw allow 80/tcp
```
(80 можно закрыть после выпуска серта, если не используешь certbot renew с standalone-методом — тогда лучше держать открытым или переключиться на webroot/nginx-плагин)

### 9. Проверка
**Fallback без пароля** (с другой машины, без прокси):
```
curl -v https://<DOMAIN>
```
Должен прийти HTML с заглушки, не ошибка.

**Статус сервиса**
```
systemctl status trojan-go --no-pager
```

**Лог**
```
journalctl -u trojan-go -n 50 --no-pager
```
Если статус active и fallback отдаётся нормально — сервер готов, дальше клиент (блок 5).

## Клиентская часть

### Windows
**Список клиентов:**

| Клиент                                          | Тип | Заметки                                                           |
| ----------------------------------------------- | --- | ----------------------------------------------------------------- |
| [NyameBox](https://github.com/qr243vbi/nekobox) | GUI | Самый удобный, поддерживает Trojan/VLESS/SS, автоконфиг по ссылке |
| [v2rayN](https://github.com/2dust/v2rayN)       | GUI | Популярный, плагин для Trojan                                     |



Рекомендация: **NekoRay**.
**Настройка:**
1. Скачать с GitHub releases
2. Новый профиль → тип Trojan
3. Адрес `<DOMAIN>`, порт 443, пароль с сервера, SNI `<DOMAIN>`
4. Режим — **только локальный прокси** (Socks/Mixed), не системный VPN/TUN
5. Запустить — поднимется локальный SOCKS5/HTTP (например `127.0.0.1:2080`)

**SwitchyOmega:**
- Тип: SOCKS5, сервер `127.0.0.1`, порт из NekoRay
- Применить профиль — браузерный трафик пошёл через туннель

**Проверка:**
Linux
```
curl --socks5 127.0.0.1:2080 https://ifconfig.me
```
Windows
```powershell
curl.exe --socks5 127.0.0.1:2080 https://ifconfig.me
```
IP должен быть впски.

---
#### Android
**Список клиентов:**

| Клиент                                                                  | Заметки                      |
| ----------------------------------------------------------------------- | ---------------------------- |
| [v2rayNG](https://github.com/2dust/v2rayNG)                             | Популярный, много настроек   |
| [NekoBox for Android](https://github.com/MatsuriDayo/NekoBoxForAndroid) | Аналог NekoRay, удобнее всех |
Ставятся через APK с GitHub releases (в Google Play часто отсутствуют).

**Особенность Android:** браузер не даёт вручную указать локальный прокси без root/расширений. Поэтому:
- Клиент поднимает **системный VPN-интерфейс** (режим, встроенный в NekoBox/v2rayNG) — весь трафик устройства идёт через Trojan
- Это не "только браузер", а всё устройство — но на практике это стандартный и самый простой путь для Android

**Настройка:** аналогично Windows — адрес, порт 443, пароль, SNI — но включить VPN-режим вместо локального SOCKS5.