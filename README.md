# n8n + NocoDB Docker Setup

This repository contains a simple Docker Compose setup for running `n8n`, `Postgres`, `NocoDB`, and `Gotenberg` locally on Windows 11 after installing `WSL 2` and `Docker Desktop`.

## Included Files

- `compose.yaml`: runs `n8n`, `Postgres`, `NocoDB`, and `Gotenberg`
- `.env.example`: optional environment values you can customize
- `local-files/`: host folder mounted into the container at `/files`
- `local-files/reports/`: host folder where generated Markdown and PDF reports are written
- `workflows/`: tracked host folder mounted into the container at `/workflows`
- `schema/job_search.sql`: expected `job_search` table definition for the report workflow
- `n8n-data/`: host folder for n8n state and SQLite data
- `postgres-data/`: host folder for Postgres database files
- `nocodb-data/`: host folder for NocoDB app files

## First Run

1. Start Docker Desktop and wait until it shows as running.
2. Open PowerShell in this project directory.
3. Optionally copy the example environment file:

```powershell
Copy-Item .env.example .env
```

4. Start the stack:

```powershell
docker compose up -d
```

5. Open [http://localhost:5678](http://localhost:5678) for `n8n`.
6. Open [http://localhost:8080](http://localhost:8080) for `NocoDB`.

## Configuration

If you create a `.env` file, you can change:

- `GENERIC_TIMEZONE`: timezone used by scheduling nodes
- `TZ`: container system timezone
- `N8N_PORT`: host port exposed by Docker
- `N8N_RESTRICT_FILE_ACCESS_TO`: semicolon-separated allowlist for n8n file read/write nodes
- `NOCODB_PORT`: host port exposed by Docker for NocoDB
- `POSTGRES_DB`: generic Postgres database name used by NocoDB and reusable from other services
- `POSTGRES_USER`: generic Postgres username used by NocoDB and reusable from other services
- `POSTGRES_PASSWORD`: generic Postgres password used by NocoDB and reusable from other services
- `NC_AUTH_JWT_SECRET`: NocoDB auth and secret-encryption key

The setup stores n8n data in the host directory `n8n-data/`, so workflows, credentials, and the n8n encryption key survive container restarts.

Postgres stores its data in the host directory `postgres-data/`. This is where the NocoDB `NC_DB` database is persisted, including NocoDB metadata and the data for new projects created in local-storage mode.

The Postgres defaults are intentionally generic so you can also connect to the same Postgres instance from `n8n` later without the credentials looking NocoDB-specific.

Postgres is also exposed on the Windows host at `127.0.0.1:5432`, so local tools can connect with the same database name, username, and password.

The NocoDB app directory is stored in the host directory `nocodb-data/` and mounted into the container at `/usr/app/data`.

The local folder `local-files/` is mounted into the container at `/files`. This is useful for nodes that read or write files on disk.

The Compose setup also allowlists `/files` for n8n file nodes via `N8N_RESTRICT_FILE_ACCESS_TO`, otherwise the `Read/Write Files from Disk` node refuses to write there even though Docker has mounted the directory.

The tracked folder `workflows/` is mounted into the container at `/workflows`. Use it for exported workflow JSON files that you want to commit to Git.

The `job-search-nice-1` workflow expects `public.job_search` to use `text` for `company`, `role`, and `link`. A ready-to-run definition and upgrade script is included in `schema/job_search.sql`.

Telemetry diagnostics and version-check notifications are disabled for `n8n`, and anonymous telemetry is disabled for `NocoDB`.

Because `n8n-data/` is a Windows host bind mount, `N8N_ENFORCE_SETTINGS_FILE_PERMISSIONS` is disabled in this setup.

`Gotenberg` runs as an internal PDF conversion service on the Docker network at `http://gotenberg:3000`. The `job-search-nice-1` workflow uses it to turn an HTML report into a PDF without relying on any external web service.

## Common Commands

Start or recreate the stack:

```powershell
docker compose up -d
```

View logs:

```powershell
docker compose logs -f
```

Export all workflows as separate JSON files into the tracked `workflows/` folder:

```powershell
& "$Env:ProgramFiles\Docker\Docker\resources\bin\docker.exe" exec -u node n8n `
  n8n export:workflow --all --separate --pretty --output=/workflows
```

Stop the stack:

```powershell
docker compose stop
```

Start it again:

```powershell
docker compose start
```

Stop and remove the containers without deleting the host data directories:

```powershell
docker compose down
```

All service data remains in `n8n-data/`, `postgres-data/`, and `nocodb-data/` even after `docker compose down`.

Generated job-search reports are written to `local-files/reports/` on the host and appear inside the `n8n` container at `/files/reports/`.

The workflow writes those reports to fixed paths:

- `local-files/reports/job-search-report.md`
- `local-files/reports/job-search-report.pdf`

Update to the newest n8n image:

```powershell
docker compose pull
docker compose up -d
```

## License

This repository is licensed under the GNU Affero General Public License v3.0. See `LICENSE`.
