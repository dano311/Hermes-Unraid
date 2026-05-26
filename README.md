# Hermes Agent Unraid Template

This repository contains an Unraid Community Applications style Docker template for running [Hermes Agent](https://github.com/NousResearch/hermes-agent) from the official `nousresearch/hermes-agent` image.

Hermes has good Docker documentation, but no Unraid-specific template. This template follows the official Docker deployment pattern:

- one container running `gateway run`
- persistent Hermes data mounted at `/opt/data`
- optional dashboard side-process via `HERMES_DASHBOARD=1`
- OpenAI-compatible API server on container-localhost, with optional LAN publishing on port `8642`
- Playwright-friendly shared memory via `--shm-size=1g`
- Unraid-friendly file ownership using `HERMES_UID=99` and `HERMES_GID=100`
- no `/opt/hermes` source-code volume mount, which avoids the dashboard permission problems seen in some Unraid attempts

## Files

- `templates/hermes-agent.xml` - Unraid Docker template.
- `ca_profile.xml` - Repository metadata for Community Applications.
- `icon.svg` - Simple repository icon placeholder.

## Manual Sideload

Copy `templates/hermes-agent.xml` to your Unraid flash drive:

```bash
cp templates/hermes-agent.xml /boot/config/plugins/dockerMan/templates-user/my-hermes-agent.xml
```

Then go to **Docker -> Add Container -> Template -> User templates -> Hermes-Agent**.

## First Run

Fill in at least one model provider key before starting, for example:

- `OPENROUTER_API_KEY`
- `ANTHROPIC_API_KEY`
- `OPENAI_API_KEY`

The dashboard is enabled by default and is published at:

```text
http://<unraid-ip>:9119/
```

The dashboard can manage API keys and local config, so do not expose it directly to the internet. Put it behind Tailscale, a VPN, a trusted reverse proxy with auth, or keep it LAN-only.

If you prefer the interactive setup wizard, start the container, open the Unraid container console, and run:

```bash
hermes setup
```

## OpenAI-Compatible API

The API server is enabled by default on `127.0.0.1` inside the container so the dashboard and gateway can talk locally. To expose it to your LAN or another container through Unraid's port mapping:

1. Keep `API_SERVER_ENABLED` set to `true`.
2. Set `API_SERVER_HOST` to `0.0.0.0`.
3. Set `API_SERVER_KEY` to a strong random value, minimum 8 characters.
4. Keep the `Gateway API Port` mapping, default `8642`.

Use `http://<unraid-ip>:8642/v1` from Open WebUI or other OpenAI-compatible clients and pass the key as the bearer token.

## Useful Mounts

The template includes a writable workspace mount at `/workspace`. Hermes can work on files there when configured or instructed to use that path.

Advanced optional mounts are included but blank by default:

- Docker socket: mount `/var/run/docker.sock` only if you intentionally want Hermes to control host Docker.
- Host share mount: point it at a specific share such as `/mnt/user/projects`, not the whole array, unless you understand the risk.

The dashboard's embedded chat/TUI tab is disabled by default. Leave `HERMES_DASHBOARD_TUI=0` if you use `HERMES_UID=99`/`HERMES_GID=100`; some Hermes image versions have permission trouble rebuilding the TUI under a remapped UID. Use Telegram/Discord, the API server, or the Unraid container console for chat until your image version is known-good.

## Community Applications Submission Notes

Before submitting to Community Applications:

1. Publish this repository publicly.
2. Replace the `Icon` URL if you want to use your own hosted icon.
3. Run the Community Applications Validate and Scan flow at `https://ca.unraid.net/submit/new`.
