Установим `sudo` , создадим пользователя `user` , добавил его в  `sudoers`

Устанавливаем
Создаём пользователя `user` и вводить его пароль
```
apt install -y sudo
adduser user
```
Добавляем его в группу `sudo` он же `sudoers`
```
usermod -aG sudo user
```
