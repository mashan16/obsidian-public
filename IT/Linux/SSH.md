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
В `/etc/ssh/sshd_config` параметр `PermitRootLogin` ставим `without-password`

Открываем `/root/.ssh/authorized_keys`
```
nano /root/.ssh/authorized_keys
```
И вставляем в таком формате
```
#mashanov.yas
ssh-rsa AAAAB3NzaC1yc2EAAAABJQ и.д ключ
```
После этого можно уже будет войти по SSH по ключу
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
