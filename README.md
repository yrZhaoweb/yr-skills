# yr-skills

Personal Claude Code skill collection.

## Skills

### cr-all (codereview-all)

Full-chain code review skill. Reconstructs the complete execution path from user action to final outcome, then identifies defects, missing fallbacks, and operational risks with concrete file/line evidence.

- **Trigger**: When the user asks for expert mode, end-to-end review, full-link review, audit, or risk triage of a workflow
- **Approach**: Traces the fixed application spine (frontend entry -> client API call -> backend route -> business logic -> data persistence -> response -> user-visible result), then attaches optional modules (provider calls, billing, async jobs, storage, permissions, etc.) only when the code actually uses them
- **Output**: Severity-ranked findings (P0-P3) with evidence, failure sequence, impact, and fix direction
- **Reference**: `references/full-chain-review-checklist.md` for comprehensive audits spanning multiple modules

### ro-db (readonly-database)

Read-only database access through SSH tunnel to remote Docker containers.

- **Trigger**: When the user wants to inspect or query data from a database inside a Docker container on a remote server
- **Approach**: Infers connection details from local project config, creates an SSH tunnel bound to `127.0.0.1`, enforces read-only mode, runs scoped queries, and cleans up the tunnel
- **Supported databases**: PostgreSQL, MySQL/MariaDB, MongoDB, Redis
- **Safety**: Hard-refuses any mutating statement (`INSERT`, `UPDATE`, `DELETE`, `DROP`, `ALTER`, etc.), never exposes credentials, never changes server configuration
