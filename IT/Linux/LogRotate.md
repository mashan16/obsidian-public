---
tags:
  - public
  - linux
---

### 🔹 3. Настроить ротацию логов только для этого файла

Если вам нужны логи, но не хочется, чтобы один файл разрастался, можно сделать **отдельное правило logrotate**.

Создайте файл `name_logrotate_1` (Поменять на своё название)
```bash
nano /etc/logrotate.d/name_logrotate_1
```
Содержимое файла:
```bash
/npm/data/logs/name_logrotate_1.log {
    daily
    rotate 7
    compress
    delaycompress
    missingok
    notifempty
    copytruncate
}
```

- `daily` — ротация каждый день
- `rotate 7` — хранить 7 архивов
- `compress` — сжимать старые
- `copytruncate` — обрезать файл без перезапуска NPM

### 🔹 4. Ограничить размер файла
Если хотите именно лимит размера, а не по времени, замените `daily` на:
`size 100M`
Тогда лог будет резаться при достижении ~100 МБ.

**Чтобы прямо сейчас проверить, что работает, запустите:**
```bash
logrotate -f /etc/logrotate.d/name_logrotate_1
```