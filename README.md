# GeoNode Lite (Local)

A single `docker compose up` brings up a full [GeoNode](https://geonode.org/) on your laptop. No `.env` file, no nginx, no certificates. Just clone and run.

## Installation

- GeoNode web UI: **http://localhost:8000**
- GeoServer admin: **http://localhost:8888/geoserver**
- Login (both): **`admin`** / **`admin`**

## Requirements

- [Docker Desktop](https://www.docker.com/products/docker-desktop/) (Windows / macOS) or Docker Engine + Compose v2 (Linux)
- **4 GB of RAM** allocated to Docker
- **7 GB of free disk** for images and data
- Ports **8000** and **8888** free

## Start

```bash
git clone <this-repo>
cd geonode-lite-local
docker compose up -d
```

First boot pulls images and runs migrations, fixtures, and admin creation. Give it **5 to 10 minutes**, then open http://localhost:8000.

To watch progress: `docker compose logs -f django`.

## Stop / start / wipe

```bash
docker compose stop      # pause, keep data
docker compose start     # resume
docker compose down      # remove containers, keep data
docker compose down -v   # remove containers AND data
```

## What's inside

| Service        | Image                                   | Memory cap |
|----------------|-----------------------------------------|------------|
| `db`           | `geonode/postgis:15-3.5-latest`         | 512 MB     |
| `redis`        | `redis:7-alpine`                        | 128 MB     |
| `data-dir-conf`| `geonode/geoserver_data:2.28.x-latest`  | 64 MB      |
| `geoserver`    | `geonode/geoserver:2.28.x-latest`       | 1.5 GB     |
| `django`       | `geonode/geonode:5.0.2`                 | 1.5 GB     |
| `celery`       | `geonode/geonode:5.0.2`                 | 1 GB       |

Idle usage is about 2 GB. Ceiling is 4.75 GB.

## Troubleshooting

- **Maps are blank.** GeoServer still starting. Wait a minute and refresh.
- **`django` is unhealthy.** First boot runs migrations and admin creation. Check `docker compose logs django`.
- **Out of memory.** Raise Docker's memory allocation to 6 GB, or lower the limits in `docker-compose.yml`.
- **Port already in use.** Free 8000 / 8888, or change the host-side ports in `docker-compose.yml`.

## Caveat

This setup is for **local use only**. Credentials, secrets, and CORS are wide open. Do not expose it to the public internet.
