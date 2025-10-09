# ☁️ Deploying to Remote Environments

SuperStack is **Docker-native**, so deployments are simple, consistent, and
portable. The goal is that only `compose.yaml` and secrets need to exist on the
remote server.

## 🧱 1. Build Your Images

If a service has a `build:` section, add your own image name and version tag:

```yaml title="compose.yaml" hl_lines="5"
services:
  caddy:
    build:
      context: ./caddy
    image: ghcr.io/youruser/yourapp-caddy:0.1.0
```

Build and push your images:

```sh
docker compose build
docker compose push
```

## 📦 2. Copy to Server

Copy your `compose.yaml` to the remote host:

```sh
scp compose.yaml youruser@yourserver:
```

### 3. Set Secrets

Your app will need credentials such as database passwords or API keys.
Choose one of these approaches:

1. **`.env` file** — simplest for manual deploys.

```sh
chmod 600 .env
```

2. **Environment variables** — pass them directly to the command line.
3. **CI/CD injection** — good for automated pipelines.

## 🚀 Launch the App

Start the application on the server:

```sh
docker compose pull
docker compose up -d
```

Your backend is now live. 🚀

---

## 🧭 Next Steps

This setup replaces the stack in place.

If you want zero-downtime deployments, rollback support, or blue-green testing,
see the Wiki page: [Advanced
Deployments](https://github.com/explodinlabs/superstack/wiki).
