---
name: ro-db
description: Use when inspecting or querying data from a database inside a Docker container on a remote server through an SSH tunnel, especially when the user wants read-only access based on local project configuration.
---

# RO DB

## Overview

Use this skill to answer data questions from a database running in a remote Docker container without exposing the database publicly or changing any data. The workflow is intentionally conservative: infer connection details from local code, create a local SSH tunnel, run only read-only queries, and close the tunnel when finished.

## Non-Negotiable Safety Rules

- Never run any statement that changes data, schema, users, permissions, replication, extensions, files, server settings, locks, or database lifecycle.
- Never run `INSERT`, `UPDATE`, `DELETE`, `UPSERT`, `MERGE`, `REPLACE`, `TRUNCATE`, `DROP`, `ALTER`, `CREATE`, `GRANT`, `REVOKE`, `VACUUM`, `ANALYZE`, `REINDEX`, `CLUSTER`, `CALL`, `DO`, `COPY ... TO/FROM PROGRAM`, `LOAD DATA`, `SET GLOBAL`, `RESET`, `KILL`, `SHUTDOWN`, `FLUSH`, or administrative equivalents.
- Never use `docker exec` to open a database shell inside the remote container unless the user explicitly authorizes that exact command and it is still read-only.
- Never expose the remote database to `0.0.0.0` or change Docker, firewall, compose, systemd, database, or application configuration.
- Never print passwords, tokens, SSH keys, connection strings, or full environment files in the final answer.
- Never put passwords directly in command arguments or database URLs when an interactive prompt, local environment variable, or client config file can be used.
- Prefer a database user that is already read-only. If credentials look privileged, warn the user and keep queries extra narrow.
- If a requested query requires mutation or risky administration, refuse that part and offer a read-only alternative.

## Success Criteria

Before claiming success, verify:

- The database host, port, type, and credentials came from local project configuration or explicit user input.
- The SSH tunnel binds only to `127.0.0.1` on a local ephemeral port.
- The database session is read-only where the database supports it.
- Every executed query is read-only and scoped with `LIMIT` or an aggregate unless the user explicitly needs a small full result set.
- The SSH tunnel process is closed or intentionally left running at the user's request.

## Workflow

### 1. Clarify the Target

Gather only the missing facts needed to connect:

- SSH target or alias, such as `prod-app` from `~/.ssh/config`.
- Project path containing Docker or app configuration.
- Database engine: PostgreSQL, MySQL/MariaDB, MongoDB, Redis, SQLite, or other.
- User's data question, not just a raw query when possible.

If the user has not named a server or project, ask for it. Do not scan unrelated directories or try random SSH hosts.

### 2. Read Local Configuration

Inspect local files first. Useful search targets:

```bash
rg --files -g 'docker-compose*.yml' -g 'compose*.yml' -g '.env*' -g '*database*' -g '*db*' -g 'application*.yml' -g 'application*.properties' -g 'settings.py' -g 'config*.{js,ts,json,yml,yaml}'
rg -n "POSTGRES|MYSQL|MARIADB|MONGO|REDIS|DATABASE_URL|DB_HOST|DB_PORT|DB_NAME|DB_USER|DB_PASSWORD|5432|3306|27017|6379"
```

Use structured parsers when practical for YAML, JSON, TOML, or framework config. Extract only the minimum connection facts:

- Service or container name
- Internal host and port
- Host-published port if present
- Database name
- Username
- Password source, without printing the secret
- Environment variable indirection

If credentials live outside the repo, ask the user to provide or export them locally. Do not fetch secret files from the server.

### 3. Choose a Tunnel Shape

Prefer the least invasive option supported by the local configuration:

1. If compose publishes the DB on remote loopback, tunnel to that host port:

```bash
ssh -N -L 127.0.0.1:${LOCAL_PORT}:127.0.0.1:${REMOTE_DB_PORT} ${SSH_TARGET}
```

2. If the DB is only reachable on the Docker bridge network and local config identifies the container host or IP, tunnel to that internal endpoint:

```bash
ssh -N -L 127.0.0.1:${LOCAL_PORT}:${CONTAINER_HOST_OR_IP}:${DB_PORT} ${SSH_TARGET}
```

