---
name: full-review
description: >
  This skill should be used when the user asks for a "full review of my agent", "full review of my
  MCP server", "review everything before I ship", or "run all the agent-review checks". Runs the
  review-tools and prod-check reviews on one repo and combines them into a single ranked report.
metadata:
  version: "0.1.2"
---

# Full review

Run every check in this plugin on one repo and produce one report.

A skill cannot call another skill, so do the work directly:

1. Read `../review-tools/SKILL.md` and `../prod-check/SKILL.md` (relative to this file), plus the shared files in `references/` at the plugin root that they name.
2. Ask for the repo path if the user did not give one. Full review needs code; for a `tools/list` response alone, run the review-tools steps and say prod-check needs the repo or the questionnaire.
3. Run the review-tools steps (inventory, `RT-*` checks, calibration).
4. Run the prod-check steps in repo mode (`PC-*` checks, calibration).
5. Apply `.agent-review.yml` suppressions once, across both.
6. Merge into one report in the `finding-format.md` shape:
   - Title: `## Full review: <repo>, <N> findings (<H> High, <M> Medium, <L> Low)`
   - A short summary: the three findings to fix first, in order, one line each.
   - All findings, High first. Keep each finding's original ID so the user can see which check raised it.
   - One Not checked section (eval quality is not covered in v0.1), one Assumptions section, and Could not parse if needed.
   - An appendix with the tool inventory table.

Do not repeat the same issue under two IDs. If an irreversible tool also has a wrong annotation, report RT-APR-1 and mention the annotation in its Fix.
