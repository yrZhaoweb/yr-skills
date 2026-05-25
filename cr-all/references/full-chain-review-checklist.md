# Full-Chain Review Checklist

Use this checklist when `cr-all` is applied to a multi-module feature review. Keep the final report focused on material findings; the checklist is for coverage, not for dumping every observation.

Full-chain review has a fixed spine and optional modules:

- Fixed spine: frontend/user entry, client API call, backend route/action, business logic, persistence/state, response, and user-visible result.
- Optional modules: provider calls, async jobs, cron/workers, billing/credits/refunds, storage/OSS, export/download, notifications, admin workflows, security-sensitive permissions, and observability.

Do not force optional modules into a report when the workflow does not contain them.

## 1. Discovery

- Read repository instructions (`AGENTS.md`, project docs, architecture notes).
- If a graph report or wiki exists and repo instructions require it, inspect it before architecture claims.
- Locate frontend routes/pages, navigation guards, forms, hooks, API client wrappers, backend routes, service layer, persistence models, and tests.
- Add optional modules only when present: cron/workers, storage helpers, billing/credit helpers, provider clients, notification jobs, admin tooling, and operational docs.
- Identify every implementation path for the same business function. Multiple parallel paths often create inconsistent validation, persistence, response, billing, transfer, and polling behavior.
- For each entry path, ask what happens if a user calls it directly with a valid token but without using the intended UI.
- Search for generic proxy, passthrough, record, update, PATCH, admin, debug, legacy, and helper endpoints that can perform the same side effect with weaker rules.
- Record whether the path is synchronous request/response, user-driven refresh, server-driven job, callback/webhook-driven, or cron-driven.

## 2. Entry And Input

- Confirm auth and role guards at page, API client, and backend levels.
- Check frontend-only validation against backend validation.
- Check limits for count, size, type, MIME, URL scheme, domain, and model-specific constraints.
- If URLs or files are accepted, check whether they can trigger SSRF-like behavior through provider fetches or server-side transfer jobs.
- If assets are uploaded or referenced, verify they are persisted in controlled storage before external use when permanence matters.
- Check i18n keys and visible errors are understandable enough for users and support.

## 3. Submission And Durable State

- Identify the first durable record created for the operation.
- Check whether a request can be retried safely after timeout or lost response.
- Check idempotency keys, unique indexes, duplicate task records, and duplicate submissions or side effects.
- Check whether state transitions are explicit and terminal states are irreversible unless intentionally reopened.
- Check whether the client is allowed to create records, choose terminal status, set failure reason, set success URLs, or trigger compensating actions. If yes, verify the server independently confirms the claim.
- Check for locks without TTL, in-memory-only flags, and async work launched after HTTP response without durable recovery.
- Check that compensation and retry guards use stable business identifiers. Localized strings, descriptions, UI messages, or provider text are not safe idempotency keys.

## 4. Billing, Credits, And Refunds (Optional)

Use this section only when the workflow changes money, credits, quotas, subscriptions, entitlements, usage counters, or refundable balances.

- Identify the single source of truth for balance.
- Check client-side optimistic balance changes against server-side committed balance.
- Verify deduction happens exactly once and is linked to a durable operation id.
- Verify refund happens exactly once and uses the original deduction breakdown.
- Check whether refund or compensation is guarded by an atomic business-state transition, such as `isRefunded != true -> true`, not only by a ledger text lookup.
- Check refund or compensation paths for relevant failures, such as provider failure after task creation, provider timeout, storage transfer failure, server crash, user tab close, and stale processing jobs.
- Check whether old data shapes make refund reconstruction unsafe.
- Check ledger/consumption records, related ids, descriptions, and auditability.

## 5. External Provider (Optional)

Use this section only when the workflow calls an upstream model, third-party API, payment processor, messaging service, analytics endpoint, or other external side effect.

- Check API keys, tokens, and secrets. Hardcoded usable credentials are P0.
- Verify provider request mapping for every model and input variant.
- Check that provider/proxy/helper endpoints cannot be used to run paid or restricted provider jobs while skipping the main workflow.
- Check response normalization, provider error code mapping, and terminal-state detection.
- Check rate limits, timeouts, retries, and whether retries can duplicate paid work.
- Check whether provider task ids are persisted before the client is told work started.
- Check provider URL expiration and whether raw provider URLs are ever stored as final user-facing URLs.

## 6. Polling, Callback, Cron, And Recovery (Optional)

Use this section only when the workflow continues after the initial HTTP response, uses asynchronous state, receives callbacks/webhooks, or depends on background jobs.

- Determine who advances the task after initial submission: browser polling, backend endpoint, callback, worker, or cron.
- Browser-only polling is insufficient for paid or persistent workflows unless a server fallback exists.
- Check stale `processing` jobs and maximum TTL behavior.
- Check batch partial failure and per-item recovery.
- Check server restarts while locks are held or async submission is mid-flight.
- For batch or multi-item flows, check durable per-item submission locks or atomic `pending -> processing` transitions before calling external side effects.
- Check lock TTL and release behavior. A lock without TTL can strand work; an in-memory lock can fail across processes.
- Check whether list endpoints refresh state or only return stored summaries.
- Check whether failure counters advance without requiring the user to keep polling manually.

## 7. Storage And Asset Permanence (Optional)

Use this section only when the workflow creates, imports, transforms, stores, previews, or exports durable files/assets.

- Verify remote outputs are transferred into controlled storage before being persisted as final result URLs.
- Check storage transfer failure behavior: retry, visible state, refund threshold, and terminal failure.
- Check whether transfer helpers silently return external URLs on failure.
- Verify persisted URLs are accepted only if they belong to expected internal storage, unless external permanence is guaranteed.
- Check update endpoints that accept arbitrary URLs. Server-side fetch of user-provided URLs needs scheme/domain/IP/MIME/size controls to avoid SSRF and arbitrary large downloads.
- For large assets, verify transfer timeout and retry policy fit file size. Short timeouts can create duplicate transfer attempts, false failure, or premature compensation.
- Check download behavior for cross-origin assets, content disposition, signed URLs, and expired links.
- Check generated ZIP/export flows and fallback behavior when an asset cannot be fetched.

## 8. Display, History, Export, And Download

- Check immediate preview and historical list use the same persisted source of truth.
- Check refresh can regress a completed item back to processing or remove a visible result.
- Check client request sequence guards, stale responses, and race conditions between tabs.
- Check optimistic UI updates against server truth. Balance, status, and result URLs need reconciliation after network errors and lost responses.
- Check delete/cancel behavior. If optional modules exist, include provider task, local record, storage object, and refund policy.
- If media or files are displayed, check preview components for cross-origin, format, and cache behavior.

## 9. Observability And Operations

- Logs should include the relevant identifiers for the workflow, such as user id, operation id, record id, provider task id, cost, model, and error code on material failures.
- Metrics should match the optional modules present, such as stuck processing counts, compensation counts, provider error rates, transfer failures, and batch partial failures.
- Admin/support views should allow tracing a user-visible task to the relevant backend record, and to provider or billing records only when those modules exist.
- Error handling should not hide provider messages that are needed for support, but should avoid leaking secrets.

## 10. Report Discipline

- Findings should be evidence-first and severity-ranked.
- Include a failure sequence for each major issue so the bug is reproducible in thought or tests.
- Call out already-good design choices when they materially reduce risk, but keep them brief.
- Keep recommendations scoped: immediate fixes for P0/P1, then reliability improvements, then cleanup.
- Suggest tests that prove the fix: unit tests for state transitions, integration tests for refund idempotency, and simulated provider/storage failures.
