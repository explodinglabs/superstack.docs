# ☁️ Deploying to Remote Environments

SuperStack is Docker-native, so deployment is simple and portable. Here's how
to deploy it to a remote server.

A goal of SuperStack is that only `compose.yaml` should be required on the
remote server.

## ✅ 1. Prepare your Images

For services that are built (they have a `build:` section in `compose.yaml`),
add your own container repository image URIs (e.g. your Docker Hub or GitHub
Container Registry account), for example:

```yaml title="compose.yaml"
caddy:
  build:
    context: ./caddy
  image: ghcr.io/youruser/yourapp-caddy:0.1.0

postgres:
  build:
    context: ./postgres
  image: ghcr.io/youruser/yourapp-postgres:0.1.0
```

## 🛠️ 2. Build and Push your Images

Build your images locally and push to your registry:

```sh
docker compose build
docker compose push
```

## 📦 3. Deploy the Compose File

Copy `compose.yaml` to the server:

```sh
scp compose.yaml youruser@yourserver:
```

### 4. Set Secrets

The stack needs your secrets (passwords, keys, etc.). There are a few options:

1. Put secrets in a `.env` file on the server. Convenient but Less secure. Be
   sure to `chmod 600 .env`.
1. Set environment variables in the the `docker compose` command. Inconvenient.
   Be sure to disable shell history.
1. Use environment injection in your CI/CD.

## 🚀 5. Launch your Stack

SSH into your server and bring up the stack:

```sh
docker compose pull
docker compose up -d
```

---

That’s it — your backend is live.

If this is the first time bringing up your stack, the migrations will run
automatically. Subsequently, to upgrade your app you should:

```sh
docker compose pull
docker compose up -d
docker compose exec postgres migrate
```
