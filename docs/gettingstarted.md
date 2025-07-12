# 🚀 Getting Started

<video controls width="100%">
  <source src="/superstack/assets/getting-started.mp4" type="video/mp4">
  <data 
    value="Music: Bensound, License: UZG5X7IWWLQOQEU1, Artist: Lunar Years" 
    hidden>
  </data>
  Your browser does not support the video tag.
</video>

SuperStack uses Docker, so make sure [Docker is
installed](https://docs.docker.com/get-docker/) before you begin.

## 1. Get SuperStack

### Option 1: Use the Template (Recommended)

The easiest way to get started:

Click [Use this template](https://github.com/explodinglabs/superstack/generate)
and create a new repository (e.g. `myapp`) on GitHub.

Clone it to your machine:

```sh
git clone https://github.com/yourname/myapp.git
cd myapp
```

### Option 2: Clone and Track Upstream (Advanced)

If you want to keep SuperStack’s Git history and pull upstream changes later,
clone SuperStack:

```sh
git clone https://github.com/explodinglabs/superstack.git myapp
cd myapp
```

[Create your own repo](https://github.com/new), then:

```sh
git remote rename origin upstream
git remote add origin https://github.com/yourname/myapp.git
git push -u origin main
```

You can now pull upstream changes with:

```sh
git pull upstream main
```

## 2. Configure Environment Variables

Copy the example environment file:

```sh
cp example.env .env
```

This `.env` file is used to configure:

- **Secrets** – Passwords, keys, etc.
- **Ports** – Adjust the exposed ports (specifically, Caddy's) depending on
  environment or application (you may bring up multiple).

> ⚠️ Important: The .env file is for local development only. Never store real
> secrets in version control or production. Use CI/CD environment variables or
> a secrets manager instead.

## 3. Start the Stack

```sh
docker compose up -d
```

That's it – your backend is live.

You can now open
[http://localhost:8000/openapi/](http://localhost:8000/openapi/) to explore
your API (assuming 8000 is your Caddy port).

---

## 🧩 What Just Happened?

SuperStack automatically:

1. Starts a fresh Postgres database
2. Applies initial migrations
3. Launches PostgREST and Swagger UI
4. Serves everything through Caddy

```mermaid
flowchart TD
    Caddy["Caddy (API Gateway)"]
    Caddy --> Services["Services (PostgREST, Swagger UI + more)"]
    Services --> Postgres
```

> 💡 Only Caddy exposes a port – all services are routed through it.

## Project Structure

```
📁 bin/                  → Helper scripts (e.g. wrappers for CLI tools)
📁 caddy/                → Custom Caddy configuration and certificates
📁 docs/                 → Markdown files for SuperStack documentation
📁 postgres/             → SQL migrations and configuration of the postgres container
📄 compose.yaml          → Main Docker Compose config
📄 compose.override.yaml → Optional local overrides (development only)
📄 example.env           → Example environment variables — copy to `.env`
📄 LICENSE               → License file (MIT)
📄 logo.png              → SuperStack logo for README/docs
📄 mkdocs.yml            → MkDocs configuration for documentation site
📄 README.md             → Overview and quick start for the repository
```

## 🔄 Resetting

If you want to start fresh:

```sh
docker compose down --volumes
docker compose up -d
```

This will wipe your database and re-run all migrations from scratch.

## ➕ What's Next?

👉 [Create your database schema with migrations](migrations.md)  
👉 [Deploy to a remote environment](deploying.md)
