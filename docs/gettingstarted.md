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

Click _**[Use this template](https://github.com/explodinglabs/superstack/generate)**_
and create a new repository (e.g. `myapp-backend`) on GitHub.

Clone it to your machine:

```sh
git clone https://github.com/yourname/myapp-backend.git
cd myapp-backend
```

## 2. Start the App

```sh
docker compose up -d
```

That's it – your backend is ready for development.

Test it with:

```sh
$ curl http://localhost:8000/healthz
OK
```

---

## ➕ What's Next?

👉 [Add services by following the Wiki](https://github.com/explodinglabs/superstack/wiki)  
👉 [Deploy to a remote environment](deploy.md)
