---
name: review-tools
description: >
  This skill should be used when the user asks to "review my MCP tools", "review my MCP server",
  "check my agent's tools", "are my tool annotations right", "audit tool design", or shares a
  tools/list response, FastMCP or TypeScript MCP server code and wants feedback before shipping.
  Reviews tool annotations, approval steps for irreversible actions, per-call authorization,
  overlap, naming, response size and tool poisoning, and returns findings with fixes.
metadata:
  version: "0.1.1"
---

# Review tools

Review the tools an agent or MCP server exposes and report findings with fixes.

Before starting, read these files from the plugin root (two levels up from this file):

- `references/checklist.md`: every check, its ID, default severity, source, and the annotation defaults.
- `references/finding-format.md`: the exact report and finding format, severity rubric and suppressions.
- `references/calibration.md`: patterns that are NOT findings. Apply it before reporting.
- `references/advisordesk-example.md`: a real 14-tool server to cite as an example when it helps.

## 1. Get the input

Accept any of these:

| Input | How to read it |
| --- | --- |
| `tools/list` JSON (an array of tools, or `{ "tools": [...] }`) | Read `name`, `description`, `inputSchema`, `annotations` directly |
| Python MCP SDK (v2 `MCPServer`, v1 `FastMCP`) or standalone FastMCP | Find `@mcp.tool(...)` / `@server.tool(...)` decorators, docstrings, type hints and the `annotations=` argument; read each handler body |
| TypeScript MCP SDK | Find `server.tool(...)`, `server.registerTool(...)` calls, their schemas and annotation objects; read each handler |
| A custom registry (tool specs built in code) | Find where specs are built and where annotations are serialized; follow the hints back to each tool |
| Markdown or a plain list | Use names and descriptions only; say under Assumptions that annotations and handlers could not be checked |

If the user gives a folder, search it for the patterns above. Skip `node_modules`, virtual environments, build output and tests. If nothing matches, ask for the file or a `tools/list` response instead of guessing.

Read handler bodies when available. A description can say "archive" while the code hard-deletes; the code wins.

For more than 20 tools, group them by verb (get/list/search, create, update/edit, delete/archive, send/publish, other) and review one group at a time so no tool is skipped. State the total count reviewed.

## 2. Build the inventory

For each tool, record: name, one-line purpose, class (Read, Reversible write, Irreversible from the classification table in `checklist.md`), the hints as declared, and the hints as effective (apply the defaults: `readOnlyHint` false, `destructiveHint` true, `idempotentHint` false, `openWorldHint` true).

Keep the inventory for the report's appendix when there are more than 5 tools.

## 3. Run the checks

Run every `RT-*` check in `checklist.md` against every tool:

1. **Annotations (RT-ANN-1/2/3).** Compare effective hints to what the tool actually does. Never flag a missing `destructiveHint` on a write tool; the default already says destructive. Cap annotation-only findings at Medium.
2. **Approvals (RT-APR-1).** For each Irreversible tool, look for a confirmation, an elicitation request, a proposal step, or a server-side precondition a model cannot satisfy alone. None found: High.
3. **Authorization (RT-AUTH-1).** Look for where the caller is identified and whether that check runs on every call (middleware, a per-request dependency) or only once. Look for write tools with no permission check at all.
4. **Overlap (RT-OVL-1).** Compare descriptions pairwise within each verb group. Flag pairs a model could not choose between from the descriptions alone.
5. **Clarity (RT-CLR-1/2).** Vague names, missing descriptions, untyped or undescribed parameters.
6. **Response size (RT-SIZE-1).** Tools that can return everything with no limit, pagination or filter.
7. **Tool poisoning (RT-POI-1).** Descriptions that instruct the model ("always call…", "do not tell the user…", "ignore…") instead of describing the tool.

Then apply `calibration.md` and drop or downgrade anything it covers. Then apply `.agent-review.yml` suppressions if the file exists.

## 4. Report

Write the report exactly as `finding-format.md` specifies:

- Title line with counts by severity, then one or two sentences on what to fix first.
- Findings ordered High, Medium, Low. Every finding has an ID, Where, Evidence (a quote), Why, Source and Fix.
- A Fix is concrete: the rewritten name, the description text, the annotation object, or the code shape (for example, "return a proposal ID instead of deleting; add `apply_deletion` that a person runs").
- End with Not checked (always include: evals, monitoring, secrets and model lifecycle are covered by `/agent-review:prod-check`), Assumptions, and Could not parse if anything was skipped.
- When there are no findings, say so plainly and list what was checked. Do not invent findings to fill the report.

Do not edit the user's code unless they ask. If they ask, apply one finding at a time and show the diff.
