# EffectiveMobile_Test

Простое веб-приложение, развернутое в Docker-контейнерах.
Приложение доступно пользователю через Nginx, который работает как reverse proxy и проксирует запросы к backend-сервису внутри Docker-сети.

## Структура проекта

```text
.
├── backend/
│   ├── Dockerfile
│   └── app.py
├── nginx/
│   └── nginx.conf
├── docker-compose.yml
├── .dockerignore
└── README.md
```

## Используемые технологии

* Python 3.12
* Python `http.server`
* Docker
* Docker Compose
* Nginx

## Архитектура

```text
User
 |
 | HTTP request: curl http://localhost
 v
Nginx container :80
 |
 | proxy_pass http://backend:8080
 v
Backend container :8080
```

Nginx принимает HTTP-запросы на порту `80`, опубликованном на хосте.
Backend-сервис слушает порт `8080`, но этот порт не публикуется наружу.
Взаимодействие между Nginx и backend происходит только внутри отдельной Docker-сети.

## Принятые технические решения

### Backend

Для backend используется простой HTTP-сервер на Python через стандартный модуль `http.server`.

Сервер:

* слушает порт `8080`;
* отвечает на путь `/`;
* возвращает текст:

```text
Hello from Effective Mobile!
```

Backend запускается в отдельном контейнере из собственного `Dockerfile`.

В `Dockerfile` используется официальный образ:

```dockerfile
python:3.12-slim
```

Также в Dockerfile:

* задан `WORKDIR`;
* используется `EXPOSE 8080` для документирования порта;
* приложение запускается через `CMD`;
* контейнер запускает приложение не от root-пользователя.

### Nginx

Для Nginx используется официальный образ:

```text
nginx:1.27-alpine
```

Отдельный Dockerfile для Nginx не используется, потому что конфигурация подключается через volume:

```yaml
volumes:
  - ./nginx/nginx.conf:/etc/nginx/conf.d/default.conf:ro
```

Nginx проксирует запросы к backend по имени сервиса Docker Compose:

```nginx
proxy_pass http://backend:8080;
```

Также передаются стандартные proxy-заголовки:

* `Host`
* `X-Real-IP`
* `X-Forwarded-For`

### Docker Compose

Docker Compose поднимает два сервиса:

* `backend`
* `nginx`

Наружу публикуется только порт Nginx:

```yaml
ports:
  - "80:80"
```

Backend использует только внутренний порт:

```yaml
expose:
  - "8080"
```

Для взаимодействия контейнеров используется отдельная Docker-сеть:

```yaml
networks:
  effective-network:
    driver: bridge
```

IP-адреса не хардкодятся. Сервисы обращаются друг к другу по именам внутри Docker-сети.

## Запуск проекта

Из корня проекта выполните:

```bash
docker compose up --build
```

Для запуска в фоновом режиме:

```bash
docker compose up --build -d
```

## Проверка работоспособности

После запуска выполните:

```bash
curl http://localhost
```

Ожидаемый ответ:

```text
Hello from Effective Mobile!
```

## Проверка сетевой изоляции backend

Backend не должен быть доступен напрямую с хоста.

Проверка:

```bash
curl http://localhost:8080
```

Ожидаемо соединение не должно установиться, потому что порт `8080` не опубликован наружу.

Backend доступен только внутри Docker-сети для Nginx.

## Остановка проекта

```bash
docker compose down
```

## Логи

Посмотреть логи всех сервисов:

```bash
docker compose logs
```

Логи backend:

```bash
docker compose logs backend
```

Логи nginx:

```bash
docker compose logs nginx
```
