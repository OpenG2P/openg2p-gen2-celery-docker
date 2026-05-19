# openg2p-gen2-celery-docker

Docker images for OpenG2P Celery beat/producers and workers.

## Celery jobs database bootstrap

Workers and beat use PostgreSQL (**`CELERY_JOBS_*_DB_DBNAME`**, commonly `{{ release }}_db` in Helm).

Schema and optional seed scripts live in the **`openg2p-gen2-celery`** repo under **`db-scripts/`**:

1. **`g2p_celery_job_tables.sql`** — creates `g2p_external_data_providers`, `g2p_external_data_queue`, and `g2p_external_data_payloads` from the definitions in **`openg2p-celery-job-models`**.
2. **`seed_data.sql`** — optional; seeds a single **`death`** external data provider row with **`data_model` = `CRVSVC`**. Queue and payload tables are **not** seeded (processing data only).

Example (adjust host, DB name, user, and paths to your checkout):

```bash
psql "postgresql://${CELERY_JOBS_WORKER_DB_USERNAME}:${CELERY_JOBS_WORKER_DB_PASSWORD}@${CELERY_JOBS_WORKER_DB_HOSTNAME}:${CELERY_JOBS_WORKER_DB_PORT}/${CELERY_JOBS_WORKER_DB_DBNAME}" \
  -f /path/to/openg2p-gen2-celery/db-scripts/g2p_celery_job_tables.sql

psql "postgresql://${CELERY_JOBS_WORKER_DB_USERNAME}:${CELERY_JOBS_WORKER_DB_PASSWORD}@${CELERY_JOBS_WORKER_DB_HOSTNAME}:${CELERY_JOBS_WORKER_DB_PORT}/${CELERY_JOBS_WORKER_DB_DBNAME}" \
  -f /path/to/openg2p-gen2-celery/db-scripts/seed_data.sql
```

On first start, **beat** also runs ORM `create_migrate()` for the same tables; the SQL file is the operator-friendly, single-file DDL reference.
