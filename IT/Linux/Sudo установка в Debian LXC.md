Установим `sudo` , создадим пользователя `user` , добавил его в  `sudoers`

Устанавливаем
```
apt install -y sudo
```
Создаём пользователя `user` и вводить его пароль
```
adduser user
```
Добавляем его в группу `sudo` он же `sudoers`
```
usermod -aG sudo user
```
