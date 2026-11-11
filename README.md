# Workspace database

PostgreSQL 17 Alpine в Docker Compose. Автоматических бэкапов пока нет.

## Запуск

Нужны работающий Docker Engine / Docker Desktop и Docker Compose.

```sh
cp .env.example .env
chmod 600 .env
openssl rand -hex 32
# Запиши полученный пароль в POSTGRES_PASSWORD в .env.
docker compose up -d --remove-orphans
docker compose ps
```

`--remove-orphans` удалит контейнер старого сервиса бэкапов, если он был запущен.
Данные PostgreSQL и ранее созданные файлы бэкапов сохраняются.

Настройки можно держать вне репозитория:

```sh
docker compose --env-file ~/.config/workspace/database.env up -d --remove-orphans
```

Используй тот же `--env-file` и для остальных команд Compose.
В git хранится только `.env.example`, без настоящего пароля.
После изменения настроек выполни `docker compose up -d`.

## Подключение

С этой машины: `127.0.0.1:5432`, база `workspace`, пользователь `workspace_admin`,
пароль из `.env`. Порт можно поменять через `POSTGRES_PORT`.
Доступ из внешней сети закрыт. Другие контейнеры нужно подключить к сети этого
Compose (обычно `database_default`) и использовать адрес `postgres:5432`.

`workspace_admin` — администратор для обслуживания.
Для приложений создай отдельные роли с минимальными правами на нужные схемы/таблицы.
Схема данных и миграции пока не заданы.

```sh
docker compose logs -f postgres
docker compose exec postgres sh -c 'psql -U "$POSTGRES_USER" -d "$POSTGRES_DB"'
```

## Хранение данных

Данные остаются в именованном Docker volume `postgres_data`.
`docker compose down` сохраняет их. **`docker compose down -v` удаляет volume с базой.**
Volume не заменяет резервную копию: при потере диска данные будут потеряны.

Изменение `POSTGRES_PASSWORD` в `.env` не меняет пароль уже существующей роли:
сначала смени его в PostgreSQL, затем в настройках.
Не меняй основную версию PostgreSQL без миграции. Переход с Debian-образа на
Alpine для существующей базы делай через выгрузку и восстановление в новый volume.
