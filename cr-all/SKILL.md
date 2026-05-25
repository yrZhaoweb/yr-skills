---
name: cr-all
description: "Expert full-chain code review workflow for feature and business-process audits. Use when the user asks for expert mode, end-to-end review, full-link review, full-chain review, audit, or risk triage of a workflow. Always trace the fixed application spine: frontend/user entry, client API call, backend route/action, business logic, data/state persistence, response, and user-visible result. Then add only the optional modules that actually exist in the workflow, such as external providers, async jobs, cron/workers, billing/credits/refunds, storage/OSS, preview/download, permissions, security, idempotency, and observability."
---

# CR All

Use this skill to perform evidence-backed, end-to-end code reviews of a product workflow. Reconstruct the actual execution path from user action to final persisted or user-visible outcome, then identify defects, missing fallbacks, and operational risks with concrete file/line evidence.

Full-chain does not mean every workflow contains payment, refunds, upstream providers, storage transfer, cron, or downloads. Treat the frontend-to-backend path as the fixed spine, then attach optional risk modules only when the code actually uses them.

## Core Rules

- Read the code directly. Do not rely on summaries, commit messages, or architecture guesses as the source of truth.
- Honor repository instructions first. If the repo has `AGENTS.md`, read it. If it mentions graphify and `graphify-out/wiki/index.md` or `graphify-out/GRAPH_REPORT.md` exists, inspect it before architecture conclusions.
- When skill context is needed, use the active Codex skill inventory from the current session and local Codex skills under `~/.codex/skills`. Do not use `~/.agents/skills` as the skill list source unless the user explicitly asks for it.
- Cite every finding with file and line references. If a conclusion is an inference, label it as such.
- Prefer severe, user-visible, money-affecting, data-loss, security, and stuck-state bugs over style comments.
- Distinguish proven defects from recommendations. Do not inflate severity for cleanup-only issues.
- Trace success, failure, timeout, retry, refresh, browser-close, process-crash, duplicate-request, and partial-completion paths when those states are meaningful for the workflow.
- Treat money, credits, refunds, payment, authentication, storage persistence, and external API protocol changes as escalation topics.
- Do not end reviews with calendar-bucketed plans such as "Immediate / This week / Later" unless the user explicitly asks for scheduling. Provide direct, prioritized solutions instead.

## Workflow

1. Define the audit scope in one sentence:
   - user entry point
   - inputs and validation
   - client API call
   - backend endpoint or action
   - business logic and data/state persistence
   - response and user-visible result
   - optional modules present in this flow, such as provider calls, billing, async jobs, storage, export/download, permissions, notifications, or audit logs

2. Gather context:
   - Read repo instructions and architecture notes.
   - If available skills or skill instructions matter to the review, read them from the current Codex skill inventory or `~/.codex/skills`, not from `.agents`.
   - Locate frontend pages/components, API clients, backend routes, services, models, persistence layers, tests, and relevant docs.
   - Add optional context only when present: cron/workers, storage helpers, payment/credit helpers, provider clients, notification jobs, admin tooling, and operational docs.
   - Use `rg` and parallel file reads where practical.
   - Build a route map before judging implementation details.

3. Build a lifecycle table:
   - entry path
   - user input source
   - API endpoint
   - auth/permission guard
   - persisted records
   - backend business action
   - response path
   - UI display or next state
   - optional: external request
   - optional: status polling/callback/cron
   - optional: credit deduction and refund source of truth
   - optional: storage transfer/persistence
   - optional: export/download path

4. Review each lifecycle stage:
   - Open page and load state
   - Fill inputs and upload/reference assets
   - Submit request and create durable records
   - Execute backend business logic
   - Persist changed state and return response
   - Present the user-visible result, history, next step, or error
   - If present, review payment/credits/refunds
   - If present, review upstream/provider calls
   - If present, review async polling/callback/worker terminal states
   - If present, review storage/OSS/export/download behavior
   - If present, review cancel/rollback/compensation and user-facing failure reasons

5. Stress the non-happy paths:
   - browser refresh or tab close
   - HTTP response lost after server-side mutation
   - duplicate submit or retry
   - stale client state versus server truth
   - optional: provider returns task id then later fails
   - optional: provider succeeds but storage transfer fails
   - optional: worker/process restarts during lock or async submission
   - optional: duplicate poll, duplicate callback, or duplicate webhook
   - optional: partial batch success
   - old data shape or migration leftovers

6. Produce a severity-ranked report:
   - Findings first, highest severity first.
   - For each finding include: severity, evidence, reproduction/failure sequence, impact, and concrete fix direction.
   - Add an architecture/lifecycle table only when it helps the reader.
   - End with a prioritized solution list that maps each fix to the relevant finding. Do not use time buckets like "this week" by default.

## High-Value Review Lenses

Apply these lenses when the relevant pattern exists in the code:

- Multiple entry points for the same business action: check whether one path bypasses validation, billing, permission checks, persistence, or audit logging.
- Client-trusted terminal state: reject designs where the browser can create paid records, mark success/failure, trigger refund, or set durable URLs without server verification.
- Compensation and idempotency: check whether retries, refunds, rollbacks, and webhook/poll repeats are guarded by a stable business key, not by mutable text like descriptions or localized messages.
- Async recovery: check whether work launched after an HTTP response has durable recovery via cron, worker, callback, queue, TTL, or an explicit stale-state policy.
- Concurrency and locks: check whether batch items, submissions, callbacks, and retries use atomic state transitions or durable locks instead of process-local memory.
- Asset permanence: check whether final user-visible asset URLs point to controlled storage, whether external URLs expire, and whether transfer failures can leave short-lived links in UI or history.
- Security boundary drift: check whether helper endpoints, PATCH/update endpoints, proxy routes, and legacy code accept broader inputs than the main workflow.

## Severity

- P0: Exploitable secret, money loss, unrecoverable data loss, permanent stuck state for common flows, or broad production outage risk.
- P1: Incorrect billing/UI truth, important workflow breakage, missing idempotency, stuck state with plausible trigger, or security weakness needing prompt fix.
- P2: Reliability, consistency, degraded user experience, missing fallback, or operational issue with bounded impact.
- P3: Maintainability, observability, redundant logic, edge-case validation, or cleanup recommendation.

## Report Template

```markdown
# Full-Chain Review: <workflow>

## 0. Architecture Map
| Flow | Entry | API/backend action | Persistence/state | User-visible result | Optional modules |
|---|---|---|---|---|---|

## 1. Scope
<One paragraph describing the audited lifecycle and assumptions.>

## 2. Findings
### P0 - <title>
- Evidence: `<file>:<line>`
- Failure sequence: ...
- Impact: ...
- Fix direction: ...

## 3. Lifecycle Notes
### Open page / inputs
### Submit / backend / persistence
### Success / display / next state
### Failure / rollback or compensation / user message
### Optional modules: upstream, billing, async jobs, storage, download, notifications

## 4. Solutions
1. Fix `<finding title>` by ...
2. Fix `<finding title>` by ...
3. Optional hardening: ...
```

## Deep Checklist

For complex audits, read `references/full-chain-review-checklist.md` and use it as a checklist. Load it only when the workflow spans several modules or when the user explicitly asks for a comprehensive review.
