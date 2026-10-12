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

## Own LiteLLM / OpenAI-compatible gateway

Needs an image newer than `v2026.1005.0` (`PAPERCLIP_IMAGE_TAG=nightly`). In the UI:
**Connectors** → *Responses API* (Codex), *Chat Completions API* (OpenCode/Hermes) or
*Messages API* (Claude) → base URL (e.g. `https://litellm.example.com/v1`) + LiteLLM key.
Onboarding has no gateway option: finish it with a subscription/API key (or press Esc), then add the
connector and use the **Custom Gateway** tile under **Agents → New agent**. Remote URLs must be HTTPS.
Type the LiteLLM model name manually; then **Run test** on the agent.
