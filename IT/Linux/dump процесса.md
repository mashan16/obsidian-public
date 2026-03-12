1. # Установите gdb если не установлен
    
    ```
    sudo yum install gdb
    ```
    
    # Найти PID процесса
    
    ```
    ps aux | grep <имя_процесса>
    ```
    
    # Снять дамп
    
    ```
    sudo gcore -o /path/to/dumpfile <PID>
    ```
    
    # Пример:
    
    ```
    sudo gcore -o /tmp/core.dump 1234
    ```