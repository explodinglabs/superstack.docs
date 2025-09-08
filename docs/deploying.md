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
  image: ghcr.io/youruser/yourapp-caddy

postgres:
  build:
    context: ./postgres
  image: ghcr.io/youruser/yourapp-postgres
```

## 🛠️ 2. Build and Push your Images

Build your images locally and push to your registry:

```sh
docker compose build
docker compose push
```

## 📦 3. Deploy the Compose File

Copy `compose.yaml` to your server:

```sh
scp compose.yaml youruser@yourserver:
```

## 🚀 4. Launch your Stack

SSH into your server and bring up the stack.

For production, avoid using `.env` files. Instead, set secrets directly:

```sh title=".env"
JWT_SECRET=your-secret \
CADDY_PORT=80 \
PG_USER=admin \
PG_PASS=supersecret \
POSTGREST_AUTHENTICATOR_PASS=supersecret \
docker compose up -d
```

> 💡 Avoid leaking secrets by disabling shell history.

Alternatively, use environment injection in your CI/CD.

---

That’s it — your backend is live.

If this is the first time bringing up your stack, the migrations will run
automatically. Subsequently, to upgrade your app you should:

```sh
docker compose pull
docker compose up -d
docker compose exec postgres migrate
```
