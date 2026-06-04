---
name: work-skill
description: Use when a user wants one manager session to coordinate a clear goal across multiple agents, modules, workstreams, task packs, or a very large single objective, especially when all implementation, testing, review, and acceptance work must be delegated.
---

# Work Skill

## Overview

Use this skill to keep the current session as the manager: define the goal, split all executable work, delegate to agents, track progress, and summarize outcomes.

The manager owns coordination only. Development, testing, review, integration, and acceptance are all delegated to child agents.

## When to Use

Use when the user asks for or implies:

- "目标 + 管理者 + 多 agent"
- "当前会话作为管理者会话"
- "按照功能模块，每个模块启用一个子 agent"
- "管理者不参与开发/测试/review/验收"
- "用这个 skill 承载一个巨大目标"
- A broad task with separable modules, routes, files, owners, rounds, or task packs
- A large program or long-running build that must be decomposed into phases and child-agent tasks
- Parallel investigation where independent findings must be coordinated into one answer
- A delivery goal where execution, verification, review, and acceptance should be handled by separate agents

Do not use when:

- The task is small enough for one direct pass
- Workstreams touch the same files or shared state so strongly that parallel edits would conflict
- The user only wants a quick answer, not execution
- The environment has no usable agent/delegation tool; ask whether to switch to a single-agent workflow instead

## Manager Contract

1. Restate the concrete goal in one sentence.
2. Define success criteria before work starts.
3. List scope boundaries, including explicit non-goals.
4. Split by stable boundaries: module, route, feature, data source, test layer, or document section.
5. Assign every executable task to a child agent, including development, testing, review, integration, and acceptance.
6. Track ownership, dependencies, blockers, and evidence returned by each agent.
7. Summarize only after the assigned acceptance agent has reported the acceptance result.

If requirements are ambiguous, surface the ambiguity before dispatch. Do not let workers guess product or scope decisions independently.

## Task Split Rules

Good worker tasks are independent and have a clear output:

- "Fix CRM analytics filter counts; report changed files, tests, and remaining risk."
- "Inspect imageWash task creation flow; do not edit TikTok code; return root cause and patch."
- "Review R3 docs for missing gates; provide file:line evidence and proposed text."
- "Run acceptance for the completed slices; report pass/fail, evidence, and unresolved risk."

Bad worker tasks are vague or overlapping:

- "Improve the app."
- "Check everything."
- "One agent edits frontend while another edits the same component."

Prefer 2-5 workers for execution plus separate agents for testing/review/acceptance when needed. More workers only help when boundaries are clean and the manager can still coordinate dependencies.

## Large Goal Mode

A single goal may be huge, but no child-agent task may be huge. Treat the goal as a program of work:

```text
One huge goal
-> phases
-> task graph
-> small child-agent tasks
-> testing/review/integration/acceptance agents
-> manager summary
```

Before dispatching implementation for a huge goal, assign child agents to produce the durable planning artifacts:

- **Goal brief**: user outcome, constraints, non-goals, and acceptance criteria.
- **Architecture/design**: module boundaries, data contracts, routes, permissions, external dependencies, and integration points.
- **Execution plan**: phases, task IDs, dependencies, owners, expected artifacts, and stop/go gates.
- **Acceptance checklist**: scenario-level checks that an acceptance agent can run without relying on manager judgment.

Use phases when the task graph is too large to fit in one reliable dispatch. A phase should have a visible deliverable and an acceptance task. The manager may start the next phase only after the assigned acceptance agent reports the current phase result.

Do not dispatch a child agent with "build the whole app" or "finish the entire system." Split until each task has a bounded file/module area, expected output, and evidence requirement.

## Required Agent Roles

For delivery work, delegate these roles instead of doing them in the manager session:

- **Implementation agent**: changes code, docs, data artifacts, or configuration.
- **Testing agent**: runs focused checks and reports command/runtime evidence.
- **Review agent**: reviews implementation quality, regressions, and scope adherence.
- **Acceptance agent**: decides whether the user goal is met and reports pass/fail with evidence.
- **Integration agent**: merges or reconciles slices when multiple implementation agents touch related areas.

One child agent can hold multiple roles only when the task is small and independence is not compromised. The manager must not take any of these roles.

## Worker Prompt Template

```text
Goal: <overall user goal>
Your slice: <bounded module/workstream>
Scope: <allowed files/features/data>
Non-goals: <what to avoid>
Expected output:
- Findings or changes
- Evidence: commands, tests, logs, file:line references, screenshots, or data rows
- Risks or unresolved blockers
Rules:
- Do not modify unrelated areas.
- Do not self-accept unless you were explicitly assigned the acceptance role.
- Stop and report if scope is coupled with another slice.
```

## Manager Loop

1. **Plan**: Create a short task list with one `in_progress` manager step.
2. **Dispatch**: Start independent workers only after scope, success criteria, roles, and dependencies are clear.
3. **Monitor**: Track each worker as pending, running, done, blocked, or needs review.
4. **Coordinate handoffs**: Route implementation outputs to testing, review, integration, and acceptance agents.
5. **Completeness check**: Confirm each required role returned its report. Do not perform technical review or acceptance yourself.
6. **Report**: Give the user the delegated final state, proof provided by agents, changed boundaries, and residual risk.

Never develop, test, review, or accept inside the manager session. If an acceptance report is missing, dispatch an acceptance agent instead of deciding yourself.

## Delegated Evidence Standard

Assign child agents to gather evidence based on risk:

- Code: focused tests, typecheck/build, lint, targeted runtime request, or reproduced bug path
- UI: screenshot/browser state, interaction result, visible text/state, console/network errors when relevant
- Data/export: inspect the generated file/table rows directly
- Docs/plans: file:line review, link/anchor validation, consistency against task gates
- Git/release: branch, remote, commit hash, clean/dirty status, push result

If a check cannot run, the responsible child agent must say exactly why and what evidence remains. The manager reports that status without converting it into acceptance.

## Common Failure Modes

- **Manager starts doing the work**: stop, create the missing child-agent task, and delegate it.
- **Workers overlap**: pause, merge scope, and reassign narrower tasks.
- **No explicit non-goals**: add them before dispatch, especially for nearby but excluded modules.
- **Too many agents**: reduce to the few independent slices that affect the goal.
- **No acceptance agent**: create one; the manager must not decide pass/fail.

## Output Shape

For substantial work, report:

```text
Goal: <done / partial / blocked>
Slices handled: <module list>
Evidence: <testing/review/acceptance reports from child agents>
Changed: <files, artifacts, commits, or docs>
Residual risk: <only real remaining risk, or "none known">
```
