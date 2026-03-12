---
tags:
  - public
  - linux
  - database
---

**Общие команды**
Показать всех пользователей
```
SELECT DISTINCT User, Host FROM mysql.user;
```
Показать все базы
```
SHOW DATABASES;
```
Просмотр прав пользователей для БД например `ITInvent`
```
SHOW GRANTS FOR 'ITInvent'@'%';
```
---
## Debian, MySQL в Docker контейнере
1. Обновляемся
```
apt update && apt upgrade -y
```
1. Устанавливаем докер
```
apt install curl -y && curl -fsSL https://get.docker.com -o get-docker.sh && sh ./get-docker.sh
```
1. Запускаем контейнер с MySql внутри (my_super_password меняем на свой)
```
docker run --name mysql-server \
  -e MYSQL_ROOT_PASSWORD='my_super_password' \
  -p 3306:3306 \
  -d mysql
```
2. Подключаемся к bash контейнера
```
docker exec -it mysql-server bash
```
3. Входим в СУБД MySQL (Вводим пароль из 1 пункта)
```
mysql -u root -p
```
4. Создаём базу данных (ITInvent как в данном случае)
```
CREATE DATABASE ITInvent CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
```
5. *Проверяем что база создалась* `SHOW DATABASES;`
6. Создаём пользователя
```
CREATE USER 'ITInvent'@'%' IDENTIFIED BY 'DB_password';
```
7. Выдаём ему права
```
GRANT ALL PRIVILEGES ON ITinvent.* TO 'ITInvent'@'%';
```
8. Перезагружает таблицы привилегий MySQL.
```
FLUSH PRIVILEGES;
```
- Что-бы сменить пароль пользователя MySQL нужно ввести команду
```
ALTER USER 'ITInvent'@'%' IDENTIFIED BY 'NewStrongPassword123!';
```