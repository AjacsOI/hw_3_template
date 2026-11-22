# Веб-счётчик в Docker Compose (Redis)

## Запуск
```sh
cp .env.example .env
docker compose up -d --build
docker compose ps
```

## Переменные окружения
- `APP_PORT` — внешний порт на хосте (по умолчанию `8080`, в контейнере `8000`)
- `GREETING` — строка приветствия (по умолчанию `Hello, Docker!`)
- `STORAGE` — режим хранения (`redis`)
- `REDIS_URL` — адрес Redis (`redis://redis:6379/0`)

## Проверки

1. Сохранение счётчика после `down` и `up -d` (без `-v`):
```sh
curl -fsS -X POST http://127.0.0.1:8080/increment
curl -fsS -X POST http://127.0.0.1:8080/increment
curl -fsS http://127.0.0.1:8080/
docker compose down
docker compose up -d
curl -fsS http://127.0.0.1:8080/
```
Счётчик доходит до `2` (`storage: "redis"`) и сохраняется после пересоздания стека без `-v`.

2. Остановка `redis`:
```sh
docker compose stop redis
curl -s -o /dev/null -w "GET /healthz -> %{http_code}\n" http://127.0.0.1:8080/healthz
curl -s -o /dev/null -w "GET /readyz -> %{http_code}\n" http://127.0.0.1:8080/readyz
curl -s -o /dev/null -w "POST /increment -> %{http_code}\n" -X POST http://127.0.0.1:8080/increment
```
Коды ответа: `200`, `503`, `503`.

3. Запуск `redis` и восстановление:
```sh
docker compose start redis
sleep 3
curl -fsS http://127.0.0.1:8080/
curl -fsS -X POST http://127.0.0.1:8080/increment
```
`GET /` возвращает `2`, следующий `POST /increment` — `3`.
