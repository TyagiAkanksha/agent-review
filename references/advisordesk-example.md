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

- **Bearer authorization on every call.** The MCP route resolves the bearer token and re-checks the owner against the current `ADMIN_EMAILS` allowlist on every request. (The cookie path did not, at first: see below.)
- **Evidence-gated acceptance.** `accept_proposal` refuses unless a baseline eval run and an after run both exist, coverage did not drop, and there are no regressions, judged by majority across runs. A model cannot talk its way past it.
- **Drafts first.** `propose_content_fix` creates a draft only; publishing is a separate, explicit step.
- **Eval gate.** An 80-question golden set gates releases, with majority voting over groups of 3 runs.

## The failure that became a check

AdvisorDesk was down for 12 days after the models it was pinned to were retired. Every test passed; nothing was watching the live app. The fix was a health probe that runs every six hours. This is the origin of PC-MOD-1 and PC-MON-1.

## Findings from the first review

agent-review was run on AdvisorDesk three times (v0.1.1). Every finding was checked against the code by hand. These two were the most serious, and both are now fixed with tests:

1. **Removing an admin didn't end their cookie session (RT-AUTH-1).** The bearer path re-checked `ADMIN_EMAILS` on every call, but the session-cookie path (admin app, in-app agent, MCP over cookie) only checked it at login, and cookies last 30 days. Fix: a resolver that re-checks the live allowlist on every cookie request. Lesson for reviewers: check *every* login path. A re-check on one path says nothing about the others (`calibration.md`, item 6).
2. **Client text could steer the admin agent (PC-INJ-1).** Questions from the public, unauthenticated chat came back verbatim from `report_weak_queries` and `report_content_gaps`, into the same turn where the agent could call `delete_content` and `publish`. Fix: once a turn reads client text, the server refuses content-changing tools for the rest of that turn; questions are capped and labelled as data in the tool descriptions.

Other real findings, smaller: `reject_proposal` and `tag_content` declared `destructiveHint: false` while removing data (RT-ANN-2); `delete_content` and `publish` had no server-side approval step (RT-APR-1); no rate limit on the MCP and agent endpoints (PC-COST-1); tool calls were not logged with caller and arguments (PC-LOG-1).

Consistency across the three runs: two found all four High findings; one missed the cookie-path issue, which led to the calibration change above.
