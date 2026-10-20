# ATEMS

**Automated Tool and Equipment Management System** — inventory, check-in/out, calibration
tracking, and reporting for tool rooms and warehouses. Flask API backed by PostgreSQL.

This tree is a public snapshot; background is in [`docs/PUBLIC_SNAPSHOT.md`](docs/PUBLIC_SNAPSHOT.md).

## Requirements

- [Docker Engine](https://docs.docker.com/engine/install/) with the [Compose plugin](https://docs.docker.com/compose/install/)
- Enough disk for a PostgreSQL 16 data volume and the API image build (see [`Dockerfile`](Dockerfile))

Official references: [Compose file reference](https://docs.docker.com/compose/compose-file/),
[PostgreSQL Docker image](https://hub.docker.com/_/postgres) (`postgres:16.14-alpine` in
[`docker-compose.yml`](docker-compose.yml)).

## Quick start (Docker Compose)

From the repository root:

1. **Create `.env`**

   ```bash
   cp .env.example .env
   ```

   Edit [`.env.example`](.env.example) copy and replace every `change-me` placeholder. These are
   **required** for Compose (no in-repo defaults):

   | Variable | Purpose |
   |----------|---------|
   | `POSTGRES_PASSWORD` | PostgreSQL password for user `atems_user` (Compose fails if unset) |
   | `SECRET_KEY` | Flask session signing key (Compose fails if unset) |
   | `ADMIN_PASSWORD` | Creates the first admin when the database has no users yet |

   Optional: `ADMIN_USERNAME` (default `admin`), `ADMIN_EMAIL`, `USER_USERNAME` / `USER_PASSWORD`
   for an extra env-based login. Do not commit `.env`.

   Inside Compose, the API uses `DATABASE_URL` built from `POSTGRES_PASSWORD`; you do not need
   to change `SQLALCHEMY_DATABASE_URI` in `.env` for the default stack.

2. **Start services**

   ```bash
   docker compose --env-file .env up -d --build
   ```

   Services are defined in [`docker-compose.yml`](docker-compose.yml): `atems-postgres` (health
   check via `pg_isready`) and `atems-api` (starts after Postgres is healthy).

3. **Open the app**

   The API is published on the host at **`http://127.0.0.1:5000`** (`127.0.0.1:5000:5000` in
   Compose). Postgres is on **`127.0.0.1:5436`** if you need host-side tools.

   Log in with `ADMIN_USERNAME` / `ADMIN_PASSWORD` from `.env`. There are no built-in demo
   passwords.

4. **Check health (optional)**

   ```bash
   curl -fsS http://127.0.0.1:5000/api/health
   ```

## Security notes

- Compose refuses empty `POSTGRES_PASSWORD` or `SECRET_KEY` (see
  [`docker-compose.yml`](docker-compose.yml)).
- On first boot against an empty database, set a strong `ADMIN_PASSWORD`; known weak passwords
  are rejected at login.
- Bind mounts use `user: ${HOST_UID:-1000}:${HOST_GID:-1000}` on the API service. On Linux, if
  file ownership matters, export `HOST_UID` / `HOST_GID` or use
  [`scripts/export_compose_uid_gid.sh`](scripts/export_compose_uid_gid.sh) to generate
  `.env.compose-host-user` and pass `--env-file .env.compose-host-user` as well.

## Development (without Docker)

Python 3 with dependencies from [`requirements.txt`](requirements.txt) and a PostgreSQL instance
you configure yourself. Set `ENVIRONMENT=development` and provide `SECRET_KEY` and
`SQLALCHEMY_DATABASE_URI`. See [`atems.py`](atems.py) and [`migrations/`](migrations/) for Flask
and Alembic layout.

## More documentation

| Topic | File |
|-------|------|
| Snapshot scope | [`docs/PUBLIC_SNAPSHOT.md`](docs/PUBLIC_SNAPSHOT.md) |
| SQLite to PostgreSQL | [`POSTGRESQL_MIGRATION.md`](POSTGRESQL_MIGRATION.md) |
| Reverse proxy / systemd | [`NGINX_DEPLOYMENT.md`](NGINX_DEPLOYMENT.md), [`RUN_ON_SERVER.md`](RUN_ON_SERVER.md) |

Issues and contributions: open a GitHub issue on this repository.
