# agent-review

A Claude plugin that reviews the design of your AI agent or MCP server before you ship it.

Point it at your tools or your repo. It reports what's likely to break in production, each finding with a quote from your code, why it matters, where the rule comes from, and a concrete fix.

> Status: v0.1, early. Feedback and issues welcome.

## What it checks

| Skill | What it reviews |
| --- | --- |
| `/agent-review:review-tools` | Tool annotations, approval steps for irreversible actions, per-call authorization, overlapping tools, naming, response size, and instructions hidden in tool descriptions |
| `/agent-review:prod-check` | Model lifecycle (what happens when your pinned model is retired), monitoring, secrets in code, prompt-injection paths, tool-call logging, cost and rate limits, third-party MCP servers |
| `/agent-review:full-review` | Both, on one repo, in one ranked report |

Every check is listed in [references/checklist.md](references/checklist.md) with its ID, severity and source.

## What it knows that a plain prompt doesn't

Anyone can paste their code into Claude and ask for a review. This plugin adds judgment a general model tends to get wrong:

- **Spec defaults.** `destructiveHint` defaults to true, so a write tool without it is not a bug. `readOnlyHint` defaults to false, so a read tool without it is.
- **Hints are not security.** The MCP spec says clients treat annotations as untrusted, so a wrong hint is never ranked High on its own.
- **A fixed severity rubric**, so the same issue gets the same severity every run.
- **Calibration from production.** [references/calibration.md](references/calibration.md) lists patterns that look wrong but aren't, learned from running [AdvisorDesk](https://github.com/TyagiAkanksha/AdvisorDesk).
- **A model-lifecycle check** that comes from a real 12-day outage when pinned models were retired.
- **An answer key.** The sample server ships with its planted issues listed, so you can check the plugin's results yourself.

## Install

In Claude Code:

```
/plugin marketplace add TyagiAkanksha/agent-review
/plugin install agent-review@agent-review
```

## Try it in a minute

The repo includes a small MCP server with problems planted on purpose:

```
/agent-review:review-tools examples/sample-mcp-server/server.py
```

Compare the result with [the answer key](examples/answer-keys/sample-mcp-server.md).

On your own project:

```
/agent-review:review-tools path/to/your/mcp/server
/agent-review:full-review path/to/your/repo
```

It accepts a `tools/list` JSON response, Python (MCP SDK `MCPServer` or `FastMCP`), TypeScript MCP SDK code, or a plain list of tools.

## Example finding

```markdown
### [RT-APR-1] High: delete_invoice runs with no approval step
Where: server.py (tool `delete_invoice`)
Evidence: "Permanently delete an invoice. This cannot be undone."
Why: an irreversible action the model can take on its own.
Source: MCP spec 2026-07-28, Tools, "User Interaction Model"
Fix: route the delete through a proposal a person accepts, or ask for confirmation with elicitation before deleting.
```

## Marking a finding as intended

Add `.agent-review.yml` at your repo root:

```yaml
ignore:
  - id: RT-ANN-2
    tool: publish
    reason: "Publishing is additive; archive reverses it."
```

## What it is not

- **Not a security scanner.** For vulnerability scanning of MCP servers, use tools built for it, such as mcp-scan, Snyk agent-scan, mcp-shield or the MCP Inspector. agent-review looks at design.
- **Not an eval generator.** It checks whether you have evals in a later version; it doesn't write them.
- **Not a summary of any book.** The checks are written from scratch and link to their sources.

## Sources and credit

The checks draw on these, each cited by the checks that use it:

- [Principles of Building AI Agents](https://mastra.ai/book) by Sam Bhagwat (Mastra), free to read. The overall framework: design tools first, a human in the loop for irreversible actions, tracing and evals.
- [MCP specification, Tools (2026-07-28)](https://modelcontextprotocol.io/specification/2026-07-28/server/tools) and [Tool annotations as risk vocabulary](https://blog.modelcontextprotocol.io/posts/2026-03-16-tool-annotations/).
- Anthropic: [Writing effective tools for AI agents](https://www.anthropic.com/engineering/writing-tools-for-agents), [Building effective AI agents](https://www.anthropic.com/engineering/building-effective-agents), [Demystifying evals for AI agents](https://www.anthropic.com/research/demystifying-evals-for-ai-agents).
- [OWASP Top 10 for LLM Applications 2025](https://genai.owasp.org/llm-top-10/) and [OWASP Top 10 for Agentic Applications](https://genai.owasp.org/2025/12/09/owasp-top-10-for-agentic-applications-the-benchmark-for-agentic-security-in-the-age-of-autonomous-ai/).
- Lessons from two production apps: [AdvisorDesk](https://github.com/TyagiAkanksha/AdvisorDesk) and SentinelBrief.

## Roadmap

- [x] v0.1: review-tools, prod-check, full-review, sample server with answer key
- [ ] Precision results on AdvisorDesk and five public MCP servers (GitHub, Google Drive, Notion, Supabase, Playwright), published here
- [ ] `/agent-review:eval-check`: does the repo have evals, a judge and a release gate?

## License

MIT. See [LICENSE](LICENSE).

Built by [Akanksha Tyagi](https://tyagiakanksha.com).
