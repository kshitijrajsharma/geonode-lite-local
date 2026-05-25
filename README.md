# GeoNode Lite (Local)

A single `docker compose up` brings up a full [GeoNode](https://geonode.org/) on your laptop. No `.env` file, no nginx, no certificates. Just clone and run.


## Requirements

- [Docker Desktop](https://www.docker.com/products/docker-desktop/) (Windows / macOS) or Docker Engine + Compose v2 (Linux)
- **4 GB of RAM** allocated to Docker
- **7 GB of free disk** for images and data
- Ports **8000** and **8888** free

## Setup

```bash
git clone https://github.com/kshitijrajsharma/geonode-lite-local.git
cd geonode-lite-local
docker compose up
```

First boot pulls images and runs migrations, fixtures, and admin creation. Give it **5 to 10 minutes**, then open http://localhost:8000.

## Launch

- GeoNode web UI: **http://localhost:8000**
- GeoServer admin: **http://localhost:8888/geoserver**
- Login (both): **`admin`** / **`admin`**


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

## How this differs from the official GeoNode docker-compose

**Services dropped:**

| Service       | Role in the official setup                   | Why dropped here                                         |
|---------------|----------------------------------------------|----------------------------------------------------------|
| `geonode`     | Nginx reverse proxy + static file serving    | uWSGI serves statics directly via `--static-map`; ports exposed straight to the host |
| `letsencrypt` | Auto-issues HTTPS certificates               | Local only, HTTP is enough                               |
| `memcached`   | Optional cache layer                         | GeoNode already runs fine with `MEMCACHED_ENABLED=False` |

**Services kept** (same images, leaner config): `db` (PostGIS), `redis`, `data-dir-conf`, `geoserver`, `django`, `celery`.

**Other simplifications:**

- No `.env` file or `create-envfile.py`. Every variable is inlined in `docker-compose.yml`.
- No `build:` step. Uses the prebuilt `geonode/geonode:5.0.2` image.
- `deploy.resources.limits.memory` set on every container.
- GeoServer JVM heap reduced from `-Xmx4g` to `-Xmx1g`.
- Celery worker concurrency reduced from 4 to 2.
- Admin credentials, DB passwords, OAuth2 client id/secret all hardcoded.

## Troubleshooting

- **Maps are blank.** GeoServer still starting. Wait a minute and refresh.
- **`django` is unhealthy.** First boot runs migrations and admin creation. Check `docker compose logs django`.
- **Out of memory.** Raise Docker's memory allocation to 6 GB, or lower the limits in `docker-compose.yml`.
- **Port already in use.** Free 8000 / 8888, or change the host-side ports in `docker-compose.yml`.

## Caveat

This setup is for **local use only**. Credentials, secrets, and CORS are wide open. Do not expose it to the public internet.
