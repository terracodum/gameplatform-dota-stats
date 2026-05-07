# dota-stats

Микросервис статистики игроков Dota 2. Часть платформы [GamePlatform](https://github.com/terracodum/gameplatform).

**Основная тема:** внешние API, Redis кэширование.

---

## Что делает

- Принимает Steam ID игрока
- Тянет статистику через STRATZ GraphQL API
- Кэширует ответы в Redis
- Отдаёт данные фронту
- После просмотра статистики отправляет событие в core

---

## Стек

| Слой | Технология |
|---|---|
| Backend | Go, chi |
| Frontend | React, MUI |
| БД | PostgreSQL |
| Кэш | Redis |
| Миграции | goose |
| API | [STRATZ GraphQL](https://stratz.com/api) |

---

## Архитектура

```
React фронт
     │
     │ HTTP/JSON
     ▼
  Go backend
     │
     ├── Redis (кэш, TTL 1 час)
     │
     ├── PostgreSQL (история запросов)
     │
     └── STRATZ API (GraphQL)
              │
         лимит: 10,000 запросов/день
```

---

## Структура проекта

```
dota-stats/
├── cmd/
│   └── main.go
├── internal/
│   ├── domain/        # сущности
│   ├── repository/    # работа с БД
│   ├── service/       # бизнес-логика
│   └── handler/       # HTTP handlers
├── migrations/        # goose миграции
├── frontend/          # React приложение
├── Dockerfile
├── docker-compose.yml
└── .env.example
```

---

## Переменные окружения

```env
STRATZ_API_KEY=your_token_here
REDIS_URL=redis://localhost:6379
DATABASE_URL=postgres://user:password@localhost:5432/dota_stats
CORE_URL=http://localhost:8080
PORT=8081
```

---

## Локальный запуск

```bash
cp .env.example .env
# заполни .env своими значениями

docker compose up --build
```

Сервис доступен на **http://localhost:8081**

---

## API

| Метод | Путь | Описание |
|---|---|---|
| GET | /api/v1/player/:steamId | Статистика игрока |
| GET | /api/v1/player/:steamId/matches | История матчей |
| GET | /api/v1/player/:steamId/heroes | Статистика по героям |
| GET | /health | Health check |

---

## Связь с платформой

После загрузки статистики сервис отправляет событие в core:

```
POST {CORE_URL}/api/v1/stats
{
  "service": "dota-stats",
  "userId": "...",
  "payload": { ... }
}
```