# 🧩 Postgres Extensions

SuperStack supports PostgreSQL extensions, letting you add powerful
features like cryptographic functions or JWT handling.

## 🛠️ Building an Extension from Source

Some extensions (like [pgjwt](https://github.com/michelp/pgjwt)) must be
compiled manually.

### 1. Clone the Extension Source

```sh
git clone https://github.com/michelp/pgjwt postgres/pgjwt
```

### 2. Modify the Postgres Dockerfile

Edit `postgres/Dockerfile` to install build tools and compile the
extension:

```
RUN apt-get update && apt-get install -y \
    build-essential \
    postgresql-server-dev-17

# pgjwt - used by the auth schema
COPY ./pgjwt /pgjwt
WORKDIR /pgjwt
RUN make && make install

# Reset workdir
WORKDIR /var/lib/postgresql
```

> 🧼 Set `WORKDIR` back to the default to avoid unintended effects.

### 3. Rebuild the Container

```sh
docker compose build postgres
```

That’s it — the extension is now available to load in your migrations.

## 🔌 Loading an Extension

To load a standard extension (like pgcrypto), create a migration file such as:

```sql title="postgres/migrations/01-extensions.sql"
create extension pgcrypto;
```

> ⚠️ `create extension` is non-transactional, so don’t wrap this in
> `BEGIN/COMMIT`.
