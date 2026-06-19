
## Проблема
При активном WG с прописанным `DNS = ...` в конфиге — весь DNS-трафик клиента
уходит на DNS из конфига, локальные/домашние имена (например `*.yan38.ru`)
резолвятся неверно (в публичный IP вместо локального).

Простой список из двух DNS-серверов не работает как fallback по "не нашёл домен" —
Windows переключается на второй сервер только если первый недоступен
(таймаут), а не если ответил NXDOMAIN.

## Решение — NRPT (policy-based DNS)

1. В конфиге WireGuard убрать строку `DNS = ...` полностью.
2. В PowerShell (от администратора):
```powershell
Add-DnsClientNrptRule -Namespace ".intera-gk.ru" -NameServers "172.10.10.9","77.88.8.8"
```
   - Запросы к `*.intera-gk.ru` → идут на рабочий DNS (172.10.10.9, фоллбэк 77.88.8.8)
   - Все остальные домены → как обычно, через DNS текущей сети (роутер)

2. Проверка правила:
```powershell
Get-DnsClientNrptRule
```
3. Удалить все правила (если нужно откатить):
```powershell
Get-DnsClientNrptRule | Remove-DnsClientNrptRule
```

## Сопутствующая проблема — OpenWrt rebind protection
Если дома резолвишь `*.yan38.ru` на локальный IP через "Записи DNS" в LuCI,
по умолчанию dnsmasq отбрасывает такие ответы (защита от DNS rebinding).

Фикс: **Службы → DNS → Фильтр → Белый список доменов** → добавить `yan38.ru`.
## Диагностика
```powershell
nslookup имя.домен          # видно, какой DNS-сервер ответил
nslookup имя.домен IP_DNS   # прямой запрос к конкретному серверу
ipconfig /flushdns          # Сбросить кэш DNS на ПК windows
```
Проверить и в браузере: secure DNS (DoH) может резолвить мимо системного DNS —
выключить в `chrome://settings/security` или Firefox `about:preferences#general`.