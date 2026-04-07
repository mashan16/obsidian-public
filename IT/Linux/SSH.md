---
tags:
  - public
  - linux
  - ssh
---

### Настройка доступа по SSH на сервер Linux 
1. Открываем конфиг sshd
```
nano /etc/ssh/sshd_config
```
2. Находим параметр `PermitRootLogin` и ставим `yes` *ctrl+w в nano*
3. Рестартуем сервер sshd
```
systemctl restart sshd.service
```
#### PermitRootLogin описание параметров
 • `yes`: Разрешает вход root-пользователя по SSH без ограничений. 
 • `no`: Запрещает вход root-пользователя по SSH. Это наиболее безопасный вариант.
 • `prohibit-password` # Запрещает вход root-пользователя по SSH с использованием пароля, но разрешает вход с использованием ключей SSH.
• `without-password`: # Разрешает вход root-пользователя по SSH только с использованием ключей SSH, без пароля. 

### Авторизация по ключу SSH
После подключения к удаленному серверу генерируем ключ ssh
```sh
ssh-keygen -t rsa -b 4096
```
Копируем публичный ключ в `authorized_keys`
```sh
cat /root/.ssh/id_rsa.pub >> /root/.ssh/authorized_keys
```
Скачиваем `id_rsa` и `id_rsa.pub` себе на комп из `/root/.ssh/`
Отключаем вход по ssh через пароль для этого закомментируем параметр `PermitRootLogin` в `/etc/ssh/sshd_config`
```sh
sudo sed -i 's/^[[:space:]]*PermitRootLogin[[:space:]]\+/# &/' /etc/ssh/sshd_config
```
Проверяем вручную что параметр закомментирован
```sh
nano /etc/ssh/sshd_config
```
Перезагружаем службу ssh
```sh
systemctl restart ssh
```
Проверяем подключение по ключу должно быть ок, по паролю не должно пускать.
Удаляем ключи, на сервере они больше нам не нужны там.
```sh
rm /root/.ssh/id_rsa /root/.ssh/id_rsa.pub
```
### Доступ по SSH между серверами
*На сервере 1*  
1. Генерируем публичный ключ, на все вопросы жмем Enter  
```
ssh-keygen -t rsa -b 4096
```
2. Посмотреть публичный ключ который мы сгенерировали
```
cat /root/.ssh/id_rsa.pub
```
3. Выделить его тем самым копируя его
*На сервере 2*
4. Открываем authorized_keys
```
nano /root/.ssh/authorized_keys
```
6. Копируем туда ключ из пункта **3**

*Должно получится так*
ssh-rsa AAAAB3NzaC и так дее ключ
*Дополнительно можем ограничить подключение по IP*
*И сверху ключа указать кому принадлежит*
># support key yasiya pupkin
>from="172.20.15.0" ssh-rsa AAAAB3NzaC и так дее ключ
