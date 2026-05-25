---
name: read-docs
description: Read and inspect user-provided online document links from Feishu/Lark, DingTalk/AliDocs, and Tencent Docs on this Mac. Use when the user gives a feishu.cn, larkoffice.com, larksuite.com, alidocs.dingtalk.com, docs.qq.com, or related document/table/wiki link and asks Codex to read, summarize, analyze, extract, export, or answer questions from its contents. Prefer complete API/CLI/MCP reads where configured; use the locally logged-in Chrome session as the fallback for links that cannot be read completely through API tooling.
---

# Read Docs

## Overview

Use this skill to read online documents from the user's Chinese collaboration tools on this Mac. Route by domain, prefer the configured machine interface first, and explicitly label whether the result came from API/CLI/MCP or Chrome fallback.

Supported local setup:
- Feishu domestic: `lark-cli` configured with `brand: feishu`.
- DingTalk: `dws` installed and authenticated for one organization, but DingTalk document links can be cross-org; use Chrome by default for ad-hoc DingTalk document links unless the requested link belongs to the active `dws` organization.
- Tencent Docs: Codex MCP server `tencent-docs` configured at `https://docs.qq.com/openapi/mcp` with `Authorization: Bearer ...`.

## Routing

1. `feishu.cn`: use `lark-cli` first.
2. `larkoffice.com` or `larksuite.com`: try `lark-cli` once; if no authenticated `brand: lark` profile exists or it fails, use Chrome fallback.
3. `alidocs.dingtalk.com`: use Chrome fallback by default for document reading. Use `dws` only when the user asks to operate within the active DingTalk organization or when Chrome cannot provide enough content and the org matches.
4. `docs.qq.com`: use the `tencent-docs` MCP first. If current-session MCP tools are not loaded, call `https://docs.qq.com/openapi/mcp` via JSON-RPC with the configured token from `~/.codex/config.toml`. Use Chrome fallback only if MCP fails or lacks support for that document type.

## Feishu Workflow

Check status:

```bash
command -v lark-cli
lark-cli profile list
lark-cli auth status
```

For wiki links, resolve the underlying object:

```bash
lark-cli wiki +node-get --token '<url>'
```

Read by object type:
- `docx`: `lark-cli docs +fetch --api-version v2 --doc '<url-or-token>'`
- `sheet`: `lark-cli sheets +info --spreadsheet-token '<obj_token>'`, compute exact ranges from each sheet's `row_count`, `column_count`, and `sheet_id`, then `lark-cli sheets +read --spreadsheet-token '<obj_token>' --range '<sheet_id>!A1:T207' --value-render-option FormattedValue`
- `markdown`: `lark-cli markdown +fetch`
- other types: inspect `<domain> --help` and prefer the matching `lark-cli` read/export command

If `missing_scope` appears, start the smallest required device-flow authorization:

```bash
lark-cli auth login --scope '<missing scopes>' --json --no-wait
```

Show the returned verification URL exactly as a raw fenced text block. After the user confirms authorization, finish with:

```bash
lark-cli auth login --device-code '<device_code>'
```

## DingTalk Workflow

For ad-hoc `alidocs.dingtalk.com` links, use Chrome fallback first. The user chose this because `dws` is organization-scoped and cross-org document reads fail even when the browser can display the page.

If using `dws`, first check active org and auth:

```bash
dws auth status
dws doc info --node '<nodeId>' --format json
dws doc read --node '<nodeId>' --format json
```

If `dws` returns a cross-organization error, stop using CLI for that link and switch to Chrome fallback. Do not ask the user to repeatedly relogin unless they explicitly want the CLI bound to that target organization.

For deeper DingTalk CLI operations, load and follow the existing `dws` skill in `~/.codex/skills/dws/SKILL.md`.

## Tencent Docs Workflow

Prefer the configured `tencent-docs` MCP. In a new session, use the MCP tool if it appears. In the current session or when the tool is not exposed, call the remote MCP endpoint directly.

Direct JSON-RPC pattern:

```bash
TOKEN='<token from ~/.codex/config.toml Authorization header without Bearer prefix>'
curl -sS 'https://docs.qq.com/openapi/mcp' \
  -H 'Content-Type: application/json' \
  -H 'Accept: application/json, text/event-stream' \
  -H "Authorization: Bearer $TOKEN" \
  --data '{"jsonrpc":"2.0","id":1,"method":"tools/call","params":{"name":"get_content","arguments":{"file_id":"<file_id>"}}}'
```

Useful tools:
- `get_content`: read document body by `file_id`
- `sheet.get_sheet_info`: list worksheet IDs, names, row counts, and column counts for `docs.qq.com/sheet/...`
- `sheet.get_cell_data`: read worksheet cells; set `return_csv: true` when summarizing tables
- `smartsheet.list_tables`, `smartsheet.list_fields`, `smartsheet.list_records`: read smart sheets
- `search_space_file` and `query_space_node`: find documents when the user provides a name instead of a URL

For Tencent sheet URLs like `https://docs.qq.com/sheet/<file_id>?tab=<sheet_id>`, extract:
- `file_id` from the path segment after `/sheet/`
- `sheet_id` from the `tab` query parameter

Then call `sheet.get_sheet_info`, followed by `sheet.get_cell_data` over the exact row/column range.

## Chrome Fallback

Use Chrome when:
- API/CLI/MCP is unavailable, unauthorized, unsupported, or cross-org blocked.
- The user specifically asks to use the browser-visible content.
- The page exposes richer content than the API result.

Fallback sequence:

1. Open the exact URL in the user's Chrome profile.
2. Confirm the visible title/object matches the user link.
3. Inspect the accessibility tree and visible content.
4. For grids/tables, use select-all/copy, export/download controls, visible range reads, and scrolling/pagination as needed.
5. If the page is virtualized or paginated, state whether the result is complete or only the visible/copyable subset.
6. Label the source as `Chrome fallback`.

Do not use Chrome to bypass permissions. If Chrome shows no access, ask the user to grant access or log in with an account that has access.

## Verification

Before saying the content is fully read, report the source and coverage:
- Source: `lark-cli`, `dws`, `tencent-docs MCP`, or `Chrome fallback`
- Document title or sheet names
- Object type
- Page/sheet/row counts when available
- Any limitation such as missing scope, cross-org block, unsupported type, virtualized visible-only data, or export failure
