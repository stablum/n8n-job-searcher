# n8n + NocoDB Docker Setup

This repository contains a simple Docker Compose setup for running `n8n` and `NocoDB` locally on Windows 11 after installing `WSL 2` and `Docker Desktop`.

## Included Files

- `compose.yaml`: runs `n8n` and `NocoDB` with persistent storage
- `.env.example`: optional environment values you can customize
- `local-files/`: host folder mounted into the container at `/files`
- `workflows/`: tracked host folder mounted into the container at `/workflows`

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
- `NOCODB_PORT`: host port exposed by Docker for NocoDB
- `POSTGRES_DB`: generic Postgres database name used by NocoDB and reusable from other services
- `POSTGRES_USER`: generic Postgres username used by NocoDB and reusable from other services
- `POSTGRES_PASSWORD`: generic Postgres password used by NocoDB and reusable from other services
- `NC_AUTH_JWT_SECRET`: NocoDB auth and secret-encryption key

The setup stores n8n data in the Docker volume `n8n_data`, so workflows and credentials survive container restarts.

Postgres stores the NocoDB `NC_DB` database in the Docker volume `postgres_data`. This is where NocoDB stores its metadata and the data for new projects created in local-storage mode.

The Postgres defaults are intentionally generic so you can also connect to the same Postgres instance from `n8n` later without the credentials looking NocoDB-specific.

The Docker volume `nocodb_data` is still mounted for NocoDB app files under `/usr/app/data`.

The local folder `local-files/` is mounted into the container at `/files`. This is useful for nodes that read or write files on disk.

The tracked folder `workflows/` is mounted into the container at `/workflows`. Use it for exported workflow JSON files that you want to commit to Git.

Telemetry diagnostics and version-check notifications are disabled for `n8n`, and anonymous telemetry is disabled for `NocoDB`.

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

Stop and remove the container without deleting the Docker volume:

```powershell
docker compose down
```

Update to the newest n8n image:

```powershell
docker compose pull
docker compose up -d
```

## License

This repository is licensed under the GNU Affero General Public License v3.0. See `LICENSE`.
