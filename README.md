# Paperclip

[Paperclip](https://github.com/paperclipai/paperclip) is open-source orchestration for teams of AI agents.

This is a docker compose setup with Paperclip, PostgreSQL and Redis.

Set `SECRET_KEY`, `PAPERCLIP_BASIC_AUTH` and API keys in `.env`, then:

```sh
# Local (http://paperclip.local.itkdev.dk)
docker compose up -d

# Server (https://${COMPOSE_SERVER_DOMAIN})
docker compose -f docker-compose.server.yml up -d
```
