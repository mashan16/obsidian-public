---
title: Руководство по производству | Документация PeerTube,en
source: https://docs.joinpeertube.org/install/any-os
author:
published:
created: 2025-11-18
description: Documentation of PeerTube, a free software to take back control of your videos!
tags:
  - linux
  - home_soft
---

# Моя инструкция Docker 
Еще не написана
# С офф сайта , Установка в Docker
Стыбрено [тут](https://docs.joinpeertube.org/install/docker)

Это руководство требует [докер](https://www.docker.com/community-edition) и [Docker-Compose V2](https://docs.docker.com/compose/install/) .

```
docker compose version # Must be > 2.x.x
```

## Установить

**PeerTube не поддерживает смену хоста веб-сервера** . Помните, что ваше доменное имя становится окончательным после первого запуска PeerTube.

#### Перейдите в свой рабочий каталог

ИНФОРМАЦИЯ

В следующем руководстве предполагается, что рабочий каталог пуст, но вы также можете клонировать репозиторий, использовать ветку master и `cd support/docker/production`.

```
cd /your/peertube/directory
```

#### Получите последнюю версию файла Compose

```
curl https://raw.githubusercontent.com/chocobozzz/PeerTube/master/support/docker/production/docker-compose.yml > docker-compose.yml
```

Просмотрите источник файла, который вы собираетесь загрузить: [docker-compose.yml](https://github.com/Chocobozzz/PeerTube/blob/master/support/docker/production/docker-compose.yml)

#### Получите последнюю версию env_file

```
curl https://raw.githubusercontent.com/Chocobozzz/PeerTube/master/support/docker/production/.env > .env
```

Просмотрите источник файла, который вы собираетесь загрузить: [.env](https://github.com/Chocobozzz/PeerTube/blob/master/support/docker/production/.env)

#### Настройте `docker-compose.yml`файл там в соответствии с вашими потребностями

```
sudo nano docker-compose.yml
```

#### Затем настройте `.env`файл для изменения настроек переменных среды

```
sudo nano .env
```

В скачанном примере [.env](https://github.com/Chocobozzz/PeerTube/blob/master/support/docker/production/.env) , необходимо заменить:

- `<MY POSTGRES USERNAME>`
- `<MY POSTGRES PASSWORD>`
- `<MY DOMAIN>`без https://
- `<MY EMAIL ADDRESS>`
- `<MY PEERTUBE SECRET>`

Другие переменные среды используются в [/support/docker/production/config/custom-environment-variables.yaml](https://github.com/Chocobozzz/PeerTube/blob/master/support/docker/production/config/custom-environment-variables.yaml) и может быть интуитивно понятен при использовании.

#### Веб-сервер

ИНФОРМАЦИЯ

Файл компоновки docker включает настроенный веб-сервер. Вы можете пропустить эту часть и прокомментировать соответствующий раздел в Docker Compose, если вы используете другой веб-сервер/прокси.

Установите шаблон, который будет использовать контейнер nginx. Контейнер сгенерирует конфигурацию, заменив `${WEBSERVER_HOST}`и `${PEERTUBE_HOST}`используя ваш докер, создайте env-файл.

```
mkdir -p docker-volume/nginx docker-volume/nginx-logs
curl https://raw.githubusercontent.com/Chocobozzz/PeerTube/master/support/nginx/peertube > docker-volume/nginx/peertube
```

Вам необходимо вручную сгенерировать первый сертификат SSL/TLS с помощью Let's Encrypt:

```
mkdir -p docker-volume/certbot
docker run -it --rm --name certbot -p 80:80 -v "$(pwd)/docker-volume/certbot/conf:/etc/letsencrypt" certbot/certbot certonly --standalone
```

Выделенный контейнер в docker-compose автоматически обновит этот сертификат и перезагрузит nginx.

#### Проверьте свою настройку

_примечание_ : новые версии Compose вызываются с помощью `docker compose`вместо `docker-compose`, поэтому удалите дефис на всех шагах, использующих эту команду, если вы получаете ошибки.

Запустите свои контейнеры:

```
docker compose up
```

#### Получение автоматически сгенерированных учетных данных администратора

Вы можете изменить автоматически созданный пароль для пользователя root, выполнив эту команду из корневого каталога peertube:

```
docker compose exec -u peertube peertube npm run reset-password -- -u root
```

Вы также можете просмотреть журналы контейнера Peertube по умолчанию. `root`пароль. Тебе захочется бежать `docker-compose logs peertube | grep -A1 root`для поиска в выводе журнала учетных данных администратора вашего нового экземпляра PeerTube, которые будут выглядеть примерно так.

```
docker compose logs peertube | grep -A1 root

peertube_1  | [example.com:443] 2019-11-16 04:26:06.082 info: Username: root
peertube_1  | [example.com:443] 2019-11-16 04:26:06.083 info: User password: abcdefghijklmnop
```

#### Получение автоматически сгенерированной записи TXT DNS DKIM

[ДКИМ](https://en.wikipedia.org/wiki/DomainKeys_Identified_Mail) отправка подписи и генерация ключей RSA включены в образе Postfix по умолчанию. `mwader/postfix-relay`с [ОпенДКИМ](http://www.opendkim.org/) .

Бегать `cat ./docker-volume/opendkim/keys/*/*.txt`чтобы отобразить запись DKIM DNS TXT, содержащую открытый ключ для настройки вашего домена:

```
cat ./docker-volume/opendkim/keys/*/*.txt

peertube._domainkey.mydomain.tld.	IN	TXT	( "v=DKIM1; h=sha256; k=rsa; "
	  "p=MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEA0Dx7wLGPFVaxVQ4TGym/eF89aQ8oMxS9v5BCc26Hij91t2Ci8Fl12DHNVqZoIPGm+9tTIoDVDFEFrlPhMOZl8i4jU9pcFjjaIISaV2+qTa8uV1j3MyByogG8pu4o5Ill7zaySYFsYB++cHJ9pjbFSC42dddCYMfuVgrBsLNrvEi3dLDMjJF5l92Uu8YeswFe26PuHX3Avr261n"
	  "j5joTnYwat4387VEUyGUnZ0aZxCERi+ndXv2/wMJ0tizq+a9+EgqIb+7lkUc2XciQPNuTujM25GhrQBEKznvHyPA6fHsFheymOuB763QpkmnQQLCxyLygAY9mE/5RY+5Q6J9oDOQIDAQAB" )  ; ----- DKIM key peertube for mydomain.tld
```

#### Пароль администратора

Посмотреть руководство по производству [Раздел «Администратор»](https://docs.joinpeertube.org/install/any-os#administrator)

#### Что теперь?

Посмотреть руководство по производству [Раздел «Что теперь»](https://docs.joinpeertube.org/install/any-os#what-now) .

## Обновление

ПРЕДУПРЕЖДЕНИЕ

Проверьте журнал изменений (в частности, _ВАЖНЫЕ ПРИМЕЧАНИЯ_ раздел): [https://github.com/Chocobozzz/PeerTube/blob/develop/CHANGELOG.md](https://github.com/Chocobozzz/PeerTube/blob/develop/CHANGELOG.md)

Загрузите последние изображения:

```
cd /your/peertube/directory
docker compose pull
```

Остановитесь, удалите контейнеры и внутренние тома (чтобы сделать недействительными статические клиентские файлы, совместно используемые `peertube`и `webserver`контейнеры):

```
docker compose down -v
```

Обновите конфигурацию nginx:

```
mv docker-volume/nginx/peertube docker-volume/nginx/peertube.bak
curl https://raw.githubusercontent.com/Chocobozzz/PeerTube/master/support/nginx/peertube > docker-volume/nginx/peertube
```

Перезапустите PeerTube:

```
docker compose up -d
```

## Обновить контейнер PostgreSQL

Если вы хотите обновить версию контейнера PostgreSQL (например, потому, что ваша текущая версия больше не будет поддерживаться), вам необходимо запланировать время простоя (чтобы экспортировать текущий кластер и повторно импортировать данные в новый).

Когда вы будете готовы, зайдите в каталог создания докера PeerTube:

```
cd /docker-compose/directory
```

Подготовьте каталог резервных копий и остановите все контейнеры, кроме базы данных:

```
mkdir -p backups
docker compose stop peertube webserver certbot
```

Зайдите внутрь контейнера базы данных:

```
docker compose exec -it postgres /bin/bash
```

И экспортируйте базу данных (заменять не нужно `$POSTGRES_*`переменные, они автоматически устанавливаются вашим env-файлом из docker compose):

```
export PGUSER="$POSTGRES_USER"
export PGDATABASE="$POSTGRES_DB"
export PGPASSWORD="$POSTGRES_PASSWORD"
pg_dumpall > "/tmp/pg.dump"
```

Выходим из контейнера:

```
exit
```

Скопируйте дамп из контейнера:

```
docker compose cp postgres:/tmp/pg.dump backups/pg.dump
```

Остановите контейнер базы данных и подготовьте данные для нового кластера:

```
docker compose stop postgres
mv ./docker-volume/db ./docker-volume/db.bak
mkdir ./docker-volume/db && chmod 700 ./docker-volume/db
```

Обновите версию контейнера PostgreSQL (например, замените `postgres:13-alpine`к `postgres:17-alpine`):

```
vim docker-compose.yml
```

Извлеките новый образ PostgreSQL Docker:

```
docker compose pull
```

Перезапустите контейнер только PostgreSQL и подождите, пока не увидите: `database system is ready to accept connections`

```
docker compose up -d postgres
docker compose logs -f postgres
```

Скопируйте дамп базы данных внутрь контейнера:

```
docker compose cp "backups/pg.dump" postgres:/tmp/pg.dump
```

Зайти внутрь контейнера

```
docker compose exec -it postgres /bin/bash
```

Проверьте версию PostgreSQL и повторно импортируйте данные базы данных. Затем сбросьте пароль пользователя базы данных PeerTube, чтобы устранить потенциальную проблему аутентификации, если старый алгоритм хэширования пароля устарел.

```
export PGUSER="$POSTGRES_USER"
export PGDATABASE="$POSTGRES_DB"
export PGPASSWORD="$POSTGRES_PASSWORD"
psql -U "$POSTGRES_USER" -c "SELECT version();"
psql -U "$POSTGRES_USER" -f /tmp/pg.dump
psql -U "$POSTGRES_USER" -c "ALTER USER $POSTGRES_USER WITH PASSWORD '$POSTGRES_PASSWORD'"
```

Выходим из контейнера:

```
exit
```

Перезапустите другие службы и проверьте, все ли в порядке:

```
docker compose up -d peertube webserver certbot
docker compose logs -f peertube
```

Если вас устраивают результаты, вы можете удалить каталог резервных копий и старые каталоги данных:

```
rm -rf ./docker-volume/db.bak backups
```

## Строить

### Производство

```
git clone https://github.com/chocobozzz/PeerTube /tmp/peertube
cd /tmp/peertube
docker build . -f ./support/docker/production/Dockerfile
```

### Разработка

У нас нет образа Docker для разработки. Видеть [руководство по содействию](https://docs.joinpeertube.org/contribute/getting-started#develop) для получения дополнительной информации о том, как взломать PeerTube!


# С офф сайта, Установка на  Linux
Стыбрено [тут](https://docs.joinpeertube.org/install/any-os)

## Руководство по производству

- [Установка](https://docs.joinpeertube.org/install/#installation)
- [Обновление](https://docs.joinpeertube.org/install/#upgrade)

## Установка

Пожалуйста, не устанавливайте PeerTube для производства на устройствах с низкой пропускной способностью (пример: ваш канал ADSL). Если вам нужна информация о соответствующем оборудовании для запуска PeerTube, см. [Часто задаваемые вопросы](https://joinpeertube.org/en_US/faq#should-i-have-a-big-server-to-run-peertube) .

### 🔨 Зависимости

Следуйте инструкциям [руководство по зависимостям](https://docs.joinpeertube.org/support/doc/dependencies) .

### 👷 Пользователь PeerTube

Создайте `peertube` пользователь с `/var/www/peertube` дом:

Установите его пароль:

Убедитесь, что корневой каталог Peertube доступен для nginx:

**Во FreeBSD**

или используйте `adduser` чтобы создать его в интерактивном режиме.

### 🗃️База данных

Создайте рабочую базу данных и пользователя Peertube внутри PostgreSQL:

Здесь следует ввести пароль для PostgreSQL `peertube` пользователя, который следует скопировать в `production.yaml` файл. Не нажимайте Enter, иначе оно будет пустым.

Затем включите расширения, необходимые PeerTube:

### 📄 Подготовьте каталог PeerTube.

Загрузите последнюю версию Peertube с тегами:

Откройте каталог peertube, создайте несколько необходимых каталогов:

Загрузите последнюю версию клиента Peertube, разархивируйте и удалите zip:

Установите Пертуб:

### 🔧 Конфигурация PeerTube

Скопируйте файл конфигурации по умолчанию, содержащий конфигурацию по умолчанию, предоставленную PeerTube. Ты **не должен** обновите этот файл.

Теперь скопируйте конфигурацию производственного примера:

Затем отредактируйте `config/production.yaml` файл в соответствии с конфигурацией вашего веб-сервера и базы данных. В частности:

- `webserver`: общедоступная информация об обратном прокси
- `secrets`: секретные строки, которые необходимо создать вручную (версия PeerTube >= 5.0).
- `database`: Настройки PostgreSQL
- `redis`: настройки Redis
- `smtp`: Если вы хотите использовать электронную почту
- `admin.email`: Чтобы правильно заполнить `root` адрес электронной почты пользователя

Ключи, определенные в `config/production.yaml` переопределит ключи, определенные в `config/default.yaml`.

**PeerTube не поддерживает смену хоста веб-сервера** . Несмотря на то [PeerTube CLI может помочь вам сменить имя хоста](https://docs.joinpeertube.org/maintain/tools#update-host-js) Официальной поддержки для этого нет, поскольку это рискованная операция, которая может привести к непредвиденным ошибкам.

### 🚚 Веб-сервер

Мы предоставляем только официальные файлы конфигурации для Nginx.

Скопируйте шаблон конфигурации nginx:

Установите домен для файла конфигурации веб-сервера, заменив `[peertube-domain]` с доменом для сервера Peertube:

Затем измените файл конфигурации веб-сервера. Пожалуйста, обратите внимание на:

- тот `alias`, `root` и `rewrite` Пути директив, пути должны соответствовать местоположению вашей файловой системы PeerTube
- тот `proxy_limit_rate` и `limit_rate` директивы, если вы планируете транслировать видео с высоким битрейтом (например, 4K со скоростью 60 кадров в секунду)

Активируйте файл конфигурации:

Чтобы сгенерировать сертификат для вашего домена, необходимый для работы https, вы можете использовать [Давайте зашифруем](https://letsencrypt.org/) :

Certbot должен был установить cron для автоматического продления вашего сертификата. Поскольку наш шаблон nginx поддерживает обновление веб-корня, мы предлагаем вам обновить файл конфигурации обновления, чтобы использовать `webroot` аутентификатор:

Если вы планируете иметь много одновременных зрителей на своем экземпляре PeerTube, рассмотрите возможность увеличения `worker_connections` ценить: [https://nginx.org/en/docs/ngx\_core\_module.html#worker\_connections](https://nginx.org/en/docs/ngx_core_module.html#worker_connections) .

**Если вы используете FreeBSD**

Во FreeBSD вы можете использовать [Обезвоженный](https://dehydrated.io/) `security/dehydrated` для [Давайте зашифруем](https://letsencrypt.org/)

### ⚗️ Настройка TCP/IP для Linux

Ваш дистрибутив может включать это по умолчанию, но, по крайней мере, Debian 9 этого не делает, а планировщик FIFO по умолчанию весьма склонен к «раздуванию буфера» и экстремальным задержкам при работе с более медленными клиентскими соединениями, с которыми мы часто сталкиваемся на видеосервере.

### 🧱система

Если ваша ОС использует systemd, скопируйте шаблон конфигурации:

Проверьте служебный файл (пути PeerTube и директивы безопасности):

Сообщите systemd перезагрузить конфигурацию:

Если вы хотите запустить PeerTube при загрузке:

Бегать:

**Если вы используете FreeBSD**

В FreeBSD скопируйте сценарий запуска и обновите rc.conf:

Бегать:

**Если вы используете OpenRC**

Если ваша ОС использует OpenRC, скопируйте служебный скрипт:

Если вы хотите запустить PeerTube при загрузке:

Запустите и распечатайте последние журналы:

### 🧑💻Администратор

Имя пользователя администратора `root` и пароль генерируется автоматически. Его можно найти в журналах PeerTube (путь указан в `production.yaml`). Вы также можете установить другой пароль с помощью:

Альтернативно вы можете установить переменную среды `PT_INITIAL_ROOT_PASSWORD`, к вашему собственному паролю администратора, хотя он должен состоять из 6 или более символов.

### 🎉Что теперь?

Теперь ваш экземпляр запущен, и вы можете:

- Добавьте свой экземпляр в общедоступный индекс экземпляров PeerTube, если вы хотите: [https://instances.joinpeertube.org/](https://instances.joinpeertube.org/)
- Проверять [доступные инструменты CLI](https://docs.joinpeertube.org/support/doc/tools)

## Обновление

### Экземпляр PeerTube

**Проверьте журнал изменений (в частности, *ВАЖНЫЕ ПРИМЕЧАНИЯ* раздел):** [https://github.com/Chocobozzz/PeerTube/blob/develop/CHANGELOG.md](https://github.com/Chocobozzz/PeerTube/blob/develop/CHANGELOG.md)

Запустите сценарий обновления (пароль, который он запрашивает, — это пароль пользователя базы данных PeerTube):

Возможно, вы захотите бежать `sudo -u peertube pnpm store prune` после нескольких обновлений, чтобы освободить место на диске.

**Предпочитаете обновление вручную?**

Сделать резервную копию SQL

Загрузите последнюю версию Peertube с тегами:

Загрузите новую версию и разархивируйте ее:

Установите зависимости узла:

Скопируйте новые значения конфигурации по умолчанию и обновите файл конфигурации:

Измените ссылку, чтобы она указывала на последнюю версию:

### Обновить конфигурацию PeerTube

Если в вашей системе есть `git` установлен, сценарий автоматического обновления должен был создать `config/production.yaml.new` файл, который объединяет ваш текущий файл конфигурации с новыми ключами конфигурации, представленными в новой версии PeerTube.

Просмотрите файл, проверьте и исправьте возможные конфликты:

Затем замените текущий файл конфигурации новым:

### Обновить конфигурацию nginx

Проверьте изменения в конфигурации nginx:

### Обновить службу systemd

Проверьте изменения в конфигурации systemd:

**Если вы используете OpenRC**

### Перезапустите PeerTube

Если вы изменили конфигурацию nginx:

Если вы изменили конфигурацию systemd:

Перезапустите PeerTube и проверьте логи:

### Что-то пошло не так?

Изменять `peertube-latest` пункт назначения к предыдущей версии и восстановите резервную копию SQL: