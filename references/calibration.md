# Calibration

Patterns that look wrong but are not. Read this before reporting, and drop any finding that matches. A false finding costs more trust than a missed one.

## Not findings

1. **A write tool with no `destructiveHint`.** The default is true, so the client already treats it as destructive. Only flag a hint that is set to the wrong value.

2. **A tool that only records a proposal.** If the tool writes a proposal or a draft and a separate step applies it, that tool is not the irreversible one. Example: AdvisorDesk's `propose_content_fix` creates a draft only; publishing is a separate tool. Review the tool that applies the change (here, `publish`) on its own merits.

3. **An irreversible tool behind a server-side precondition.** If the handler refuses unless a check passes (for example, AdvisorDesk's `accept_proposal` refuses unless eval runs before and after show the fix helped), report the precondition under Assumptions instead of raising RT-APR-1. Raise it only if a model alone can satisfy the precondition.

4. **Soft delete.** A delete that marks a row as deleted and keeps it restorable is a reversible write. Check the handler before raising RT-APR-1. If the code says the tombstone is not restorable, it is irreversible.

5. **Overlap that differs by scope the model can see.** `search_content` (metadata) and a separate vector search are not overlapping if both descriptions say which one to use when.

6. **Authorization in middleware, on every path.** If every tool call passes through one gate that re-checks the caller on every request (for example, resolving a bearer token and re-checking an allowlist), RT-AUTH-1 is satisfied for that path. List every way a caller can authenticate (bearer token, session cookie, API key, an internal agent loop) and check each one separately. A re-check on one path says nothing about the others: a cookie that is only checked at login can outlive the caller's removal by days.

7. **Read tools that return a lot by design, with a limit.** A report tool that returns up to 50 grouped items with a documented cap is bounded. RT-SIZE-1 is for results with no cap at all.

8. **`openWorldHint` left at its default on a closed tool.** The default is true, which is the cautious direction: a client may treat the tool as reaching outside. Like a missing `destructiveHint`, the omission is not a finding. Only flag `openWorldHint: false` on a tool that does reach outside (RT-ANN-3), and mention setting it to false on closed tools inside another finding's Fix if you like.

9. **Monitoring on code with no deployment.** PC-MON-1 is about a live app. If the repo has no deployment config (no Dockerfile, CI deploy job, hosting config or infrastructure code), there is nothing to monitor yet. List PC-MON-1 under Not checked with that reason instead of raising it.

## Borderline: report, but say why

- **`publish` with `destructiveHint: false`.** Publishing adds content, so false is defensible when another tool reverses it. Report as Low only if publishing replaces existing content in place.
- **A tool that triggers a destructive side effect.** If `reject_proposal` also archives a published draft, its effect includes a removal. `destructiveHint: false` is then wrong (RT-ANN-2), even though the tool's main purpose is a status change.
