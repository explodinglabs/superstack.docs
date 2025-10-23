# ☁️ Deploying to Remote Environments

SuperStack is **Docker-native**, so deployments are simple, consistent, and
portable.

The goal is that only `compose.yaml` and secrets need to exist on the remote
server.

## 🧱 1. Build Your Images

If a service has a `build:` section, add your own `image:` name and version tag:

```yaml title="app/compose.yaml" hl_lines="5"
services:
  caddy:
    build:
      context: ./caddy
    image: ghcr.io/youruser/yourapp-caddy:0.1.0
```

Build and push your images:

```sh
cd app
docker compose build
docker compose push
```

## 📦 2. Copy to Server

Copy your `compose.yaml` to the remote host:

```sh
scp compose.yaml youruser@yourserver:
```

## 3. Set Secrets

Your app will need credentials such as database passwords or API keys. Choose
one of these approaches:

1. **`.env` file** — simply place a `.env` file alongside your `compose.yaml`.
   Be sure to `chmod 600 .env`.
2. **Environment variables** — pass secrets directly to the command line.
3. **CI/CD injection** — good for automated pipelines.

## 🚀 3. Launch the App

Start the application on the server:

```sh
docker compose up -d
```

Your backend is now live. 🚀

---

## Upgrading

To upgrade your app, simply increment the image tag versions in `compose.yaml`.

The rest is the same:

2. `docker compose build`
3. `docker compose push`
4. `scp compose.yaml yourserver:`
5. `docker compose up -d`

## 🧭 Next Steps

If you want zero-downtime deployments, rollback support, or blue-green testing,
continue to [Advanced Deployments](advanced.md).

## ⚡ GitHub Actions

Here's a Github Actions workflow you can use to automate deployments.

```sh
mkdir -p .github/workflows
```

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
          target: "app/"

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
            cp .env app/
            cd app

            # Pull images
            echo "$GHCR_PAT" | docker login ghcr.io --username "${{ github.actor }}" --password-stdin
            DOCKER_CLIENT_TIMEOUT=300 COMPOSE_HTTP_TIMEOUT=300 docker compose pull --quiet

            # Bring up stack and run healthchecks
            trap 'docker compose down' ERR
            docker compose up -d
            docker compose exec -T caddy curl -fsS http://caddy:80/healthz
            # Add more healthchecks here
            # docker compose exec -T caddy curl -fsS http://api:8080/healthz
            # docker compose exec -T caddy curl -fsS http://postgrest:3000/
```

</details>
