# Effective Mobile — Backend Service

Простой HTTP-сервер на Python за реверс-прокси Nginx,
упакованный в Docker Compose.

---

## Технологии

| Слой | Технология |
|------|-----------|
| Язык | Python 3.12 |
| HTTP-сервер | `http.server` (stdlib) |
| Реверс-прокси | Nginx 1.27 (Alpine) |
| Контейнеризация | Docker + Docker Compose |

---

## Структура проекта

```
.
├── .env                  # переменные окружения (не коммитить секреты)
├── .gitignore
├── docker-compose.yml
├── README.md
├── backend/
│   ├── Dockerfile        # multistage-сборка
│   └── server.py         # HTTP-сервер
└── nginx/
    ├── Dockerfile
    └── default.conf      # конфигурация реверс-прокси
```

---

## Запуск

### Требования

- Docker >= 24
- Docker Compose >= 2.20

### Шаги

```bash
# 1. Клонировать репозиторий
git clone <repo-url>
cd <repo-dir>

# 2. (Опционально) Изменить порты или имена в .env
#    По умолчанию nginx слушает порт 80 на хосте

# 3. Собрать образы и запустить
docker compose up --build -d

# 4. Убедиться, что оба контейнера здоровы
docker compose ps
```

### Остановка

```bash
docker compose down
```

---

## Проверка работоспособности

```bash
# Через curl
curl http://localhost/

# Ожидаемый ответ:
# Hello from Effective Mobile!

# Проверить статус healthcheck-ов
docker inspect --format='{{.Name}} → {{.State.Health.Status}}' \
    effective-mobile-backend effective-mobile-nginx
```

---

## Архитектура

```
                        Docker-сеть: effective-mobile-net
                       ┌─────────────────────────────────┐
                       │                                 │
  Клиент               │  ┌──────────┐    ┌───────────┐  │
    │                  │  │          │    │           │  │
    │  HTTP :80        │  │  nginx   │    │  backend  │  │
    └──────────────────┼──►  :80     ├────►  :8080    │  │
                       │  │          │    │           │  │
                       │  └──────────┘    └───────────┘  │
                       │                                 │
                       └─────────────────────────────────┘

Хост видит только порт 80 (nginx).
Backend не имеет проброшенных портов наружу.
```

**Поток запроса:**

1. Клиент отправляет `GET http://localhost/`
2. **Nginx** принимает запрос на порту `80`
3. Nginx проксирует запрос на `http://backend:8080/` (по имени сервиса внутри docker-сети)
4. **Backend** обрабатывает запрос и возвращает `Hello from Effective Mobile!`
5. Nginx передаёт ответ клиенту

---

## Безопасность

- **Не root** — backend запускается от пользователя `appuser`
- **Минимальные образы** — `python:3.12-slim` и `nginx:1.27-alpine` уменьшают поверхность атаки
- **Порты** — наружу пробрасывается только порт `80` (nginx); backend недоступен с хоста напрямую
- **Секреты** — `.env` добавлен в `.gitignore`; в репозитории хранится только `.env.example`
- **Фиксированные версии** — нет тегов `latest`, образы воспроизводимы
- **Healthcheck** — оба сервиса проверяют собственную доступность; nginx стартует только после healthy backend

---

## Переменные окружения

Все настройки вынесены в `.env`. Для начала работы скопируйте пример:

```bash
cp .env.example .env
```

| Переменная | Значение по умолчанию | Описание |
|---|---|---|
| `HOST_PORT` | `80` | Порт на хосте для nginx |
| `BACKEND_PORT` | `8080` | Внутренний порт backend |
| `BACKEND_CONTAINER` | `effective-mobile-backend` | Имя контейнера |
| `NGINX_CONTAINER` | `effective-mobile-nginx` | Имя контейнера |
| `NETWORK_NAME` | `effective-mobile-net` | Имя docker-сети |
