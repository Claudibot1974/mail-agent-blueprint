# Module 07. Weekly recap (Wolf, the Chronicler) · optional

## Purpose

A short narrative of what happened in a project over a chosen period, written from the mails filed there, so you can catch up on a project in two minutes or brief someone else.

## Behaviour

- **Manual only.** The user picks one or more projects and a period (from/to dates) on a panel page and starts the run. Nothing runs on a schedule.
- **Input:** the mails filed in the project during the period (from `state/filed_log.json` and the project folder), plus its open deadlines.
- **Prompt package:** the module builds a ready prompt (the mails' recaps or full texts within a size budget, the period, the previous report's conclusions). The user can read and edit it before the run.
- **Three ways to run it:** copy the prompt into any AI tool; run it headless with the user's AI subscription or command-line tool; or call an AI API from a provider list kept in one place.
- **Output:** a structured JSON (summary, decisions, open points, people, risks), rendered into an HTML report in `reports/recaps/<Project>/`, one dated file per run, each linking to the previous one. An index page lists every report.
- **Cursor:** per project, the last period covered, so the next run can start where the last one ended.
- **Writing quality:** optionally run a tone check on the text (no filler, no hype, no invented facts).

## Tests

A period with no filed mails produces a "nothing happened" report, not an error; two runs never overwrite each other; the cursor advances only after a successful render.
