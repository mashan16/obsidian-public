---
tags:
  - linux
  - free_internet
  - public
---
[GitHub - Flowseal/tg-ws-proxy: Local MTProto proxy server for partial bypassing of Telegram loading · GitHub](https://github.com/Flowseal/tg-ws-proxy)

## Update tg-ws-proxy (debian CLI systemd)

[[Скрипт для обновления]] (еще не проверял)

1. Клонируем новую версию
```bash
mkdir /opt/tg-ws-proxy-X.X.X
cd /opt/tg-ws-proxy-X.X.X
git clone --branch vX.X.X --depth 1 https://github.com/Flowseal/tg-ws-proxy.git .
# Точка в конце обязательна — клонирует в текущую папку, а не в подпапку
```
2. Создаём venv и устанавливаем зависимости
```bash
python3 -m venv venv
# requirements.txt может отсутствовать (как в 1.7.0) — ставим через pyproject.toml
/opt/tg-ws-proxy-X.X.X/venv/bin/pip install -e /opt/tg-ws-proxy-X.X.X/
```
3. Создаём systemd-сервис
```bash
cat > /etc/systemd/system/tg-ws-proxy-X.X.X.service << 'EOF'
[Unit]
Description=tg-ws-proxy X.X.X
After=network.target

[Service]
Type=simple
User=root
Group=root
WorkingDirectory=/opt/tg-ws-proxy-X.X.X/
ExecStart=/opt/tg-ws-proxy-X.X.X/venv/bin/python proxy/tg_ws_proxy.py --host 0.0.0.0 --port XXXX --secret ВАШ_СЕКРЕТ
Restart=on-failure

[Install]
WantedBy=multi-user.target
EOF

systemctl daemon-reload
systemctl enable --now tg-ws-proxy-X.X.X.service
```
 4. Проверяем
```bash
sleep 5 && systemctl status tg-ws-proxy-X.X.X.service
ss -tlnp | grep XXXX
```
## Install tg-ws-proxy (debian CLI systemd)
Отталкивался от инструкции [Vps · Flowseal/tg-ws-proxy · Discussion #106 · GitHub](https://github.com/Flowseal/tg-ws-proxy/discussions/106#discussioncomment-16178233)

- Ставим зависимости, cоздаём директорию, клонируем репозиторий
- Создаём и включаем venv окружение, собираем python пакет
```shell
apt update -y
apt install -y python3 python3-venv python3-pip git
mkdir /opt/tg-ws-proxy
cd /opt/tg-ws-proxy
git clone https://github.com/Flowseal/tg-ws-proxy.git .
python3 -m venv venv
source venv/bin/activate
pip install .
```
Генерируем `secret` и создаем сервис systemd на порту `1080` 
```shell
YOUR_SECRET=$(openssl rand -hex 16) && \
cat <<EOF > /etc/systemd/system/tg-ws-proxy.service
[Unit]
Description=tg-ws-proxy
After=network.target

[Service]
Type=simple
User=root
Group=root
WorkingDirectory=/opt/tg-ws-proxy/
ExecStart=/opt/tg-ws-proxy/venv/bin/python proxy/tg_ws_proxy.py --host 0.0.0.0 --port 1080 --secret $YOUR_SECRET
Restart=on-failure

[Install]
WantedBy=multi-user.target
EOF
```
Включаем, проверяем статус (В статусе будет ссылка для подключения)
```shell
systemctl daemon-reload && systemctl enable tg-ws-proxy && systemctl start tg-ws-proxy && systemctl status tg-ws-proxy
```
Чтобы запустить вручную (`secret` генерируется случайно в каждом запуске, используйте ключ `--secret ваш_secret` для своего секрета)
```
/opt/tg-ws-proxy/venv/bin/python proxy/tg_ws_proxy.py --host 0.0.0.0 --port 1080```
```
