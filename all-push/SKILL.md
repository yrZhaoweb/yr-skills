---
name: all-push
description: Git workflow for committing all uncommitted code by functional grouping and pushing to the remote. Use when the user says "对所有未提交代码按照功能创建commit并push", "所有未提交代码创建commit并push", "创建commit并push", "commit并push", "all-push", or otherwise asks Codex to turn the current worktree's pending changes into one or more feature-scoped commits and push them. Requires stopping for explicit user confirmation before committing or pushing from a primary branch such as main, master, trunk, production, or the remote default branch.
---

# All Push

Use this skill to finish a repository's local work by creating coherent commits for every uncommitted change, then pushing the result.

## Success Criteria

- Account for every staged, unstaged, and untracked file before committing.
- Keep commits grouped by user-visible function or fix, not by file type or random edit order.
- Stop and ask before committing or pushing when the current branch is a primary branch.
- Leave no uncommitted changes unless the user explicitly chooses to exclude them.
- Push the committed branch and report commit hashes plus verification performed.

## Workflow

1. Inspect repository state:
   - Run `git status --short --branch`.
   - Run `git branch --show-current`.
   - Run `git remote -v`.
   - Inspect changes with `git diff --stat`, `git diff --name-status`, `git diff --cached --name-status`, and `git ls-files --others --exclude-standard`.
   - If needed, read focused diffs with `git diff -- <path>` and `git diff --cached -- <path>`.

2. Protect primary branches:
   - Treat `main`, `master`, `trunk`, `production`, `prod`, and the remote default branch as primary branches.
   - To detect the remote default branch, use `git remote show origin` when `origin` exists.
   - If the current branch is primary, stop before staging, committing, or pushing. Ask the user for explicit confirmation in Chinese, naming the exact branch and the planned action.
   - Continue only after the user clearly confirms.

3. Confirm commit scope:
   - If the worktree is clean, say there is nothing to commit or push.
   - If suspicious files appear, such as `.env`, private keys, credentials, logs, archives, build artifacts, or large generated outputs, stop and ask whether to exclude them.
   - Preserve user changes. Do not revert, discard, reformat, or clean unrelated files.

4. Group changes by function:
   - Build a short grouping plan from the actual diffs.
   - Prefer one commit per coherent feature, bugfix, documentation update, test update, or config change.
   - Include tests/docs with the feature they validate or describe when they are part of the same functional change.
   - If a file contains edits that span multiple groups and clean non-interactive splitting is risky, keep that file in the most relevant group and mention the compromise.
   - Avoid creating tiny commits that only exist because files are in different directories.

5. Verify before committing:
   - If recent test output in the current turn already verifies the exact final state, reuse it and state that.
   - Otherwise run the smallest existing checks that reasonably cover the changed areas.
   - If verification cannot be run, explain why before committing.

6. Create commits:
   - Stage only files for the current group.
   - Run `git diff --cached --stat` before each commit to verify the staged scope.
   - Write concise commit messages in the repo's existing style when visible in `git log --oneline -5`.
   - Use conventional prefixes only when the repository already uses them.
   - Repeat until all intended changes are committed.

7. Push:
   - Run `git status --short --branch` after committing.
   - Push to the current branch's upstream when it exists.
   - If no upstream exists, push with `git push -u origin <current-branch>` when `origin` exists.
   - If no suitable remote exists, stop and report the commits created but not pushed.

8. Final report:
   - List each commit hash and subject.
   - State the branch and push target.
   - State verification commands and results.
   - State whether the worktree is clean or what remains uncommitted.

## Safety Rules

- Never use `git reset --hard`, `git checkout --`, `git clean`, or force-push unless the user explicitly asks for that exact operation.
- Never amend, squash, rebase, or rewrite existing commits unless the user explicitly asks.
- Never commit secrets or environment files silently.
- Never skip the primary-branch confirmation, even when the user asked to commit and push all changes.
