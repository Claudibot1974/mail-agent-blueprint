# Module 06. Deadlines and weekly plan (Ant, the Planner) · optional

## Purpose

Keep the commitments that arrive by mail in one place, plan the week around them, and notice when something slips, without ever adding or closing anything by itself.

## Data

`state/deadlines.json` is the master store, owned by one module:

```json
{
  "id": "3f8a21bc09de",
  "text": "Send purchase order for 4 pumps to Acme",
  "when": "by Tuesday",
  "due": "2026-03-10",
  "due_time": null,
  "priority": "high",
  "project": "Plant_Maintenance",
  "status": "open",
  "note": "",
  "source": "mail Dana Reyes 2026-03-04",
  "created": "2026-03-04T17:02:00",
  "closed": null
}
```

- `id` = first 12 hex characters of a SHA-1 of the normalized text: stable and duplicate-proof.
- `due` is a real ISO date or `null` (P11). Vague timing stays in `when`.
- `project` is the exact project folder name, or empty. Never a short form: the restructuring tool rewrites these on rename and merge.
- `status`: `open` or `done`. Closed items older than 30 days move to `state/deadlines_archive.json`.
- `state/weekly_plans.json` holds the weekly plans (items per day, linked to deadlines by id). Closing a plan item linked to a deadline closes the deadline.
- Human-readable mirrors (`Deadlines.md`, `Weekly_Plan.md`) are generated after every change and never edited (P8).

## Behaviour

- **Capture:** during triage, the AI proposes deadlines found in the mails it files; it adds only what the user confirms.
- **Closure proposals from sent mail:** when a sent mail is ingested, compare it with the open deadlines and write candidates to `state/deadline_suggestions.json`. Word overlap alone produces mostly noise (in the original system 717 of 722 word-match proposals were rejected), so keep the matcher strict, or let the AI judge each candidate, and always ask the user. Proposals for deadlines already closed are dropped.
- **Deviation signals:** a small engine flags `MISSING` (due, not planned), `LATE`, `AT_RISK` (due within 3 days), `OVERLOAD` (4+ items one day), `NO_DATE` (high priority, no date), `SLIPPED` (carried over).
- **Rituals** (a protocol the AI follows): daily check, plan the week, weekly review (which ends by proposing to freeze projects that have been quiet for more than 60 days with no open deadlines).
- **Query tool:** a command-line tool answering one-line queries (`--open`, `--overdue`, `--due 7`, `--project X`, `--search words`). The AI uses it instead of reading the JSON file whole: these files grow to hundreds of records.

## Tests

Adding the same text twice gives one record; a vague date stays `null`; renaming a project rewrites its deadlines; a closed deadline's suggestions disappear; the mirror is regenerated after every change.
