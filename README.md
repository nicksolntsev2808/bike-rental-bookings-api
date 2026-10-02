# bike-rental-bookings-api

Учёт бронирований велопроката. Приложение на Python + FastAPI, подключение к MongoDB через `motor` (асинхронный драйвер).

### Структура
```
├── app/
│   └── main.py
├── requirements.txt
└── .env.example
```

## API методы

### Проверка состояния
```bash
curl http://127.0.0.1:8030/health
```

### Создать бронирование
```bash
curl -X POST http://127.0.0.1:8030/api/v1/bookings \
  -H 'Content-Type: application/json' \
  -d '{"customer_name":"Олег","bike_type":"горный","returned":false}'
```

### Получить список (с фильтрами)
```bash
curl http://127.0.0.1:8030/api/v1/bookings
curl "http://127.0.0.1:8030/api/v1/bookings?bike_type=горный"
curl "http://127.0.0.1:8030/api/v1/bookings?returned=false"
```

### Получить по ID
```bash
curl http://127.0.0.1:8030/api/v1/bookings/<id>
```

### Обновить (частично)
```bash
curl -X PATCH http://127.0.0.1:8030/api/v1/bookings/<id> \
  -H 'Content-Type: application/json' \
  -d '{"returned":true}'
```

### Удалить
```bash
curl -X DELETE http://127.0.0.1:8030/api/v1/bookings/<id>
```

---

## Задача: запустить через Docker Compose (MongoDB)

Ни `Dockerfile`, ни `docker-compose.yml` в репозитории нет — это и есть задача, написать их самостоятельно. Обрати внимание: приложение асинхронное (FastAPI + `motor`), а не на SQLAlchemy, как в прошлых задачах — но сама механика Docker Compose от этого не меняется.

1. **Dockerfile** для `app/` — базовый образ Python, установка зависимостей из `requirements.txt`, запуск через `uvicorn app.main:app --host 0.0.0.0 --port <порт>`.
2. **docker-compose.yml** с двумя сервисами:
   - `db` — официальный образ `mongo` (определись с тегом — та же история с `latest`, что и раньше);
   - `app` — собирается из твоего `Dockerfile`, пробрасывает порт наружу.
3. Выясни, какие переменные окружения понимает официальный образ `mongo` для создания root-пользователя при первом старте (имена отличаются и от Postgres, и от MySQL).
4. Добавь **volume** для `db` — в какую директорию MongoDB пишет свои данные внутри контейнера?
5. Настрой **`healthcheck`** для `db`. У MongoDB нет отдельной утилиты вроде `pg_isready`/`mysqladmin` — проверка обычно делается через сам клиент `mongosh` (или `mongo` в старых образах) с командой на `ping`. Поищи, как это обычно пишут в `CMD-SHELL`.
6. Настрой **`depends_on`**, чтобы `app` поднимался после `db` только когда тот реально готов.
7. Если включишь root-авторизацию у `mongo` (`MONGO_INITDB_ROOT_USERNAME`/`MONGO_INITDB_ROOT_PASSWORD`), подумай, что это добавляет в строку подключения `MONGO_URI` — простого `mongodb://db:27017` может не хватить. Если хочешь на первом заходе не усложнять — можно поднять `mongo` вообще без авторизации и вернуться к этому моменту отдельно.
8. Не забудь про хост в `MONGO_URI` — внутри docker-сети это не `localhost` и не `127.0.0.1`.
9. Поднять всё одной командой (`docker compose up -d --build`), проверить `docker compose ps`.
10. Проверить все пять эндпоинтов через `curl`.
11. Убедиться, что данные переживают перезапуск: `docker compose down`, затем `docker compose up -d` — бронирования должны остаться (volume сработал).
