---
tags:
  - public
  - windows
  - linux
  - backup
  - sync
  - syncthing
title:
---
Хорошая статья про Syncthing на [Remontra.pro](https://remontka.pro/syncthing)


# Windows + Автозапуск

- Скачиваем последнюю версию с [офф сайта ](https://syncthing.net/downloads/)  
- А лучше сразу с [GitHub](https://github.com/syncthing/syncthing/releases?q=&expanded=true) ищем тег `Latest` и архитектуру `syncthing-windows-amd64`
![[Pasted image 20250728154821.png]]
---
## Автозапуск
Оригинальная [статья](https://b.tinyops.ru/syncthing-as-windows-service)
Официальное руководство [предлагает](https://docs.syncthing.net/users/autostart.html) два способа:
1. Запускать через планировщик задач Windows (Проще)
-  *Cпособ  костыльный и перестанет работать если пользователь поменяет пароль, поэтому лучше настроить службу Windows*
1. Запускать в виде службы (Чуть сложнее)
- *Если антивирус  ругаться на NSSM, добавляем в исключение*
---
## Запускать через планировщик задач Windows
1. Открываем Powershell **от имени администратора**
2. Вставляем команду которая создаст нам папку `syncthing` в  `C:\Program Files` и создаст задание в планировщике на автозапуск
```
New-Item -Path "C:\Program Files\syncthing" -ItemType Directory -Force ; Register-ScheduledTask -TaskName "Syncthing run" -Trigger (New-ScheduledTaskTrigger -AtLogOn) -Action (New-ScheduledTaskAction -Execute "C:\Program Files\syncthing\syncthing.exe" -Argument "--no-console --no-browser") -Principal (New-ScheduledTaskPrincipal -UserId $env:USERNAME -LogonType Interactive) -Settings (New-ScheduledTaskSettingsSet -ExecutionTimeLimit (New-TimeSpan -Hours 72)) -Description "Start Syncthing at logon"
```
4. Распаковываем содержимое архива где находится syncthing.exe в `C:\Program Files\syncthing`
5. Адрес web панели после запуска
```
http://127.0.0.1:8384
```

## Запускать в виде службы
Для настройки необходимо использовать утилиту которая умеет запускать Windows-приложения как службы.
Подобных утилит достаточно много, но мы возьмём одну из самых удобных - NSSM (The Non-Sucking Service Manager). 
- Скачиваем с официального сайта дистрибутив NSSM [nssm.cc/download](https://nssm.cc/download) ([Прямая ссылка](https://nssm.cc/release/nssm-2.24.zip)) [[nssm-2.24.zip]]
1. Распаковываем NSSM  в `c:\soft\nssm`
2. Распаковываем Syncthing в `c:\soft\Sycnthing`
- Полный путь до исполняемого файла Syncthing у нас получился такой - `c:\apps\syncthing\syncthing.exe`
- А до исполняемого файла nssm такой - `c:\soft\nssm\win64\nssm.exe`

**Создание службы Windows**
Запускаем командную строку Windows с правами администратора и создаём службу:
```shell
c:\soft\nssm\win64\nssm.exe install Syncthing c:\soft\Synthin\syncthing.exe
c:\soft\nssm\win64\nssm.exe set Syncthing AppDirectory c:\soft\Synthin\syncthing
c:\soft\nssm\win64\nssm.exe set Syncthing start SERVICE_DELAYED_AUTO_START
```
**Настраиваем пользователя**
В целях безопасности рекомендуется запускать Syncthing от имени пользователя без привилегий администратора.
Заходим в оснастку служб Windows, переходим в свойств службы `Syncthing`.
![[Pasted image 20250728155831.png]]
Выбираем существующего пользователя и указываем пароль.
![[Pasted image 20250728155842.png]]
Жмём "ОК" и перезапускаем службу.

Приложение станет доступно по ссылке [http://127.0.0.1:8384](http://127.0.0.1:8384/).

# Установка на Debian через apt
Официальная инструкция от разрабов Syncthing [apt.syncthing.net](https://apt.syncthing.net)

Запускаем все из под Root без sudo

- Обновляем и устанавливаем все пакеты в ОС
- Ставим Curl
- Создаём директорию для ключа
- Чтобы система могла проверить подлинность пакетов, необходимо предоставить ключ выпуска.
- Добавляем канал stable-v2 в APT
- Обновляем пакеты
- Устанавливаем syncthing
- Создаем пользователя syncthing под который и будет запускаться наш Syncthing
- Изменяем конфиг syncthing, иначе не сможем попасть в веб интерфейс по 192.168.1.9 а только по 127.0.0.1
-  `systemctl` Включаем автозапуск, Стартуем, проверяем статус  (если  `active (running)` то все отлично)
-  Готово! syncthing теперь доступен по IP адресу нашего хоста например: http://192.168.1.9:8384
```
apt update && apt upgrade -y
apt install curl
mkdir -p /etc/apt/keyrings
curl -L -o /etc/apt/keyrings/syncthing-archive-keyring.gpg https://syncthing.net/release-key.gpg
echo "deb [signed-by=/etc/apt/keyrings/syncthing-archive-keyring.gpg] https://apt.syncthing.net/ syncthing stable-v2" | tee /etc/apt/sources.list.d/syncthing.list
apt update
apt install syncthing
useradd -r -m -U -s /usr/sbin/nologin -c "Syncthing User" syncthing
sed -i 's|<address>127\.0\.0\.1:8384</address>|<address>0.0.0.0:8384</address>|' /var/lib/syncthing/.config/syncthing/config.xml
systemctl enable syncthing@syncthing.service
systemctl start syncthing@syncthing.service
systemctl status syncthing@syncthing.service
```
