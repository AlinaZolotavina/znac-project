# ZNAC Project

[English version](README.md)

Репозиторий инфраструктуры и деплоя ZNAC.

ZNAC разделен на три GitHub-репозитория:

- `znac-project`: Docker Compose, Nginx и production-деплой.
- `znac`: React frontend.
- `znac-api`: Express API.

Сайт: https://znac.org

## Связанные Репозитории

- Frontend: https://github.com/AlinaZolotavina/znac
- Backend API: https://github.com/AlinaZolotavina/znac-api

## Ответственность

- Хранить production-конфигурацию Docker Compose.
- Собирать и раздавать React frontend через Nginx.
- Проксировать запросы `/api/*` и `/uploads/*` в Express backend.
- Запускать production-деплой через GitHub Actions и SSH.
- Держать production-инфраструктуру отдельно от кода приложений.

## Архитектура

```text
Browser
  |
  v
Nginx container
  |-- serves React build
  |-- /api/*     -> Express backend container
  |-- /uploads/* -> Express backend container
  |
  v
MongoDB on AWS host
```

Production работает на AWS Lightsail.

MongoDB установлена на хост-машине, а не внутри Docker. Backend container подключается к ней через `host.docker.internal`.

## Структура Репозитория

```text
znac-project/
  .github/workflows/deploy.yml
  docker-compose.yml
  nginx/
    Dockerfile
    nginx.conf
  znac/      # отдельный git-репозиторий, игнорируется здесь
  znac-api/  # отдельный git-репозиторий, игнорируется здесь
```

## Структура На Сервере

Deploy workflow ожидает такую же структуру на сервере:

```text
/home/ubuntu/znac-project/
  docker-compose.yml
  nginx/
  znac/
  znac-api/
```

`znac` и `znac-api` должны быть валидными git-репозиториями на сервере.

Production-секреты backend остаются на сервере в файле:

```text
/home/ubuntu/znac-project/znac-api/.env.docker
```

## Окружение

GitHub repository secrets, необходимые для `znac-project`:

- `AWS_HOST`: публичный IP-адрес или домен AWS Lightsail сервера.
- `AWS_USER`: SSH-пользователь, обычно `ubuntu`.
- `AWS_SSH_KEY`: приватный SSH-ключ с доступом к серверу.

GitHub repository secrets, необходимые для `znac` и `znac-api`:

- `ZNAC_PROJECT_DEPLOY_TOKEN`: GitHub token, которому разрешено запускать deploy workflow в `znac-project`.

## Разработка

Этот репозиторий не используется для локальной разработки frontend или backend фич.

Используй репозитории приложений напрямую:

```bash
cd znac
npm start
```

```bash
cd znac-api
npm run dev
```

## Docker

Запустить полный production-like stack из этого репозитория:

```bash
docker compose up --build
```

Stack содержит:

- `backend`: Express API container.
- `nginx`: Nginx container, который собирает и раздает React, завершает HTTPS и проксирует API-трафик.

## CI/CD

Деплой может быть запущен тремя способами:

- Push в `main` в `znac-project`.
- Успешный CI после push в `main` в `znac`.
- Успешный CI после push в `main` в `znac-api`.

Deploy flow:

```text
Push to main
  |
  v
GitHub Actions
  |
  v
Deploy workflow in znac-project
  |
  v
SSH to AWS Lightsail
  |
  v
git pull znac-project, znac, znac-api
  |
  v
docker compose up -d --build
```

Деплой также можно запустить вручную в GitHub:

```text
Actions -> Deploy -> Run workflow
```

## Полезные Команды

Запустить production stack:

```bash
docker compose up -d --build
```

Посмотреть логи:

```bash
docker compose logs -f nginx backend
```

Проверить контейнеры:

```bash
docker compose ps
```

Проверить backend через Nginx:

```bash
curl -Ik https://znac.org/api/health
```

Проверить backend из Nginx container:

```bash
docker compose exec nginx wget -S -O- http://backend:4000/health
```

## Операционные Заметки

- HTTPS-сертификаты монтируются из `/etc/letsencrypt`.
- ACME challenge files монтируются из `/var/www/certbot`.
- MongoDB должна быть запущена на host до успешного старта backend.
- Секреты приложений не хранятся в этом репозитории.
