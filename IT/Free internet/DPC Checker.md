---
tags:
  - linux
  - free_internet
  - public
---

## DPI Detector by [Runnin4ik](https://github.com/Runnin4ik)[[]]
Репозиторий https://github.com/Runnin4ik/dpi-detector

Инструмент для анализа цензуры трафика в России: обнаруживает и классифицирует блокировки сайтов, хостингов и CDN (TCP16-20 блокировки), а также подмену DNS-запросов провайдером.

```shell
docker run --rm -it --pull=always ghcr.io/runnin4ik/dpi-detector:latest
```
Или запускайте с указанием определенной версии  
Это избавляет от постоянных скачиваний, но нужно следить за актуальностью версий
```shell
docker run --rm -it ghcr.io/runnin4ik/dpi-detector:3.1.0
```

## dpi-checkers by [hyperion-cs](https://github.com/hyperion-cs)
Репозиторий [github\hyperion-cs\dpi-checkers](https://github.com/hyperion-cs/dpi-checkers)

В этом репозитории содержатся шашки, которые позволяют определить, есть ли у вашего жилого интернет-провайдера DPI, а также конкретные методы (и их параметры) цензуры, используемые для ограничений.

Ставим `unzip` проверяем действующие ссылки
```sh
apt install unzip -y
curl -s https://api.github.com/repos/hyperion-cs/dpi-checkers/releases/latest | grep browser_download_url
```
Скачиваем  последнюю версию для `linux-amd64` (в моём случаем это `v0.3.1`)
Распаковываем, удаляем архив, запускаем
```
curl -L -o dpich.zip https://github.com/hyperion-cs/dpi-checkers/releases/download/dpich-v0.3.1/dpich-v0.3.1-linux-amd64.zip
unzip dpich.zip
rm dpich.zip
./dpich
```
Далее запуск уже командой
```sh
./dpich
```
Вернутся назад в меню клавиша `m`