3. If the endpoint cannot be inferred from local code, ask the user for the remote host/port or for permission to run a specific read-only inspection command such as `docker ps` or `docker inspect`. Do not improvise broader remote exploration.

Choose an unused high local port and bind only to `127.0.0.1`. Keep the SSH process in an identifiable terminal/session so it can be stopped cleanly.

### 4. Prove Read-Only Mode

After connecting, set read-only mode before running the user's query when the database supports it.

PostgreSQL:

```sql
BEGIN READ ONLY;
SET LOCAL statement_timeout = '10s';
SHOW transaction_read_only;
```

MySQL/MariaDB:

```sql
SET SESSION TRANSACTION READ ONLY;
START TRANSACTION READ ONLY;
SELECT @@session.transaction_read_only;
```

MongoDB:

- Use only read operations such as `find`, `aggregate` without `$out` or `$merge`, `countDocuments`, and metadata reads.
- Add `.limit(...)` for exploratory reads.

Redis:

- Use only inspection commands such as `GET`, `MGET`, `HGET`, `HGETALL`, `LRANGE`, `ZRANGE`, `SCAN`, `TTL`, `TYPE`, and `INFO`.
- Never run write, eviction, persistence, config, replication, script, or admin commands.

If read-only mode cannot be enforced, say so before querying and keep the query narrow.

### 5. Validate the Query

Accept only read-only intent.

Allowed SQL patterns:

- `SELECT`
- `WITH ... SELECT`
- `SHOW`, `DESCRIBE`, or database-specific metadata reads
- `EXPLAIN` for a read-only `SELECT`

Required safeguards:

- Add `LIMIT` for row-returning exploration.
- Prefer aggregates for counts, sums, and existence checks.
- Avoid `SELECT *` unless the table is known to be small or the user specifically needs raw rows.
- Avoid long-running cross joins, unbounded text/blob reads, and full-table exports.
- Run one statement at a time unless all statements are read-only setup plus one read-only query.

If the user supplies a risky query, rewrite it into a read-only equivalent and explain the change briefly.

### 6. Return Results Safely

Summarize results instead of dumping large tables. Mask secrets and sensitive identifiers when they are not essential to the user's question.

Include:

- What data source was queried, without secrets.
- The read-only query or a concise paraphrase.
- Key rows, counts, or findings.
- Any limitations, such as sampling, `LIMIT`, timeout, or missing read-only enforcement.

### 7. Cleanup

Close the SSH tunnel when the query is done unless the user asks to keep it open. If the tunnel stays open, state the local port and how to stop the process.

## Refusal Examples

- User asks to update a row: refuse the mutation and offer a `SELECT` to preview affected rows.
- User asks to drop or rebuild tables: refuse and offer schema inspection queries.
- User asks for a production credential dump: refuse and offer aggregate or masked inspection.
- User asks to run arbitrary remote commands: ask for a specific read-only purpose and command.

## Quick Query Templates

PostgreSQL:

```bash
PGPASSWORD="${DB_PASSWORD}" psql \
  --host=127.0.0.1 \
  --port="${LOCAL_PORT}" \
  --username="${DB_USER}" \
  --dbname="${DB_NAME}" \
  --set ON_ERROR_STOP=on \
  --command "BEGIN READ ONLY; SET LOCAL statement_timeout = '10s'; SELECT ... LIMIT 50; ROLLBACK;"
```

MySQL/MariaDB:

```bash
mysql --host=127.0.0.1 --port="${LOCAL_PORT}" --user="${DB_USER}" --password \
  --database="${DB_NAME}" \
  --execute="SET SESSION TRANSACTION READ ONLY; START TRANSACTION READ ONLY; SELECT ... LIMIT 50; ROLLBACK;"
```

MongoDB:

```bash
mongosh --host 127.0.0.1 --port "${LOCAL_PORT}" \
  --username "${DB_USER}" --password \
  --authenticationDatabase "${DB_NAME}" \
  --eval 'db.collection.find(query, projection).limit(50).toArray()'
```
