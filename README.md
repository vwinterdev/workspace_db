# Workspace database

PostgreSQL 17 Alpine в Docker Compose. Нужен запущенный Docker.

## Запуск

```sh
cp .env.example .env
chmod 600 .env
# Укажи POSTGRES_PASSWORD в .env.
docker compose up -d
```