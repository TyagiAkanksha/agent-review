# Worked example: AdvisorDesk

[AdvisorDesk](https://github.com/TyagiAkanksha/AdvisorDesk) is a production RAG app for a financial advisory firm's published guidance. It exposes 14 MCP tools to Claude over OAuth 2.1. This file shows how its design maps to the checks, so the skills can point to a real system instead of a toy.

## Tool inventory and annotations

AdvisorDesk declares hints through a `ToolHints(read_only, destructive)` object on each tool, serialized as `readOnlyHint` and `destructiveHint`.

| Tool | Class | readOnlyHint | destructiveHint |
| --- | --- | --- | --- |
| search_content | Read | true | false |
| count_content | Read | true | false |
| report_content_gaps | Read | true | false |
| report_weak_queries | Read | true | false |
| list_proposals | Read | true | false |
| create_draft | Reversible write | false | false |
| tag_content | Reversible write | false | false |
| propose_content_fix | Reversible write (draft only) | false | false |
| publish | Write, visible to clients | false | false |
| edit_content | Write, overwrites | false | true |
| archive | Write, removes chunks | false | true |
| accept_proposal | Write, gated by eval evidence | false | true |
| delete_content | Irreversible (tombstone, not restorable) | false | true |
| reject_proposal | Write, may archive a published draft | false | false |

Five tools are read-only and nine change data.

## What it does well (use as positive examples)

- **Authorization on every call.** The MCP route resolves the bearer token and re-checks the owner against the current `ADMIN_EMAILS` allowlist on every request, so removing an admin takes effect immediately. This satisfies RT-AUTH-1.
- **Evidence-gated acceptance.** `accept_proposal` refuses unless a baseline eval run and an after run both exist, coverage did not drop, and there are no regressions, judged by majority across runs. A model cannot talk its way past it.
- **Drafts first.** `propose_content_fix` creates a draft only; publishing is a separate, explicit step.
- **Eval gate.** An 80-question golden set gates releases, with majority voting over groups of 3 runs.

## The failure that became a check

AdvisorDesk was down for 12 days after the models it was pinned to were retired. Every test passed; nothing was watching the live app. The fix was a health probe that runs every six hours. This is the origin of PC-MOD-1 and PC-MON-1.

## Open questions a review should raise

These are the kind of findings the plugin should produce on AdvisorDesk. They are listed here as calibration targets, not as confirmed bugs.

- `reject_proposal` sets `destructiveHint: false`, but its description says a published draft is archived, which removes chunks. That is a destructive side effect (RT-ANN-2, Medium).
- `delete_content` is irreversible and callable by the agent directly. The client should confirm because `destructiveHint` is true, but the server has no confirmation or proposal step of its own (RT-APR-1; the reviewer should check whether the deployment relies on client confirmation and say so).
- `publish` sets `destructiveHint: false`, which is defensible because it is additive and `archive` reverses it (see `calibration.md`, borderline).
