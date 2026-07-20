# ZNAC Project Deploy

Infrastructure repository for deploying ZNAC to AWS Lightsail with Docker Compose.

This repository stores:

- `docker-compose.yml`
- `nginx/`
- GitHub Actions deploy workflow

The application repositories are kept separately and are not committed here:

- `znac`
- `znac-api`

## GitHub Secrets

Add these secrets to the `znac-project` GitHub repository:

- `AWS_HOST`: server IP or domain, for example `znac.org`
- `AWS_USER`: SSH user, for example `ubuntu`
- `AWS_SSH_KEY`: private SSH key that can connect to the server

## Server Layout

The deploy workflow expects this structure on the server:

```text
/home/ubuntu/znac-project
├── docker-compose.yml
├── nginx/
├── znac/
└── znac-api/
```

`znac` and `znac-api` must be git repositories on the server.

Backend runtime secrets should stay on the server in:

```text
/home/ubuntu/znac-project/znac-api/.env.docker
```

## Deploy

Deploy runs automatically on push to `main`.

You can also run it manually from GitHub:

```text
Actions -> Deploy -> Run workflow
```

The workflow runs:

```bash
git pull --ff-only origin main
git -C znac pull --ff-only origin main
git -C znac-api pull --ff-only origin main
docker compose up -d --build
docker image prune -f
```
