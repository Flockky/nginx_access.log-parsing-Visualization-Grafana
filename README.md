# 🧾 Nginx Access Log Parsing & Visualization with Vector + ClickHouse + Grafana

Полный пайплайн для сбора, хранения и визуализации Nginx access-логов.  
Проект основан на **Vector** → **ClickHouse** → **Grafana**, и включает готовый дашборд, конфигурации и инструкции по развёртыванию.

<img width="1688" height="611" alt="fce3b411-a646-460e-a53d-0d498346adc6" src="https://github.com/user-attachments/assets/0bc829da-3f92-4e53-b0c4-b27dd0c7e1d5" />
---



## 📦 Структура проекта

```
.
├── clickhouse/
│   ├── docker-compose.yaml      # Контейнер ClickHouse
│   └── users.xml                # Конфигурация пользователей ClickHouse
├── grafana/
│   ├── NGINX_Global_Dashboard_Grafana.json  # Готовый дашборд
│   └── datasource.yaml (если нужен)
└── vector/
    ├── docker-compose.yml       # Контейнер Vector
    └── vector.yaml              # Основной конфиг Vector
```

---

## 🚀 Возможности

✔ Подача Nginx access логов в ClickHouse через Vector  
✔ Готовая схема таблицы для хранения расширенных логов  
✔ Grafana dashboard для визуализации трафика, статусов, latency, user-agents и т.д.  
✔ Авто-очистка данных через ClickHouse TTL  
✔ Контейнеризация всех компонентов (Vector, ClickHouse, Grafana)

---

# 🛠 Установка и настройка

Ниже приведены шаги для полного развёртывания системы.

---

## 1️⃣ Разворачиваем Vector на сервере с Nginx

Vector должен работать на том сервере, где находится ваш **nginx_access.log**.

1. Перейдите в директорию `vector/`  
2. Укажите путь до access-логов и URL ClickHouse в `vector.yaml`  
3. Запустите контейнер:

```bash
docker compose up -d
```

---

## 2️⃣ Развёртываем ClickHouse

1. Откройте директорию `clickhouse/`
2. В `users.xml` укажите:
   - разрешённые сети/хосты
   - пользователей / пароли
3. Запустите ClickHouse:

```bash
docker compose up -d
```

---

## 3️⃣ Создаём таблицу в ClickHouse

Перейдите в браузере по адресу:

```
http://<Ваш_Сервер_Clickhouse>:8123
```

И выполните SQL:

```sql
CREATE TABLE zabbix_logs.nginx_access_extended (
    timestamp DateTime,
    remote_addr String,
    remote_user String,
    request String,
    status UInt16,
    body_bytes_sent UInt64,
    http_referer String,
    http_user_agent String,
    request_time Float32,
    upstream_response_time String
) ENGINE = MergeTree()
ORDER BY (timestamp)
TTL timestamp + INTERVAL 2 DAY
SETTINGS index_granularity = 8192;
```

📝 **TTL можно изменить**, чтобы указать желаемый срок хранения данных.

---

## 4️⃣ Перезапускаем Vector и проверяем запись логов

```bash
docker restart vector
```

Проверьте есть ли новые записи:

```sql
SELECT * FROM zabbix_logs.nginx_access_extended LIMIT 10;
```

---

## 5️⃣ Подключаем ClickHouse в Grafana

1. Откройте Grafana → *Configuration → Data sources*  
2. Добавьте новый источник **ClickHouse**  
3. Укажите:
   - URL ClickHouse
   - базу `zabbix_logs`
   - пользователя/пароль

---

## 6️⃣ Импортируем Grafana Dashboard 📊

Перейдите в *Dashboards → Import* и загрузите:

```
grafana/NGINX_Global_Dashboard_Grafana.json
```

После импорта выберите созданный ClickHouse datasource.

---

## 7️⃣ Готово 🎉

Вы получите полноценный мониторинг Nginx логов в реальном времени:

- 📈 RPS  
- 🧭 География и IP  
- 📡 User-Agents  
- 🔁 Upstream response time  
- 🚦 HTTP status codes  
- ⏱ Request latency  

