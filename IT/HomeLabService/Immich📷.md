---
tags:
  - public
  - linux
  - Docker
  - home_soft
---
Обновление #Docker [Статья](https://wildserver.ru/category/soft/immich-install/)
1. Обновляем пакеты ОС
```
apt update && apt upgrade -y && apt autoremove -y
```
2. Переходим в каталог с приложением immich
```
cd /root/immich-app/
```
3. Обновляем файл конфигурации, переименовать и удалить старый файл на всяк случай
```
mv docker-compose.yml docker-compose.yml.$(date +%d-%m-%Y).bak
```
4. Скачать актуальный `docker-compose.yml` из последнего релиза Immich на GitHub
```
wget https://github.com/Immich-app/Immich/releases/latest/download/docker-compose.yml
```
5. обновляет образы из `docker-compose.yml` и затем запускает контейнеры в фоне
```
docker compose pull && docker compose up -d
```
6. Очищаем старые образы docker
```
docker system prune -a -f
```

 