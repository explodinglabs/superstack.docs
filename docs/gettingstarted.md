# 🚀 Getting Started

<!--video controls width="100%">
  <source src="/superstack/assets/getting-started.mp4" type="video/mp4">
  <data
    value="Music: Bensound, License: UZG5X7IWWLQOQEU1, Artist: Lunar Years"
    hidden>
  </data>
  Your browser does not support the video tag.
</video-->

SuperStack uses Docker, so make sure [Docker is
installed](https://docs.docker.com/get-docker/) before you begin.

## 1. Get SuperStack

### Option 1: Use the Template (Easiest)

Click [Use this template](https://github.com/explodinglabs/superstack/generate)
and create a new repository (e.g. `myapp-backend`) on GitHub.

Clone it to your machine:

```sh
git clone https://github.com/yourname/myapp-backend.git
cd myapp-backend
```

### Option 2: Clone and Track Upstream (Advanced)

If you want to keep SuperStack’s Git history and pull upstream changes later,
clone SuperStack:

```sh
git clone https://github.com/explodinglabs/superstack.git myapp-backend
cd myapp-backend
```

[Create your own repo](https://github.com/new), then:

```sh
git remote rename origin upstream
git remote add origin https://github.com/yourname/myapp-backend.git
git push -u origin main
```

You can now pull upstream changes with:

```sh
git pull upstream main
```

## 2. Configure Environment Variables

Copy the example environment file:

```sh
cd app
cp example.env .env
```

The `.env` file is used to set secrets, passwords, keys, etc.

> ⚠️ Never store secrets in version control.

## 3. Start the App

```sh
docker compose up -d
```

That's it – your backend is live.

Test it with:

```sh
$ curl http://localhost:8000/healthcheck
OK
```

---

## ➕ What's Next?

👉 [Add services by following the Wiki](https://github.com/explodinglabs/superstack/wiki)  
👉 [Deploy to a remote environment](deploy.md)
