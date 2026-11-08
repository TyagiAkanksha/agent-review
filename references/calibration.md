# Calibration

Patterns that look wrong but are not. Read this before reporting, and drop any finding that matches. A false finding costs more trust than a missed one.

## Not findings

1. **A write tool with no `destructiveHint`.** The default is true, so the client already treats it as destructive. Only flag a hint that is set to the wrong value.

2. **A tool that only records a proposal.** If the tool writes a proposal or a draft and a separate step applies it, that tool is not the irreversible one. Example: AdvisorDesk's `propose_content_fix` creates a draft only; publishing is a separate tool. Review the tool that applies the change (here, `publish`) on its own merits.

3. **An irreversible tool behind a server-side precondition.** If the handler refuses unless a check passes (for example, AdvisorDesk's `accept_proposal` refuses unless eval runs before and after show the fix helped), report the precondition under Assumptions instead of raising RT-APR-1. Raise it only if a model alone can satisfy the precondition.

4. **Soft delete.** A delete that marks a row as deleted and keeps it restorable is a reversible write. Check the handler before raising RT-APR-1. If the code says the tombstone is not restorable, it is irreversible.

5. **Overlap that differs by scope the model can see.** `search_content` (metadata) and a separate vector search are not overlapping if both descriptions say which one to use when.

6. **Authorization in middleware.** If every tool call passes through one gate that re-checks the caller (for example, a request handler that resolves the bearer token and re-checks an allowlist on every request), RT-AUTH-1 is satisfied even if individual handlers don't check.

7. **Read tools that return a lot by design, with a limit.** A report tool that returns up to 50 grouped items with a documented cap is bounded. RT-SIZE-1 is for results with no cap at all.

## Borderline: report, but say why

- **`publish` with `destructiveHint: false`.** Publishing adds content, so false is defensible when another tool reverses it. Report as Low only if publishing replaces existing content in place.
- **A tool that triggers a destructive side effect.** If `reject_proposal` also archives a published draft, its effect includes a removal. `destructiveHint: false` is then wrong (RT-ANN-2), even though the tool's main purpose is a status change.
