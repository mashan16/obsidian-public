---
tags:
  - linux
  - public
  - free_internet
---
[MTProxy GitHub](https://github.com/TelegramMessenger/MTProxy)

## Installation in Docker
Из статьи [Повышаем стабильность Telegram: поднимаем партизанский MTProxy с Fake TLS / Хабр](https://habr.com/ru/articles/994934/?code=9d4b27970262897967ec76e38551ecc6&state=eqDpNObL1I3psmQevEikIGQE&hl=ru)

### Docker


```
docker run --rm nineseconds/mtg:2 generate-secret --hex memchik.ru
docker run -d -p444:444 --name=mtproto-proxy2 -v proxy-config:/data -e SECRET=ee274ef826f52e3b6e47a9d4cf4bcf4e526d656d6368696b2e7275 telegrammessenger/proxy:latest
```

### Docker compose
1. Убедитесь, что Docker установлен, Если нет — ставим
```
docker --version
curl -fsSL https://get.docker.com | sh
```
2. Проверьте порт 443 *MTProxy с Fake TLS **должен** слушать на порту 443. Если он занят (Nginx/Apache), остановите их:*
```
ss -tulpn | grep 443
systemctl stop nginx
```
3. Генерация секрета с Fake TLS
   Используем образ **nineseconds/mtg**. Генерируем секрет для нужного домена:
   Вывод будет примерно таким: `eedc591e24110c174ffd773d832dd69b1531632e7275`
   📌 **Сохраните этот секрет!** Он содержит префикс `ee` и закодированный домен маскировки
```
docker run --rm nineseconds/mtg:2 generate-secret --hex 1c.ru
```
3. Создаем директорию ,переходим туда, открываем `docker-compose.yml`
```
mkdir /opt/MTProto-docker
cd /opt/MTProto-docker
nano docker-compose.yml
```
4. Копируем туда, конфигурацию и вставляем наш `secret` *из 3 пункта* ,Cntrl+X сохраняем
```
services:
  mtproto-proxy:
    image: nineseconds/mtg:2
    container_name: mtproto-proxy
    restart: unless-stopped
    ports:
      - "443:443"
    command: ["simple-run", "-n", "1.1.1.1", "-i", "prefer-ipv4", "0.0.0.0:443", "ваш_секрет"]
```

```
services:
  mtproto-proxy:
    image: telegrammessenger/proxy
    container_name: mtproto-proxy
    restart: unless-stopped
    ports:
      - "443:443"
    environment:
      - PORT=443
      - SECRET=ee3fdd3027ac6760ba9b01e3281a81f8c231632e7275
      - TAG=
    command: 1.1.1.1
```


4. Запускаем
```
docker compose up -d
```

tg://proxy?server=103.137.251.138&port=443&secret=ee3fdd3027ac6760ba9b01e3281a81f8c231632e7275

### Подключение (генерируем ссылку)
```
tg://proxy?server=IP_Server&port=443&secret=Your_Secret
```
Где:
- `IP_Server` - IP нашего сервера VPS\VDS
- `Your_Secret` - secret созданный на 3 шаге

В итоге у нас получается такая ссылка:
```
tg://proxy?server=102.137.230.133&port=443&secret=ee3fdd3020ac6750ba9b01e3281a81f8c231632e9809
```
Копиуем в ТГ кому надо и подключаемся) 

## Install for debian from binares (не получилось)
### Не получилось запустить Лог ошибки 
```
➜  MTProto git:(master) ✗
➜  MTProto git:(master) ✗ ./objs/bin/mtproto-proxy -u nobody -p 8888 -H 444 -S 22044a0b2ef3b2797b45f577f794a27e --aes-pwd proxy-secret proxy-multi.conf -M 1
[2212230][2026-02-11 08:05:09.347388 local] Invoking engine mtproxy-0.02 compiled at Feb 10 2026 09:01:47 by gcc 12.2.0 64-bit after commit cafc3380a81671579ce366d0594b                    9a8e450827e9
[2212230][2026-02-11 08:05:09.347749 local] config_filename = 'proxy-multi.conf'
[2212230][2026-02-11 08:05:09.348268 local] creating 1 workers
mtproto-proxy: common/pid.c:42: init_common_PID: Assertion `!(p & 0xffff0000)' failed.
[pid 2212230] [time 1770793509]
------- Stack Backtrace -------
./objs/bin/mtproto-proxy(print_backtrace+0x19)[0x56124b5b0bc9]
./objs/bin/mtproto-proxy(extended_debug_handler+0x10)[0x56124b5b0d10]
/lib/x86_64-linux-gnu/libc.so.6(+0x3c050)[0x7f9b7fc5a050]
/lib/x86_64-linux-gnu/libc.so.6(+0x8aeec)[0x7f9b7fca8eec]
/lib/x86_64-linux-gnu/libc.so.6(gsignal+0x12)[0x7f9b7fc59fb2]
/lib/x86_64-linux-gnu/libc.so.6(abort+0xd3)[0x7f9b7fc44472]
/lib/x86_64-linux-gnu/libc.so.6(+0x26395)[0x7f9b7fc44395]
/lib/x86_64-linux-gnu/libc.so.6(+0x34ec2)[0x7f9b7fc52ec2]
./objs/bin/mtproto-proxy(+0x4b921)[0x56124b5b4921]
./objs/bin/mtproto-proxy(engine_init+0xe4)[0x56124b5ab254]
./objs/bin/mtproto-proxy(default_main+0x10f)[0x56124b5ac08f]
/lib/x86_64-linux-gnu/libc.so.6(+0x2724a)[0x7f9b7fc4524a]
/lib/x86_64-linux-gnu/libc.so.6(__libc_start_main+0x85)[0x7f9b7fc45305]
./objs/bin/mtproto-proxy(_start+0x21)[0x56124b57acf1]
[pid 2212230] [time 1770793509] -------------------------------
[pid 2212230] [time 1770793509] mtproxy-0.02 compiled at Feb 10 2026 09:01:47 by gcc 12.2.0 64-bit after commit cafc3380a81671579ce366d0594b9a8e450827e9[pid 2212230] [t                    ime 1770793509]
mtproto-proxy: mtproto/mtproto-proxy.c:2311: mtfront_pre_init: Assertion `parent_pid == real_parent_pid' failed.
[pid 2212231] [time 1770793509]
------- Stack Backtrace -------
./objs/bin/mtproto-proxy(print_backtrace+0x19)[0x56124b5b0bc9]
./objs/bin/mtproto-proxy(extended_debug_handler+0x10)[0x56124b5b0d10]
/lib/x86_64-linux-gnu/libc.so.6(+0x3c050)[0x7f9b7fc5a050]
/lib/x86_64-linux-gnu/libc.so.6(+0x8aeec)[0x7f9b7fca8eec]
/lib/x86_64-linux-gnu/libc.so.6(gsignal+0x12)[0x7f9b7fc59fb2]
/lib/x86_64-linux-gnu/libc.so.6(abort+0xd3)[0x7f9b7fc44472]
/lib/x86_64-linux-gnu/libc.so.6(+0x26395)[0x7f9b7fc44395]
/lib/x86_64-linux-gnu/libc.so.6(+0x34ec2)[0x7f9b7fc52ec2]
./objs/bin/mtproto-proxy(mtfront_pre_init+0x26d)[0x56124b58228d]
./objs/bin/mtproto-proxy(default_main+0x100)[0x56124b5ac080]
/lib/x86_64-linux-gnu/libc.so.6(+0x2724a)[0x7f9b7fc4524a]
/lib/x86_64-linux-gnu/libc.so.6(__libc_start_main+0x85)[0x7f9b7fc45305]
./objs/bin/mtproto-proxy(_start+0x21)[0x56124b57acf1]
[pid 2212231] [time 1770793509] -------------------------------
[pid 2212231] [time 1770793509] mtproxy-0.02 compiled at Feb 10 2026 09:01:47 by gcc 12.2.0 64-bit after commit cafc3380a81671579ce366d0594b9a8e450827e9[pid 2212231] [t                    ime 1770793509]
➜  MTProto git:(master) ✗

```
### Установка
Устанавливаем зависимости
```
apt install git curl build-essential libssl-dev zlib1g-dev xxd -y
```
Клонируем репозиторий
```
mkdir /opt/MTProto 
ccd /opt/MTProto
git clone https://github.com/TelegramMessenger/MTProxy
```
Собираем бинарник
```
make && cd objs/bin
```
### Настройка
1. Получите secret, используемый для подключения к серверам Telegram.
```
curl -s https://core.telegram.org/getProxySecret -o proxy-secret
```
2. Получите текущую конфигурацию telegram. Он может меняться (иногда), поэтому мы рекомендуем обновлять его раз в день.
```
curl -s https://core.telegram.org/getProxyConfig -o proxy-multi.conf
```
 3. Сгенерируйте секрет, который пользователи будут использовать для подключения к вашему прокси.
```
head -c 16 /dev/urandom | xxd -ps
```
4. Запуска proxy
```
./mtproto-proxy -u nobody -p 8888 -H 443 -S <secret> --aes-pwd proxy-secret proxy-multi.conf -M 1
```

Ииии что-то пошло не так(