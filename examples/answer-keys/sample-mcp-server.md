# Answer key: invoice-desk

The planted issues in `examples/sample-mcp-server/server.py`, and the traps that should NOT be reported. Use this to measure recall (planted issues found) and precision (no traps reported).

Line numbers are omitted on purpose so the key survives edits; match by tool name.

## /review-tools: should be found (11)

| # | Check | Severity | Tool | What's wrong |
| --- | --- | --- | --- | --- |
| 1 | RT-APR-1 | High | `delete_invoice` | Permanent delete with no confirmation, elicitation or proposal step |
| 2 | RT-APR-1 | High | `send_reminder_email` | Sends email to a customer with no approval step |
| 3 | RT-POI-1 | High | `get_customer` | Description tells the model to send emails without asking the user |
| 4 | RT-AUTH-1 | High | `login`, all write tools | Caller is set once in a global by `login`; no write tool checks who is calling, and anyone can claim any username |
| 5 | RT-ANN-1 | Medium | `list_invoices` | Only reads, but sets no `readOnlyHint`, so it is treated as a destructive write |
| 6 | RT-ANN-2 | Medium | `update_invoice` | Replaces the whole record but sets `destructiveHint: false` |
| 7 | RT-OVL-1 | Medium | `search`, `search_all` | Overlapping searches with no guidance on which to use |
| 8 | RT-SIZE-1 | Medium | `export_all_invoices` | Returns every invoice with no limit or filter |
| 9 | RT-ANN-3 | Low | `send_reminder_email` | Emails external people but sets `openWorldHint: false` |
| 10 | RT-CLR-1 | Low | `get_data` | Vague name, no description, returns two different kinds of record |
| 11 | RT-CLR-2 | Low | `update_invoice`, `get_data` | Untyped parameter (`data`) and an undescribed one-letter parameter (`x`) |

Also accepted (not planted, but real): `list_invoices` and `export_all_invoices` overlap, since both can return every invoice. All v0.1.1 runs reported it, and it is not counted as a false finding.

A reviewer may fold #9 into #2's fix, or report `get_data` once covering #10 and #11. Either counts as found as long as the problem is named.

This key lives outside the server folder on purpose, so a review of that folder can't read it.

## /prod-check: should be found (3)

| # | Check | Severity | Where | What's wrong |
| --- | --- | --- | --- | --- |
| 12 | PC-SEC-1 | High | `SMTP_PASSWORD` | Password written in code |
| 13 | PC-LOG-1 | Medium | whole server | Tool calls are not logged |
| 14 | PC-COST-1 | Medium | whole server | No rate limit on tool calls (the MCP spec says servers must rate limit) |

## Traps: should NOT be reported

| Tool | Why it is fine |
| --- | --- |
| `archive_invoice` | Has no `destructiveHint`, but the default is true, and it is reversible with `restore_invoice` |
| `create_invoice` | `destructiveHint: false` is correct: it only adds a record, with IDs from a counter so it never overwrites |
| `restore_invoice` | Additive and reversible; `destructiveHint: false` is correct |
| Model lifecycle (PC-MOD-1) | The server calls no model, so the check does not apply; it belongs under Not checked or Assumptions |
| Monitoring (PC-MON-1) | A sample with no deployment has nothing to monitor; it belongs under Not checked |
| `openWorldHint` defaults on in-memory tools | The default (true) is the cautious direction, so leaving it unset is not a finding |

## Scoring

- **Recall** = planted issues found / 14. Target: 14 of 14 on each of 3 runs.
- **Precision** = real findings / all findings reported. Any trap reported counts against precision.

## Results

### v0.1.0, 3 independent runs (2026-11-07)

Each run used `full-review` on the sample folder, with the answer key removed from disk.

| Run | Planted issues found | Findings reported | Not real | High findings |
| --- | --- | --- | --- | --- |
| 1 | 14 / 14 | 16 | 1 (PC-MON-1) | 5, same set |
| 2 | 14 / 14 | 16 | 1 (PC-MON-1) | 5, same set |
| 3 | 14 / 14 | 17 | 2 (PC-MON-1, openWorldHint defaults) | 5, same set |

All three runs also flagged `create_invoice` for overwriting existing invoices. That was a real bug in the sample, not a planted one: IDs came from the record count, so after a delete a new invoice took an existing ID. The reviewers were right and this key was wrong. Fixed in v0.1.1 by issuing IDs from a counter.

The two false findings led to calibration items 8 and 9 in `references/calibration.md`.

### v0.1.1, 3 independent runs (2026-11-07)

Same setup, after fixing the sample and adding calibration items 8 and 9.

| Run | Planted issues found | Findings reported | Not real | High findings |
| --- | --- | --- | --- | --- |
| 1 | 14 / 14 | 14 | 0 | 5, same set |
| 2 | 14 / 14 | 14 | 0 | 5, same set |
| 3 | 14 / 14 | 14 | 0 | 5, same set |

Every run folded #9 (`openWorldHint`) into the fix for #2 and reported the accepted `list_invoices` / `export_all_invoices` overlap. Every run listed PC-MON-1 under Not checked because the sample has no deployment.
