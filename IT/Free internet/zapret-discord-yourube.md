---
tags:
  - windows
  - free_internet
  - public
---
[Flowseal/zapret-discord-youtube](https://github.com/Flowseal/zapret-discord-youtube)
[Releases · Flowseal/zapret-discord-youtube](https://github.com/Flowseal/zapret-discord-youtube/releases)

## Краткие описания файлов
- [**`service.bat`**](https://github.com/Flowseal/zapret-discord-youtube/blob/main/service.bat) - установка в автозапуск и другие функции:
    - **`Install Service`** - установка любой стратегии в автозапуск (services.msc)
    - **`Remove Services`** - удаление стратегии и WinDivert из служб
    - **`Check Status`** - проверка статуса обхода и служб (стратегии на автозапуске и WinDivert)
    - **`Game Filter`** - переключение режима обхода для игр (и других сервисов, использующих UDP и TCP на портах выше 1023).  
        **После переключения требуется перезапуск стратегии.**  
        В скобках указан текущий статус (включено/выключено).
    - **`IPSet Filter`** - переключение режима обхода сервисов из `ipset-all.txt`.  
        Полезно при тестировании, если не работает ресурс, который без zapret работает  
        В скобках указан текущий статус:
        - `none` - никакие айпи не попадают под проверку
        - `loaded` - айпи проверяется на вхождение в список
        - `any` - любой айпи попадает под фильтр
    - **`Auto-Update Check`** - Вкл/Выкл автоматическую проверку на обновления
    - **`Update IPSet List`** - обновление списка `ipset-all.txt` актуальным из репозитория
    - **`Update Hosts File`** - обновление файла hosts **для починки подключения к голосовому чату Discord**
    - **`Check for Updates`** - проверка на обновления
    - **`Run Diagnostics`** - диагностика на распространённые причины, по которым zapret может не работать.  
        В конце можно очистить кэш `Discord`, что может помочь, если он неожиданно перестал работать
    - **`Run Tests`** - запуск утилиты для проверки стратегий на работоспособность:
        - `Standard tests` - проверка сайтов из `utils/targets.txt`
        - `DPI checkers` - проверка DPI на различных провайдерах (Cloudflare, Amazon и др.)

## Добавление адресов прочих ресурсов
[## Добавление адресов прочих ресурсов](https://github.com/Flowseal/zapret-discord-youtube#️добавление-адресов-прочих-ресурсов)

Список адресов для обхода можно расширить, добавляя их в:
- [`list-general.txt`](https://github.com/Flowseal/zapret-discord-youtube/blob/main/lists/list-general.txt) для доменов (поддомены автоматически учитываются)
- [`list-exclude.txt`](https://github.com/Flowseal/zapret-discord-youtube/blob/main/lists/list-exclude.txt) для исключения доменов (например, если айпи сети указан в `ipset-all.txt`, но конкретный домен из этой сети не надо фильтровать)
- [`ipset-all.txt`](https://github.com/Flowseal/zapret-discord-youtube/blob/main/lists/ipset-all.txt) для IP и подсетей
- [`ipset-exclude.txt`](https://github.com/Flowseal/zapret-discord-youtube/blob/main/lists/ipset-exclude.txt) для исключения IP и подсетей