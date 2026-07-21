# ZNAC Project

[Russian version](README.ru.md)

Infrastructure and deployment repository for ZNAC.

ZNAC is split into three GitHub repositories:

- `znac-project`: Docker Compose, Nginx, and production deployment.
- `znac`: React frontend.
- `znac-api`: Express API.

Live site: https://znac.org

## Related Repositories

- Frontend: https://github.com/AlinaZolotavina/znac
- Backend API: https://github.com/AlinaZolotavina/znac-api

## Responsibilities

- Store production Docker Compose configuration.
- Build and serve the React frontend through Nginx.
- Proxy `/api/*` and `/uploads/*` requests to the Express backend.
- Run production deployment through GitHub Actions and SSH.
- Keep production infrastructure separate from application code.

## Architecture

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

Production runs on AWS Lightsail.

MongoDB is installed on the host machine, not inside Docker. The backend container connects to it through `host.docker.internal`.

## Repository Layout

```text
znac-project/
  .github/workflows/deploy.yml
  docker-compose.yml
  nginx/
    Dockerfile
    nginx.conf
  znac/      # separate git repository, ignored here
  znac-api/  # separate git repository, ignored here
```

## Server Layout

The deploy workflow expects the same layout on the server:

```text
/home/ubuntu/znac-project/
  docker-compose.yml
  nginx/
  znac/
  znac-api/
```

`znac` and `znac-api` must be valid git repositories on the server.

Backend runtime secrets stay on the server in:

```text
/home/ubuntu/znac-project/znac-api/.env.docker
```

## Environment

GitHub repository secrets required by `znac-project`:

- `AWS_HOST`: public IP address or domain of the AWS Lightsail server.
- `AWS_USER`: SSH user, usually `ubuntu`.
- `AWS_SSH_KEY`: private SSH key with access to the server.

GitHub repository secrets required by `znac` and `znac-api`:

- `ZNAC_PROJECT_DEPLOY_TOKEN`: GitHub token allowed to trigger the deploy workflow in `znac-project`.

## Development

This repository is not used for local frontend or backend feature development.

Use the application repositories directly:

```bash
cd znac
npm start
```

```bash
cd znac-api
npm run dev
```

## Docker

Run the complete production-like stack from this repository:

```bash
docker compose up --build
```

The stack contains:

- `backend`: Express API container.
- `nginx`: Nginx container that builds and serves React, terminates HTTPS, and proxies API traffic.

## CI/CD

Deployment can be triggered in three ways:

- Push to `main` in `znac-project`.
- Successful CI on push to `main` in `znac`.
- Successful CI on push to `main` in `znac-api`.

Deployment flow:

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

Manual deployment is also available in GitHub:

```text
Actions -> Deploy -> Run workflow
```

## Useful Commands

Run production stack:

```bash
docker compose up -d --build
```

View logs:

```bash
docker compose logs -f nginx backend
```

Check containers:

```bash
docker compose ps
```

Check backend through Nginx:

```bash
curl -Ik https://znac.org/api/health
```

Check backend from the Nginx container:

```bash
docker compose exec nginx wget -S -O- http://backend:4000/health
```

## Operational Notes

- HTTPS certificates are mounted from `/etc/letsencrypt`.
- ACME challenge files are mounted from `/var/www/certbot`.
- MongoDB must be running on the host before the backend can start successfully.
- Application secrets are not stored in this repository.
