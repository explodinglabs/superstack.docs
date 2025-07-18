# 🖥️ Using psql

`psql` is the command-line tool for interacting with your PostgreSQL
database. SuperStack makes it easy to run psql inside the container using a
helper script.

## 📟 Open a psql Shell

To connect interactively:

```sh
bin/postgres psql
```

Example output:

```
psql (17.5 (Debian 17.5-1.pgdg120+1))
Type "help" for help.

app=#
```

## 🔹 Run Inline SQL Commands

You can also run SQL directly from the command line:

```
bin/postgres psql -c 'select * from movie;'
```

## ⚙️ Customize psql Behavior

You can persist your preferences using `.psqlrc` and `.inputrc`.

### 🔧 Step 1: Create a config directory

```sh
mkdir -p postgres/rc
```

### 📄 .psqlrc

Create `postgres/rc/.psqlrc` with your preferred settings:

```
\pset pager off
\setenv PAGER 'less -S'
```

See the [official psql
reference](https://www.postgresql.org/docs/current/app-psql.html) for all
available options.

### 📄 .inputrc

Create `postgres/rc/.inputrc` to set readline behavior:

```
set editing-mode vi
```

## 🔗 Step 2: Mount and apply the configs

Add to your `compose.override.yaml` (this file is for development-only
overriding of `compose.yaml`):

```yaml
services:
  postgres:
    volumes:
      - ./postgres/rc:/rc:ro
    environment:
      PSQLRC: /rc/.psqlrc
      INPUTRC: /rc/.inputrc
```

## 🔁 Step 3: Restart the Postgres container

```sh
docker compose down postgres
docker compose up -d postgres
```
