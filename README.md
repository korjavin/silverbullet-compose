# SilverBullet with OAuth2 Protection

Docker Compose setup for [SilverBullet](https://silverbullet.md/) — a self-hosted, extensible Markdown note-taking app — with OAuth2 authentication via forward-auth, designed for GitOps deployment with Portainer.

## Features

- **Markdown Notes** — Fast, extensible, self-hosted note-taking with SilverBullet
- **OAuth2 Protection** — Protected by Traefik forward-auth middleware
- **GHCR Vendoring** — Upstream image mirrored weekly to GHCR for version pinning
- **Fully Parameterized** — All configuration via environment variables
- **GitOps Ready** — No secrets in repository
- **Portainer Compatible** — Easy deployment via Portainer stacks

## Prerequisites

- Docker runtime
- Portainer (for GitOps deployment)
- Traefik reverse proxy with external `traefik_default` network
- forward-auth (oauth2-proxy) deployed and providing `forward-auth@docker` middleware
- Domain/subdomain pointing to your server

## Quick Start with Portainer

### 1. Prepare the Space Directory

Create the directory on your server where notes will be stored:

```bash
mkdir -p /path/to/silverbullet/space
```

SilverBullet auto-detects the UID/GID of this directory and runs as that user — no manual `chown` needed.

### 2. Deploy in Portainer

1. Go to **Stacks** → **Add Stack**
2. **Name**: `silverbullet`
3. **Build method**: Select **Repository**
4. **Repository URL**: `https://github.com/korjavin/silverbullet-compose`
5. **Repository reference**: `deploy` ← **IMPORTANT: not master**
6. **Compose path**: `docker-compose.yml`

### 3. Configure Environment Variables

In the **Environment variables** section, set at minimum:

```
SILVERBULLET_HOST=silverbullet.yourdomain.com
SPACE_PATH=/path/to/silverbullet/space
TRAEFIK_CERTRESOLVER=myresolver
```

See the full variable reference below.

### 4. Enable Auto-Deploy Webhook

In Portainer: **Stack** → **Webhooks** → enable and copy the URL.
Add it as a GitHub secret named `PORTAINER_REDEPLOY_HOOK`.

### 5. Trigger First Deploy

Push to `master` or run the **Deploy SilverBullet Stack** workflow manually in GitHub Actions.

## Environment Variables

| Variable | Default | Required | Description |
|---|---|---|---|
| `SILVERBULLET_IMAGE` | `ghcr.io/korjavin/silverbullet-vendor:latest` | No | Docker image to deploy |
| `SILVERBULLET_CONTAINER_NAME` | `silverbullet` | No | Container name |
| `SILVERBULLET_HOST` | — | **Yes** | Hostname for Traefik routing (e.g. `silverbullet.example.com`) |
| `TRAEFIK_NETWORK_NAME` | `traefik_default` | No | External Traefik network name |
| `TRAEFIK_CERTRESOLVER` | `myresolver` | No | Traefik TLS cert resolver name |
| `SPACE_PATH` | `./space` | No | Host path for notes storage (use absolute path in production) |
| `SB_PORT` | `3000` | No | Port SilverBullet listens on |
| `SB_INDEX_PAGE` | `index` | No | Default page to load |
| `SB_USER` | — | No | Built-in auth (`username:password`). Not needed when using forward-auth. |

## GitHub Secrets

| Secret | Description |
|---|---|
| `PORTAINER_REDEPLOY_HOOK` | Portainer stack webhook URL (triggers redeploy on push) |

`GITHUB_TOKEN` is automatic — no setup needed for GHCR vendoring.

## How Updates Work

### Compose changes
```
git push origin master
  → GitHub Actions: create/update deploy branch
  → Portainer: pulls updated docker-compose.yml and redeploys
```

### Image updates (vendor-images.yml)
```
Weekly cron (Monday 04:00 UTC)
  → Pulls ghcr.io/silverbulletmd/silverbullet:latest
  → Pushes to ghcr.io/korjavin/silverbullet-vendor:latest
  → Records digest in vendor-digests.log on deploy branch
  → Triggers Portainer redeploy
```

To pin a specific digest, set `SILVERBULLET_IMAGE` in Portainer to:
```
ghcr.io/korjavin/silverbullet-vendor:latest@sha256:<digest>
```
(Find digests in `vendor-digests.log` on the `deploy` branch.)

## Links

- [SilverBullet documentation](https://silverbullet.md/)
- [SilverBullet Docker install guide](https://silverbullet.md/Install/Docker)
- [Upstream image (ghcr.io/silverbulletmd/silverbullet)](https://github.com/silverbulletmd/silverbullet/pkgs/container/silverbullet)
