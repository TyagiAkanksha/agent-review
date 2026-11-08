# Checklist

Every check this plugin runs, with its ID, default severity and source. Skills cite these IDs in findings.

Spec revision pinned: **MCP 2026-07-28**. Review this file when a new spec revision ships.

## Tool annotation defaults (MCP 2026-07-28, `ToolAnnotations`)

| Hint | Default when omitted | Meaning |
| --- | --- | --- |
| `readOnlyHint` | false | true = the tool does not modify its environment |
| `destructiveHint` | true | true = may perform destructive updates; false = only additive updates. Meaningful only when `readOnlyHint` is false |
| `idempotentHint` | false | true = repeating a call with the same arguments has no additional effect. Meaningful only when `readOnlyHint` is false |
| `openWorldHint` | true | true = may interact with external entities (web, email, third-party APIs) |

Consequences the skills must apply:

- A write tool with **no** `destructiveHint` is already treated as destructive. Do not flag the omission.
- A read-only tool with no `readOnlyHint` is treated as a write tool, so clients may ask for confirmation on every read. Flag it.
- Clients must treat annotations as untrusted unless the server is trusted. A correct hint is not a safety control.

## /review-tools checks

| ID | Check | Default severity | Source |
| --- | --- | --- | --- |
| RT-ANN-1 | Tool only reads data but does not set `readOnlyHint: true` | Medium | MCP spec, ToolAnnotations |
| RT-ANN-2 | Tool deletes, overwrites or removes data but sets `destructiveHint: false` | Medium | MCP spec, ToolAnnotations |
| RT-ANN-3 | `idempotentHint` or `openWorldHint` contradicts what the code or description says (for example, sends email with `openWorldHint: false`) | Low | MCP spec, ToolAnnotations |
| RT-APR-1 | Irreversible action (delete, publish to outside users, send, pay, transfer) runs with no confirmation, elicitation or proposal step | High | MCP spec, Tools, "User Interaction Model"; *Principles of Building AI Agents*, human-in-the-loop |
| RT-AUTH-1 | On any authentication path (bearer, cookie, API key, internal agent), authorization is checked once (at login or startup) instead of on every call, or a write tool does no permission check | High | OWASP Top 10 for Agentic Applications, identity and privilege abuse; MCP spec, Tools, "Security Considerations" |
| RT-OVL-1 | Two or more tools do nearly the same thing, so the model will pick between them inconsistently | Medium | Anthropic, "Writing effective tools for AI agents" |
| RT-CLR-1 | Vague tool name (`get_data`, `process`, `handle`) or missing description | Low | Anthropic, "Writing effective tools for AI agents" |
| RT-CLR-2 | Parameter has no type, or no description when its meaning isn't obvious from the name | Low | Anthropic, "Writing effective tools for AI agents" |
| RT-SIZE-1 | Tool can return an unbounded result (no limit, pagination or filter) | Medium | Anthropic, "Writing effective tools for AI agents"; OWASP LLM10 Unbounded Consumption |
| RT-POI-1 | Description contains instructions to the model ("always call X after this", "ignore previous") rather than a description of the tool | High | OWASP LLM01 Prompt Injection; MCP tool annotations post |

## /prod-check checks

| ID | Check | Default severity | Source |
| --- | --- | --- | --- |
| PC-MOD-1 | Model versions are pinned with no plan for retirement: no probe that would notice, no fallback model | High | AdvisorDesk 12-day outage (see `advisordesk-example.md`) |
| PC-MON-1 | No scheduled health check or alert on the live app | Medium | *Principles of Building AI Agents*, tracing and evals |
| PC-SEC-1 | Secret, password or API key written in code or committed config | High | OWASP LLM02 Sensitive Information Disclosure |
| PC-INJ-1 | Untrusted content (web pages, emails, user documents, tool results) reaches the model with no boundary, and the model can call write tools in the same turn | High | OWASP LLM01 Prompt Injection |
| PC-LOG-1 | Tool calls are not logged with arguments and caller | Medium | MCP spec, Tools, "Security Considerations" |
| PC-COST-1 | No limit on agent loop steps, tokens or retries; for an MCP server, no rate limit on tool calls | Medium | OWASP LLM10 Unbounded Consumption; MCP spec, Tools, "Security Considerations" (servers must rate limit) |
| PC-MCP-1 | Third-party MCP servers are used without a pinned version or review | Medium | *Principles of Building AI Agents*, MCP chapter; OWASP LLM03 Supply Chain |

## Classifying actions (used by RT-APR-1 and RT-ANN-2)

| Class | Examples | Needs approval? |
| --- | --- | --- |
| Read | search, list, get, count, report | No |
| Reversible write | create draft, tag, archive (restorable), edit with history | No, but should be logged |
| Irreversible | hard delete, publish to outside users, send email or message, payment, transfer, deploy to production | Yes |

When the code is not available and only a `tools/list` response is, classify from the name and description, and say so under Assumptions.
