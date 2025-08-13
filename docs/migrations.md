# 📜 Migrations

SuperStack includes a simple built-in system for managing database schema
migrations.

## ✍️ Writing Migrations

Place your migration scripts in:

```
postgres/migrations/
```

Each file should be:

- An `.sql` file
- Numbered in order (e.g. `00-init.sql`, `01-extensions.sql`, `02-auth.sql`)
- Written in plain SQL
- But can include environment variables.

## ▶️ Applying Migrations

When the Postgres container starts with no existing data, SuperStack will
automatically run migrations once.

After the first startup, migrations will only run if you manually apply
them.

To apply your migrations, run:

```sh
bin/postgres migrate
```

This command will:

1. **Apply new migrations,** in filename order.
2. **Record applied migrations** in a file named `.applied_migrations`.

Already-applied scripts are skipped on subsequent runs.

> 💡 `bin/postgres` is a small script that effectively aliases `docker compose exec postgres`

Here's an example migration script:

```sql title="postgres/migrations/02-create_table_example.sql"
begin;

create table director (
  id serial primary key,
  name text not null
);

create table movie (
  id serial primary key,
  name text not null,
  director_id integer references director(id)
);

commit;
```

## 🔁 Transactions

Use `begin;` and `commit;` to wrap statements in a transaction. This ensures
that all changes are applied atomically. Any statements outside of transactions
will be auto-committed.

Avoid wrapping non-transactional operations in a transaction — these will
cause errors if used inside `begin ... commit`. Examples of
non-transactional statements include:

```sql
ALTER SYSTEM
CLUSTER
CREATE DATABASE
CREATE EXTENSION
CREATE ROLE
CREATE TABLESPACE
DISCARD ALL
DROP DATABASE
DROP EXTENSION
DROP TABLESPACE
LOAD
REINDEX
VACUUM
```

## Suggested File Layout

SuperStack doesn’t enforce any particular migration file names or layout, but here’s a simple structure you might adopt during development (before
production):

```
01-extensions.sql
02-auth_schema.sql  (if using PostgREST for auth)
03-api_schema.sql
04-roles.sql
05-grants.sql
```

While developing, you can reset and rebuild the database from scratch as often
as needed:

```sh
docker compose down --volumes; docker compose up -d
```

Once you’ve deployed to production (or another persistent environment), avoid
recreating the database. Instead:

- Add new migrations starting from `06-...` onwards.
- Apply them with:

```sh
bin/postgres migrate
```

In other environments where `bin/postgres` isn't available:

```sh
docker compose exec postgres migrate
```

This approach keeps early development simple while providing a clear, ordered
history once the database must be preserved.

## 🔄 Nuke Everything

If you want to start fresh, wipe your database and re-run all migrations from
scratch:

```sh
docker compose down --volumes
docker compose up -d
```
