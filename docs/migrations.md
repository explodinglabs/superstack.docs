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

This will:

1. Run any migration files that haven’t been applied yet (in filename order)
2. Record each successfully applied file in `.applied_migrations` (this
   file lives in the postgres data volume)

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

Use `begin;` and `commit;` to wrap statements in a transaction. This
ensures that all changes are applied atomically. Any statements outside of
transactions will be auto-committed.

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

SuperStack doesn't require any file names or layout, that's up to you.

But here's one you might use while developing (pre-production):

```
01-extensions.sql
02-create_auth_schema.sql  (if using PostgREST for auth)
03-create_api_schema.sql
04-roles.sql
05-grants.sql
```

During development, make changes and:

```sh
docker compose down --volumes; docker compose up -d
```

Once the app's reached production (or some other environment), you can no
longer keep recreating the database from scratch. Add smaller migrations from
`06-` onwards, and use `bin/postgres migrate` (that's a shortcut you can use
locally, on other environments the `bin/postgres` script isn't there so use
`docker compose exec postgres migrate`).

## 🔄 Nuke Everything

If you want to start fresh, wipe your database and re-run all migrations from
scratch:

```sh
docker compose down --volumes
docker compose up -d
```
