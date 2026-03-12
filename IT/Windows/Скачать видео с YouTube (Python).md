---
tags:
  - public
  - windows
  - powershell
  - youtube
  - script
---
==! Инструкция не закончена!==

yt-dlp.exe -f bestvideo+bestaudio --merge-output-format mp4


Статьи на habr [Часть 1](https://habr.com/ru/articles/870110/)  [Часть 2](https://habr.com/ru/articles/871202/) *Обвиваем YouTube змеем, или как смотреть и скачивать видео с YouTube без VPN на чистом Python-е.*

Запускаем *PowerShell* или *Терминал* от имени администратора 
**Создаем папку C:\yt_dw и пустые файлы в ней**
```
New-Item -Path "C:\yt_dw" -ItemType Directory -Force; New-Item -Path "C:\yt_dw\blacklist.txt", "C:\yt_dw\yt_downloader.py", "C:\yt_dw\yt_downloader2.py" -ItemType File -Force
```
**Копируем в blacklist.txt список доменов Youtube**
```
youtube.com
youtu.be
yt.be
googlevideo.com
ytimg.com
ggpht.com
gvt1.com
youtube-nocookie.com
youtube-ui.l.google.com
youtubeembeddedplayer.googleapis.com
youtube.googleapis.com
youtubei.googleapis.com
yt-video-upload.l.google.com
wide-youtube.l.google.com
```
**Устанавливаем Python**
```
winget install 9NQ7512CXL7T
```
### Копируем код в yt_downloader.py (Скачать видео в 360p)
```python
import random
import asyncio

BLOCKED = [line.rstrip().encode() for line in open('blacklist.txt', 'r', encoding='utf-8')]
TASKS = []
async def main(host, port):

    server = await asyncio.start_server(new_conn, host, port)
    await server.serve_forever()
async def pipe(reader, writer):

    while not reader.at_eof() and not writer.is_closing():
        try:
            writer.write(await reader.read(1500))
            await writer.drain()
        except:
            break

    writer.close()
async def new_conn(local_reader, local_writer):

    http_data = await local_reader.read(1500)

    try:
        type, target = http_data.split(b"\r\n")[0].split(b" ")[0:2]
        host, port = target.split(b":")
    except:
        local_writer.close()
        return

    if type != b"CONNECT":
        local_writer.close()
        return

    local_writer.write(b'HTTP/1.1 200 OK\n\n')
    await local_writer.drain()

    try:
        remote_reader, remote_writer = await asyncio.open_connection(host, port)
    except:
        local_writer.close()
        return

    if port == b'443':
        await fragemtn_data(local_reader, remote_writer)

    TASKS.append(asyncio.create_task(pipe(local_reader, remote_writer)))
    TASKS.append(asyncio.create_task(pipe(remote_reader, local_writer)))
async def fragemtn_data(local_reader, remote_writer):

    head = await local_reader.read(5)
    data = await local_reader.read(1500)
    parts = []

    if all([data.find(site) == -1 for site in BLOCKED]):
        remote_writer.write(head + data)
        await remote_writer.drain()

        return

    while data:
        part_len = random.randint(1, len(data))
        parts.append(bytes.fromhex("1603") + bytes([random.randint(0, 255)]) + int(
            part_len).to_bytes(2, byteorder='big') + data[0:part_len])

        data = data[part_len:]

    remote_writer.write(b''.join(parts))
    await remote_writer.drain()
asyncio.run(main(host='127.0.0.1', port=8881))
```




**Устанавливаем библиотеку pytubefix** 
```
py -m pip install pytubefix
```

