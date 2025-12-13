# Cron-Systemd

# 📋 ОТЧЕТ: Сравнение Cron и Systemd для планирования задач в Linux

## 📌 Информация о выполнении
- **Студент:** Султанаева Рената
- **Группа:** ПИ-430Б
- **Дата выполнения:** 13.12.2025

---

## 🎯 Цель работы
Сравнить два метода планирования периодических задач в Linux: традиционный **Cron** и современный **Systemd Timer**. Реализовать один и тот же скрипт обоими способами.

---

## 📁 Структура проекта
```
/home/david44/
├── cron_scripts/
│   └── send_request.sh      # Bash-скрипт для отправки HTTP-запросов
├── cron_requests.log        # Лог-файл выполнения скрипта через Cron
├── monitoring-stack/        # Дополнительные материалы
└── ansible-managed-host/    # Дополнительные материалы
```

---

## 📝 Часть 1: Реализация через Cron

### 1.1 Созданный bash-скрипт
**Файл:** `~/cron_scripts/send_request.sh`

```bash
#!/bin/bash
TARGET_URL="https://api.example.com/webhook"
curl -s -w "\nHTTP Status: %{http_code}\n" "$TARGET_URL" >> /home/Renata/cron_requests.log 2>&1
echo "[$(date '+%Y-%m-%d %H:%M:%S')] Request sent to $TARGET_URL" >> /home/Renata/cron_requests.log
exit 0

```

*(Скриншот 1: Содержимое скрипта send_request.sh)*
<img width="1186" height="159" alt="image" src="https://github.com/user-attachments/assets/b7404587-c330-4aa1-a9fa-e6ae082015a9" />



---

### 1.2 Настройка расписания в Crontab
**Команда для просмотра:** `crontab -l`

```
*/5 * * * * /home/david44/cron_scripts/send_request.sh
```

**Объяснение формата:**
```
*/5  *    *    *    *    /home/david44/cron_scripts/send_request.sh
 │    │    │    │    │      │
 │    │    │    │    │      └── Команда для выполнения
 │    │    │    │    └───────── День недели (0-7, 0=воскресенье)
 │    │    │    └────────────── Месяц (1-12)
 │    │    └─────────────────── День месяца (1-31)
 │    └──────────────────────── Час (0-23)
 └───────────────────────────── Минута (0-59) → каждые 5 минут
```

*(Скриншот 2: Вывод команды crontab -l)*
<img width="636" height="516" alt="image" src="https://github.com/user-attachments/assets/82da46be-4416-4c5b-a9ba-836e1448e920" />



---

### 1.3 Проверка работы Cron
**Лог-файл выполнения:** `~/cron_requests.log`

```bash
# Команда для просмотра логов
tail -10 ~/cron_requests.log
```

**Пример вывода:**
```
[2025-12-10 12:00:02] Request sent to https://api.example.com/webhook
HTTP Status: 000
[2025-12-10 12:05:01] Request sent to https://api.example.com/webhook
HTTP Status: 000
[2025-12-13 00:05:03] Request sent to https://api.example.com/webhook
HTTP Status: 000
```

*(Скриншот 3: Содержимое лог-файла)*
<img width="640" height="138" alt="image" src="https://github.com/user-attachments/assets/e0e33e2a-724e-4311-9ed5-1e33098786a2" />



---

### 1.4 Статус Cron демона
```bash
sudo systemctl status cron --no-pager | head -10
```

*(Скриншот 4: Статус службы cron)*
<img width="763" height="203" alt="image" src="https://github.com/user-attachments/assets/5ae925d4-c109-43a7-b0ad-d188636417bd" />


---
📊 Сравнительная таблица Cron vs Systemd
Критерий	Cron	Systemd Timer
Синтаксис	*/5 * * * *	OnUnitActiveSec=10min или OnCalendar=*-*-* *:0/5:00
Логирование	Файл логов (~/cron_requests.log)	Через journald (journalctl -u service)
Зависимости	Нет контроля зависимостей	Есть контроль (After, Requires, Wants)
Пропущенные запуски	Пропускаются без компенсации	Persistent=true компенсирует пропуски
Управление	crontab -e, crontab -l	systemctl, journalctl
Интеграция	Отдельный демон	Интегрирован в systemd экосистему
Безопасность	Выполняется от пользователя	Контроль через SecurityContext, Capabilities
Мониторинг	Логи в файле, нужен доступ к файлу	Единый журнал через journald
Гибкость	Только временные интервалы	Зависимости, условия запуска, контроль ресурсов
