# Finding format

Every skill in this plugin reports findings in this exact shape, so reports are comparable across runs and easy to act on.

## One finding

```markdown
### [RT-APR-1] High: delete_invoice runs with no approval step
Where: server.py, line 88 (tool `delete_invoice`)
Evidence: "def delete_invoice(invoice_id: str) -> str: del INVOICES[invoice_id]"
Why: an irreversible action the model can take on its own.
Source: MCP spec 2026-07-28, Tools, "User Interaction Model"
Fix: route the delete through a proposal a person accepts, or ask for confirmation with elicitation before deleting.
```

Rules for each field:

- **ID**: the check ID from `checklist.md`, so a reader can look up the rule.
- **Severity**: High, Medium or Low, from the rubric below. Never invent a fourth level.
- **Where**: file and line, or the tool name when the input is a `tools/list` response.
- **Evidence**: a short quote from the user's input. No quote, no finding.
- **Why**: one sentence on the consequence, not the rule.
- **Source**: the check's source from `checklist.md`.
- **Fix**: a concrete change: a rewritten name, description, annotation or code shape.

## Severity rubric

- **High**: an irreversible action with no approval step, authorization that is not checked on every call, a secret in code, an open prompt-injection path, or instructions hidden in a tool description.
- **Medium**: the model is likely to pick the wrong tool or act on wrong data: overlapping tools, a wrong annotation, unbounded responses, no monitoring.
- **Low**: clarity: a vague name, a missing description, an untyped parameter.

Annotation-only findings are capped at Medium. The MCP spec tells clients to treat annotations as untrusted unless the server is trusted, so a wrong hint is never the only thing standing between the model and damage.

## The report

```markdown
## <Skill name>: <what was reviewed>, <N> findings (<H> High, <M> Medium, <L> Low)

<one or two sentences: the most important thing to fix first>

<findings, High first, then Medium, then Low>

### Not checked
<what this skill did not look at, and which skill does>

### Assumptions
<framework detected, files read, anything inferred rather than seen>

### Could not parse
<inputs the skill could not read; omit the section if none>
```

## Suppressions

If the repo has `.agent-review.yml` at its root, skip findings it lists and say how many were skipped:

```yaml
ignore:
  - id: RT-ANN-2
    tool: publish
    reason: "Publishing is additive; archive reverses it."
```

Report: "Skipped 1 finding marked as intended in .agent-review.yml."
