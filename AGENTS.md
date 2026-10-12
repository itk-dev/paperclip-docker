# Repository Guidelines

## Project Overview
Docker Compose deployment of [Paperclip](https://github.com/paperclipai/paperclip) (AI agent orchestration) behind Traefik. No application source lives here: the upstream image `ghcr.io/paperclipai/paperclip` is used as-is, with PostgreSQL. Application/provider behaviour (agents, AI connections, LiteLLM gateways) is configured in the Paperclip UI or upstream env vars, not in this repo.

## Architecture & Data Flow
```
Browser → Traefik (network `frontend`, basic auth `paperclip-auth`) → paperclip:3100 → postgres:5432 (network `app`)
```
- `paperclip`: on `frontend` (external, must already exist) and `app`. Waits for `postgres` healthcheck (`depends_on: condition: service_healthy`).
- `postgres`: `postgres:17-alpine`, `app` only, never exposed. DB user and name are hard-coded `paperclip` in `DATABASE_URL`, `POSTGRES_USER`, `POSTGRES_DB`, `pg_isready`.
- State is in bind mounts: `./data/paperclip` → `/paperclip` (owned by `USER_UID`/`USER_GID`), `./data/postgres` → `/var/lib/postgresql/data`.

## Key Directories
- `data/`: runtime state, gitignored except `data/.gitkeep`. `data/redis/` is an unused leftover; there is no Redis service.

## Important Files
- `docker-compose.yml`: local. Plain HTTP, `ports: "3100"`, one router `${COMPOSE_PROJECT_NAME}`.
- `docker-compose.server.yml`: server. `restart: unless-stopped`, `expose: "3100"`, routers `${COMPOSE_PROJECT_NAME}-http` (`web`, redirects to HTTPS) and `-https` (`websecure`), service `${COMPOSE_PROJECT_NAME}-paperclip`.
- `.env.example`: template for every variable the compose files read.
- `.env`: real secrets. **Do not read or edit `.env`.** Change `.env.example` and tell the user what to copy.
- `README.md`: run instructions and LiteLLM gateway setup. It is stale in places: it mentions Redis and `SECRET_KEY`, and neither exists.

## Code Conventions & Common Patterns
- Keep both compose files in sync. They should differ only in restart policy, port publishing, `PAPERCLIP_PUBLIC_URL` (`http://${COMPOSE_DOMAIN}` vs `https://${COMPOSE_SERVER_DOMAIN}`) and Traefik routers.
- Every config value comes from env interpolation. A new variable goes into both compose files and into `.env.example` with a comment.
- Required secrets use `${VAR:?message}` (e.g. `BETTER_AUTH_SECRET`); optional ones use `${VAR:-default}` (e.g. `PAPERCLIP_IMAGE_TAG:-latest`, `USER_UID:-1000`).
- Traefik names are derived from `${COMPOSE_PROJECT_NAME}`. `$` in `PAPERCLIP_BASIC_AUTH` must be doubled (`$$`).
- Before adding env vars, check that upstream Paperclip reads them ([env reference](https://docs.paperclip.ing/reference/deploy/environment-variables/)). Guessed variables do nothing. For example, `OPENAI_BASE_URL` is cleared for managed AI connections; LiteLLM is set up as a connection under **Connectors** instead.

## Runtime/Tooling Preferences
- Docker Compose v2 (`docker compose`), plus a Traefik instance on the external `frontend` network.
- Image tag comes from `PAPERCLIP_IMAGE_TAG`: `latest` (stable release), or `nightly`/`canary`. Custom gateways need an image newer than `v2026.1005.0`. To check which image a tag holds, read the `org.opencontainers.image.revision` label.

## Development Commands
```sh
cp .env.example .env                                  # user does this; fill PAPERCLIP_BASIC_AUTH, secrets
docker compose up -d                                   # local: http://${COMPOSE_DOMAIN}
docker compose -f docker-compose.server.yml up -d      # server: https://${COMPOSE_SERVER_DOMAIN}
docker compose logs -f paperclip
docker compose pull                                    # after changing PAPERCLIP_IMAGE_TAG
```

## Testing & QA
There are no tests, CI or linters. Check changes with:
```sh
docker compose config --quiet
docker compose -f docker-compose.server.yml config --quiet
docker compose config --images                         # resolved image tag
```
Without `--quiet`, `config` prints the interpolated secrets from `.env`, so avoid it in shared output. The runtime smoke test is `up -d`, then `docker compose ps`, then the logs.
