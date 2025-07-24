<style>
  .logo-responsive {
    display: block;
    float: right;
    padding: 0 1em 0 2em;

    @media (max-width: 768px) {
      float: none;
      display: block;
      text-align: center;
      margin: 0 auto 1em auto;
      padding: 0;
    }
  }
</style>

<img src="assets/logo.png" alt="SuperStack Logo" class="logo-responsive" />

# SuperStack

Jump to:
[GitHub](https://github.com/explodinglabs/superstack) | [Developer Wiki](https://github.com/explodinglabs/superstack/wiki)

_SuperStack_ is a minimal, modular backend powered by PostgreSQL — perfect for
indie developers, SaaS builders, and teams who want full rontrol without the
bloat.

Spin up a fully working backend in seconds. Just clone, run, and start
building.

---

## 🚀 What Can I Do with SuperStack?

It's perfect for:

- 🧱 Building SaaS apps
- 💻 Running multiple stacks locally
- 📦 Easy database migrations
- 🔧 Customizing your toolchain

Everything runs inside Docker and routes through a single exposed port (via
Caddy), making it easy to develop locally or deploy remotely.

---

## 🏛️ Architecture

```mermaid
flowchart TD
    APIGateway["API Gateway (Caddy)"]
    APIGateway --> Services["Services (PostgREST, + add more)"]
    Services --> Database["Database (Postgres)"]
```

---

## 📚 What's next?

👉 [Getting Started](gettingstarted.md) – a guide to installing SuperStack and
launching the stack.
