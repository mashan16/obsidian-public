---
tags:
  - linux
  - public
  - free_internet
---
Репозитории Github 
Telment: https://github.com/telemt/telemt/tree/main
Telment_panel: https://github.com/amirotin/telemt_panel

## Telemt через Systemd
Из офф доки на гитхаб
Скачали, переместили бинарник, дали права на исполнение, создали юзера, создали директорию для конфигурации и пустышку
```sh
apt install curl sudo
wget -qO- "https://github.com/telemt/telemt/releases/latest/download/telemt-$(uname -m)-linux-$(ldd --version 2>&1 | grep -iq musl && echo musl || echo gnu).tar.gz" | tar -xz
mv telemt /bin
chmod +x /bin/telemt
useradd -d /opt/telemt -m -r -U telemt
mkdir /etc/telemt
chown -R telemt:telemt /etc/telemt
```
Создаём конфигурацию для telemt c генерацией `secret`
```sh
SECRET=$(openssl rand -hex 16) && cat > /etc/telemt/telemt.toml << EOF
# === General Settings ===
[general]
# ad_tag = "00000000000000000000000000000000"
use_middle_proxy = false

[general.modes]
classic = false
secure = false
tls = true

[server]
port = 443

[server.api]
enabled = true
# listen = "127.0.0.1:9091"
# whitelist = ["127.0.0.1/32"]
# read_only = true

# === Anti-Censorship & Masking ===
[censorship]
tls_domain = "vk.com"

[access.users]
# format: "username" = "32_hex_chars_secret"
hello = "$SECRET"
EOF
```
Создаём службу
```sh
cat > /etc/systemd/system/telemt.service << EOF
[Unit]
Description=Telemt
After=network-online.target
Wants=network-online.target

[Service]
Type=simple
User=telemt
Group=telemt
WorkingDirectory=/opt/telemt
ExecStart=/bin/telemt /etc/telemt/telemt.toml
Restart=on-failure
LimitNOFILE=65536
AmbientCapabilities=CAP_NET_ADMIN CAP_NET_BIND_SERVICE
CapabilityBoundingSet=CAP_NET_ADMIN CAP_NET_BIND_SERVICE
NoNewPrivileges=true

[Install]
WantedBy=multi-user.target
EOF
```
Устанавливаем панель, добавляем пользователя `telemt-panel` в группу `telemt`
Запускаем, включаем службы на автозапуск, проверяем
```sh
curl -fsSL https://raw.githubusercontent.com/amirotin/telemt_panel/main/install.sh | bash
usermod -aG systemd-journal telemt-panel
usermod -aG telemt telemt-panel
chmod -R g+w /etc/telemt/
systemctl daemon-reload && systemctl enable telemt telemt-panel
systemctl restart telemt telemt-panel && systemctl status telemt telemt-panel
```





