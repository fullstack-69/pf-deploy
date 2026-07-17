# Prefight - Deployment

> Note: This version allow multiple instances of the app to be deployed on the same VM.

## Get started

- Make `.env` from `.env.example` (Make necessary changes.)
- Take care of `./_entrypoint/init.sh`
  - Windows: Make sure that you save with LF option.
  - Mac: `chmod +x ./\_entrypoint/init.sh`
- `docker compose up -d --force-recreate`

## Deployment Note

There was a startup race between the backend migration step and the Postgres initialization script in `./_entrypoint/init.sh`.

`depends_on` only waits for the database container to start, not for Postgres to finish accepting connections or for the init script to complete. On slower machines, the backend could try to run `pnpm run db:migrate` before the database user and credentials created by the init script were ready, which caused the migration step to fail even though the container later worked fine when run manually.

The compose update addresses this by adding a Postgres healthcheck and making the backend wait for `service_healthy` before starting its migration hook. That changes the dependency from "container started" to "database is actually ready".
