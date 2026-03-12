---
tags:
  - public
  - android
  - backup
---
**Backup**
1. Запустите Cygwin Терминал и введите (при необходимости замените mmcblk0):
```
adb forward tcp:5555 tcp:5555
```

```
adb shell
```

```
su
```

```
/system/xbin/busybox nc -l -p 5555 -e /system/xbin/busybox dd if=/dev/block/mmcblk0
```
- Вы увидите мигающий курсор на следующей строке слева. На данный момент аппарат ожидает передачи Блока по сети
2. Откройте другой Cygwin Терминал и введите:
```
adb forward tcp:5555 tcp:5555
```

```
cd /usr/backup_r8pro/
```

```
nc 127.0.0.1 5555 | pv -i 0.5 > mmcblk0.raw
```
---
**Restore**
Для восстановления сохраненных Разделов необходимо перезагрузить аппарат в так называемый «Fastboot mode» (для Teclast x98 он называется Droidboot), делается это следующим образом:

1 вариант — аппарат должен быть полностью выключенным (Power Off), зажимаете одновременно клавиши Volume"+" и Power и ждете загрузки, или

2 вариант — через команду из Терминала
adb reboot bootloader

При этом драйверы и бинарный файл fastboot должен быть установлен в системе (Windows, Linux и т.д.)

  

После загрузки аппарата в «Fastboot mode», в терминале на компьютере последовательно вводите следующее

Затирает boot
```
fastboot erase boot 
```

Затирает recovery
```
fastboot erase recovery 
```
Затирает system
```
fastboot erase system 
```
Затирает data
```
fastboot erase data 
```
Затирает cache
```
fastboot erase cache 
```
Записывает в раздел data сохраненный образ из файла data.img
```
fastboot flash data data.img
``` 
Записывает в раздел cache сохраненный образ из файла cache.img
```
fastboot flash cache cache.img 
```
Записывает в раздел data сохраненный образ из файла data.img
```
fastboot flash boot boot.img 
```
Записывает в раздел recovery сохраненный образ из файла recovery.img
```
fastboot flash recovery recovery.img 
```
Записывает в раздел system сохраненный образ из файла system.img
```
fastboot flash system system.img 
```
Если для аппарата имеется cwm.img, тогда записываем его в раздел recovery тоже
```
fastboot flash recovery cwm.img 
```

```
fastboot reboot
```

Что будете затирать и что будете записывать зависит от Вашего выбора.