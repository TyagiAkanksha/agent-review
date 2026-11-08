---
name: prod-check
description: >
  This skill should be used when the user asks "is my agent ready for production", "production
  readiness check", "what could break when I ship this agent", "check my MCP server before launch",
  or wants a review of model lifecycle, monitoring, secrets, prompt injection, logging and cost limits
  for an AI agent or MCP server. Returns a ranked risk list with fixes.
metadata:
  version: "0.1.0"
---

# Production check

Find the risks that make an agent fail after launch, not in the demo.

Before starting, read from the plugin root (two levels up from this file): `references/checklist.md` (the `PC-*` checks), `references/finding-format.md` and `references/calibration.md`. Read `references/advisordesk-example.md` for the outage that motivates PC-MOD-1.

## 1. Choose the mode

- **Repo mode**: the user gives a folder. Inspect the code and config directly.
- **Questionnaire mode**: no code available. Ask the questions in step 3 in one message, then review the answers. Mark every finding from answers as "Reported by user" under Assumptions.

## 2. Repo mode: where to look

| Check | Look in |
| --- | --- |
| PC-MOD-1 model lifecycle | Model IDs in code and config (strings like `claude-`, `gpt-`, `gemini-`, dated version suffixes); is there a fallback, an env var to switch, a check that fails loudly when a model is gone? |
| PC-MON-1 monitoring | CI workflows with `schedule:` / cron, uptime checks, health endpoints and whether anything calls them, alerting config |
| PC-SEC-1 secrets | Literal keys, tokens and passwords in code; committed `.env` files; config with credentials. Quote only the variable name and the first 4 characters, never the full secret |
| PC-INJ-1 injection | Where untrusted text (fetched pages, emails, uploaded documents, tool results) enters the prompt, and whether write tools are callable in the same turn |
| PC-LOG-1 logging | Whether tool calls are logged with tool name, arguments and caller |
| PC-COST-1 cost limits | Max steps or iterations in the agent loop, max tokens, retry caps and backoff; for an MCP server, rate limiting on tool calls |
| PC-MCP-1 third-party MCP | `.mcp.json`, client config, `npx`/`uvx` commands without a pinned version |

Skip `node_modules`, virtual environments and build output. A committed `.env.example` with placeholder values is not a finding.

## 3. Questionnaire mode: ask these

1. Which model IDs does the app call, and what happens if one is retired tomorrow?
2. What checks the live app on a schedule, and who gets alerted?
3. Where are API keys and passwords stored?
4. Does the agent read content from outside (web, email, user files)? Can it call write tools in the same turn?
5. Are tool calls logged with arguments and who made them?
6. Is there a cap on agent steps, tokens or retries?
7. Which third-party MCP servers does it use, and are their versions pinned?

## 4. Report

Follow `finding-format.md` exactly. Rank High first. For PC-MOD-1, tie the fix to the real failure: "AdvisorDesk was down 12 days when its pinned models were retired; a six-hourly probe now catches this." Use that story once per report, not per finding.

Not checked: always list tool design (covered by `/agent-review:review-tools`) and eval quality (not covered in v0.1).
