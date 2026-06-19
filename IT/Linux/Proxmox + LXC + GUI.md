---
tags:
  - шаблон
---
[Браузер в контейнере \| Proxmox + LXC + GUI - YouTube](https://www.youtube.com/watch?v=y4QU6KhwwDo)
[[Sudo установка в Debian LXC]]

Обновляем пакеты и устанавливаем недостающие:
```bash
apt update && apt upgrade -y
apt install -y firefox-esr x11vnc xvfb fluxbox git websockify mc wget
git clone https://github.com/novnc/noVNC.git /opt/novnc
```

Переименовываем для удобства
Включаем иксы:
```bash
Xvfb :1 -screen 0 1920x1080x24 &
export DISPLAY=:1
```

Проверяем:
```bash
echo $DISPLAY
```

Запускаем оконный менеджер:
```bash
fluxbox &
```

Запускаем VNC:
```bash
x11vnc -display :1 -nopw -forever -bg
```
Доступен по IP_контейнера:5900

Запускаем noVNC:
```bash
websockify -D --web /opt/novnc 80 localhost:5900
```
Доступен по http://IP_контейнера/

АВТОМАТИЗАЦИЯ Создаём скрипт

``nano start.sh``

Копируем, вставляем:

```bash
#!/bin/bash
Xvfb :1 -screen 0 1920x1080x24 &
sleep 2


export DISPLAY=:1W
fluxbox &

x11vnc -display :1 -nopw -forever -bg -shared
sleep 2

websockify -D --web /opt/novnc 80 localhost:5900

while true; do
    /usr/bin/firefox-esr
    sleep 2
done &

wait
```

# Делаем файл исполняемым:
chmod +x start.sh

# Добавляем в cron:
`crontab -e`
# Добавляем строку:
@reboot /root/start.sh

Перезапуск сервера 
```bash
reboot now
```

# Установка chrome:
wget https://dl.google.com/linux/direct/go...
dpkg -i google-chrome-stable_current_amd64.deb
apt --fix-broken install -y
dpkg -i google-chrome-stable_current_amd64.deb

В скрипте поменять на google-chrome-stable --no-sandbox