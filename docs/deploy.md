# ☁️ Deploying to Remote Environments

SuperStack is **Docker-native**, so deployments are simple, consistent, and
portable. The goal is that only `compose.yaml` and secrets need to exist on the
remote server.

## Prepare your Application

Services that are built (they have a `build:` section in `compose.yaml`), add
your own container repository image URIs (e.g. your Docker Hub or GitHub
Container Registry account), for example:

```yaml title="compose.yaml"
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

## 📦 Deploy

Copy `compose.yaml` to the server:

```sh
scp compose.yaml youruser@yourserver:
```

### Secrets

The app also needs your secrets (passwords, keys, etc.).

There are a few options:

1. Put secrets in a `.env` file on the server, alongside your compose.yaml.
   Convenient but Less secure. Be sure to `chmod 600 .env`.
1. Set environment variables in the the `docker compose` command. Inconvenient.
   Be sure to disable shell history.
1. Use environment injection in CI/CD.

## 🚀 Launch your Stack

Bring up the stack:

```sh
docker compose pull
docker compose up -d
```

---

That’s it — your backend is live.

This type of rolling deployment makes it hard to test before going live and to
rollback, and can have some downtime while upgrading. Consider reading the Wiki
page on [Advanced Deployments]().

## Github Actions Workflow

TODO

---

## ➕ What Now?

Add to your app by following guides in the [Wiki](https://github.com/explodinlabs/superstack/wiki).
