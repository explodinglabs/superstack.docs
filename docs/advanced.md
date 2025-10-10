# ⚙️ Advanced Deployments

By default, SuperStack runs as a **single project** that's upgraded in place.

While this is simple, it has some trade-offs:

- Some downtime during upgrades
- No way to test a new version while another is live (blue/green)
- No quick rollback once upgraded

When your app is ready for production, you can enable a **traffic-router** in
to eliminate downtime, enable blue/green testing and easy rollbacks.

## 🧭 How It Works

The traffic router is a lightweight proxy project (already included with
SuperStack) that sits in front of your app. Its responsibilities:

- Route traffic to the active app stack
- Simplify switching between versions
- Handle TLS termination

```mermaid
flowchart TD
    Proxy["Traffic Router"]
    Proxy --> LiveApp["Live App"]
    NextApp["Next App"]
```

In standard mode, the app exposes ports directly. In advanced mode, the **proxy
owns the ports**, and apps connect to it internally over Docker networks.

## 🔄 Deployment Flow

1. Stop exposing ports in the app project — only the proxy will listen on `:80`
   and `:443`.
1. Enable the proxy project (included in the repository).
1. For each deployment, bring up a new app stack (e.g. `app/<commit>`),
   connected to the proxy’s network.
1. Test the new app while the current one remains live.
1. Flip traffic in the proxy to point to the new app.
1. Tear down the old one when ready.

## 🧱 1. Start the Proxy

A `proxy` project already exists in your SuperStack project.

> For consistent environments, use the proxy in all environments including
> development.

Start it:

```sh
docker compose up -d
```

## ⚙️ 2. Adjust the Application

Remove the app's exposed ports and connect it to the proxy's network:

```yaml title="app/compose.yaml" hl_lines="5-11 13-15"
services:
  caddy:
    build:
      context: ./caddy
    environment:
      CADDY_SITE_ADDRESS: ":80"
    networks:
      default:
      proxy_default:
        aliases:
          - ${COMPOSE_PROJECT_NAME}_caddy

networks:
  proxy_default:
    external: true
```

What's Changed?

1. Exposed ports were removed.
1. `CADDY_SITE_ADDRESS` now listens internally on port `:80`.
1. The app joins the proxy's network so traffic can be routed to it.
1. A container alias (`_caddy`) lets the proxy target this service reliably.

You can also remove the `CADDY_SITE_ADDRESS` override in
`compose.override.yaml`.

Recreate the app's Caddy container:

```sh
docker compose up -d --force-recreate caddy
```

Commit these changes – your app is now "proxy-ready".

## 🚀 3. Deploying

The proxy is deployed once (usually manually), and each app is deployed
separately into its own directory.

```
proxy/
  compose.yaml
app/
  a/
    compose.yaml
    .env
  b/
    compose.yaml
    .env
```

Before deploying, build and push your own proxy image by adding an image name
to the Compose file:

```yaml title="proxy/compose.yaml" hl_lines="5"
services:
  caddy:
    build:
      context: ./caddy
    image: ghcr.io/youruser/yourapp-proxy:0.1.0
```

Build and push it:

```sh
docker compose build
docker compose push
```

Create a proxy directory on the server:

```sh
mkdir proxy
```

Copy the proxy's Compose file:

```sh
scp proxy/compose.yaml app-backend:proxy/
```

Start the proxy:

docker compose up -d

## 🆕 4. Deploy the New App Stack

Deploy your app into a new directory (e.g. `b/`):

```sh
scp compose.yaml yourserver:app/b/
```

Start it on the server:

```sh
cd app/b
docker compose up -d
```

Optionally, verify it's healthy before switching traffic:

```sh
docker compose exec -T caddy curl -fsS http://caddy:80/healthz
```

## 🔁 5. Flip Traffic

```sh
cd proxy
docker compose exec caddy curl -X PATCH -d '"newapp_caddy:80"' \
  http://localhost:2019/config/apps/http/servers/srv0/routes/0/handle/0/upstreams/0/dial
```

Traffic now points to the new stack.

## ⚡ GitHub Actions Example

<details>
<summary>Show full workflow</summary>

```yaml title=".github/workflows/ci.yaml"
name: Deploy

on:
  push:
    branches:
      - prod

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Copy compose.yaml from repository to deployment dir
        uses: appleboy/scp-action@master
        with:
          host: ${{ secrets.VPS_HOST }}
          username: ${{ secrets.VPS_USER }}
          key: ${{ secrets.VPS_SSH_KEY }}
          source: "app/compose.yaml"
          target: "app/${{ github.sha }}/"
          strip_components: 1

      - name: Deploy with Docker Compose
        uses: appleboy/ssh-action@v1.0.3
        env:
          GHCR_PAT: ${{ secrets.GHCR_PAT }}
        with:
          host: ${{ secrets.VPS_HOST }}
          username: ${{ secrets.VPS_USER }}
          key: ${{ secrets.VPS_SSH_KEY }}
          envs: GHCR_PAT
          script: |
            set -euo pipefail
            cp .env app/${{ github.sha }}/
            cd app/${{ github.sha }}

            # Pull images
            echo "$GHCR_PAT" | docker login ghcr.io --username "${{ github.actor }}" --password-stdin
            DOCKER_CLIENT_TIMEOUT=300 COMPOSE_HTTP_TIMEOUT=300 docker compose pull --quiet

            # Bring up stack and run healthchecks
            trap 'docker compose down' ERR
            docker compose up --detach
            docker compose exec -T caddy curl -fsS http://caddy:80/healthz
            # Add more healthchecks here
            # docker compose exec -T caddy curl -fsS http://api:8080/healthz
            # docker compose exec -T caddy curl -fsS http://postgrest:3000/

      - name: Flip traffic
        uses: appleboy/ssh-action@v1.0.3
        with:
          host: ${{ secrets.VPS_HOST }}
          username: ${{ secrets.VPS_USER }}
          key: ${{ secrets.VPS_SSH_KEY }}
          script: |
            set -euo pipefail
            cd proxy/caddy

            # Grab the formerly-active stack so we can stop the containers later
            OLD_HASH=$(grep '^reverse_proxy' Caddyfile | awk '{print $2}' | cut -d_ -f1)

            # Flip traffic
            sed -i "s|^reverse_proxy .*:80|reverse_proxy ${{ github.sha }}_caddy:80|" Caddyfile
            docker compose exec caddy caddy reload --config /etc/caddy/Caddyfile

            # Stop the old stack
            cd ~/app/$OLD_HASH
            docker compose down

            # Add to deploy.log
            mkdir -p /var/log/sku-generator
            echo "$(date -u +"%Y-%m-%dT%H:%M:%SZ") ${{ github.sha }}" >> /var/log/sku-generator/deploy.log
```

</details>
