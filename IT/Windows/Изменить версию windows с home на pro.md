---
tags:
  - pub
---
Статья [winitpro.ru](https://winitpro.ru/index.php/2020/10/12/upgrade-redakcii-windows-10/)

**1**. Откройте командную строку и [проверьте текущую версию и редакцию Windows](https://winitpro.ru/index.php/2023/08/29/uznat-versiyu-bild-windows/):
-  ![[Pasted image 20250610155312.png]]
- *В этом примере редакция Windows 10 **Home**  (в Windows 11 домашняя редакция называется **Core**).*
```
DISM /online /Get-CurrentEdition
```
**2**. Выведите список редакций, до которых можно обновить вашу версию Windows:
```
DISM /online /Get-TargetEditions
````
- ![[Pasted image 20250610155622.png]]
- *В списке есть редакция Professional, до которой мы хотим обновить ОС.*
- *Чтобы выполнить обновление Home редакции до Pro, воспользуйтесь встроенной утилиты **Changepk.exe**. Запустите эту команду и выберите Change product key и укажите приобретённый вами ключ для Windows 10/11 Professional. Подтвердите апгрейд редакции.*
- 
**3** Вы можете очистить предыдущий ключ и задать новый из командной строки:  
```
slui.exe /upk
```
**4** Если у вас пока отсутствует приобретенный ключ для Windows Pro, укажите ключ `VK7JG-NPHTM-C97JM-9MPGT-3V66T`
```
changepk.exe /ProductKey VK7JG-NPHTM-C97JM-9MPGT-3V66T
```
**5** После этого перезагрузите компьютер, чтобы начать обновление редакции.