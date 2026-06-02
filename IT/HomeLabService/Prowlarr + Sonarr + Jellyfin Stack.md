[ChatGPT Диалог](https://chatgpt.com/c/6a05f27e-56e0-832f-a33b-e53772e56bcb)

**Краткое описание**
Self-hosted медиа-инфраструктура с разделением сервисов между VPS и домашним Proxmox сервером. VPS используется для поиска торрент-раздач и обхода блокировок, домашний сервер — для скачивания, хранения и просмотра медиаконтента через Jellyfin.

**Схема**
![[ChatGPT Image 15 мая 2026 г., 00_23_18.png]]
## Media Stack — VPS + HomeLab

## VPS (Europe)

### Сервисы
- Prowlarr — поиск торрентов
- FlareSolverr — обход Cloudflare
- WireGuard — VPN домой
- Nginx Proxy Manager — reverse proxy / HTTPS
### Задача
- стабильный доступ к трекерам
- европейский IP
- единая точка поиска торрентов
---

## Home Server (Proxmox)
### Сервисы
- qBittorrent — скачивание торрентов
- Sonarr — автоматизация сериалов
- Radarr — автоматизация фильмов
- Jellyfin — медиасервер
- Bazarr — субтитры
### Задача
- хранение медиа
- скачивание
- просмотр контента

---

## Схема работы

```text
Sonarr / Radarr
        ↓
     Prowlarr (VPS)
        ↓
   Torrent Trackers
        ↓
   qBittorrent (Home)
        ↓
      Jellyfin