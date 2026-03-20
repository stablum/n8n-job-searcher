# n8n Docker Setup

These notes assume `WSL 2` and `Docker Desktop` are already installed on Windows 11.

## Start n8n

1. Start Docker Desktop and wait until it shows as running.
2. Open PowerShell.
3. Create a persistent Docker volume:

```powershell
docker volume create n8n_data
```

4. Start n8n:

```powershell
docker run -d `
  --name n8n `
  -p 5678:5678 `
  -e GENERIC_TIMEZONE="Europe/Amsterdam" `
  -e TZ="Europe/Amsterdam" `
  -e N8N_ENFORCE_SETTINGS_FILE_PERMISSIONS=true `
  -e N8N_RUNNERS_ENABLED=true `
  -v n8n_data:/home/node/.n8n `
  docker.n8n.io/n8nio/n8n
```

5. Open [http://localhost:5678](http://localhost:5678).

## Basic Docker Commands

Stop n8n:

```powershell
docker stop n8n
```

Start it again:

```powershell
docker start n8n
```

View logs:

```powershell
docker logs -f n8n
```

Remove the container without deleting workflow data:

```powershell
docker rm -f n8n
```

The `n8n_data` volume keeps your data between container restarts or container re-creation.
