# yr-skills

Personal Claude Code skill collection.

## Skills

### all-push

Commit all uncommitted repository changes by functional grouping, then push the current branch.

- **Trigger**: When the user says "对所有未提交代码按照功能创建commit并push", "所有未提交代码创建commit并push", "创建commit并push", "commit并push", or "all-push"
- **Approach**: Inspects staged, unstaged, and untracked changes; protects primary branches with an explicit confirmation; groups related files into coherent commits; verifies the final state before commit/push when practical
- **Safety**: Stops before acting on `main`, `master`, `trunk`, `production`, `prod`, or the remote default branch; never force-pushes, rewrites commits, discards changes, or commits suspicious secret files silently

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

### read-docs

Read and inspect online documents from Feishu/Lark, DingTalk/AliDocs, and Tencent Docs.

- **Trigger**: When the user provides a `feishu.cn`, `larkoffice.com`, `larksuite.com`, `alidocs.dingtalk.com`, or `docs.qq.com` document link and asks to read, summarize, analyze, or extract its contents
- **Approach**: Routes by domain — uses `lark-cli` for Feishu, `dws` for DingTalk org docs, Tencent Docs MCP for QQ Docs, and falls back to the locally logged-in Chrome session when API/CLI/MCP is unavailable or insufficient
- **Supported sources**: Feishu (domestic & international), DingTalk (cross-org via Chrome), Tencent Docs (documents, sheets, smart sheets)
- **Verification**: Always reports the source method, document title, object type, and coverage completeness

### work-skill

Coordinate a clear goal across multiple child agents while the manager session only handles planning, dispatch, tracking, and summary.

- **Trigger**: When the user wants "目标 + 管理者 + 多 agent", says "当前会话作为管理者会话", or wants to use one goal to carry a very large program of work
- **Approach**: Splits the goal into phases, task graph, bounded child-agent tasks, and delegated implementation/testing/review/integration/acceptance roles
- **Large goals**: Requires durable planning artifacts such as goal brief, architecture/design, execution plan, and acceptance checklist before implementation dispatch
- **Boundary**: The manager does not develop, test, review, integrate, or accept; missing work is delegated to the appropriate child agent
