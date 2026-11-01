# Module 04. Search (Lion, the Librarian) · core

## Purpose

Find past mails by words, and summarise a project, fast enough to use in the middle of a triage.

## Behaviour

- **All words must match**, in any order, anywhere in subject, sender or text.
- **Accent-insensitive** (Unicode decomposition, drop combining marks) and **case-insensitive**.
- **Transliteration** for non-Latin scripts the user receives (for example Greek ↔ Latin): a Latin query finds the original script and the reverse.
- **Ranking:** subject matches score higher than body matches; recent mails get a boost.
- **Grouping:** results grouped by conversation (reply prefixes stripped from the subject).
- **Brief:** `--brief <Project>` prints mail count, threads, first and last date, days since the last mail, top senders, recent mails, open deadlines (if the deadline module exists) and the project description.
- **Command line and API:** the same functions serve the AI (`search "words" --limit 10`) and the panel.

## Performance rules (learned the hard way)

- **Cap the indexed text per mail.** Index the body in full and a fixed slice of each attachment (for example 15,000 characters each, 40,000 in total). Uncapped, a few spreadsheet dumps made the original index 155 MB and its load took a minute.
- **Store each text once.** Keep a normalized copy for matching; keep the transliterated copy only when it differs.
- **Incremental refresh.** Keep a cache with a per-folder stamp (folder path + modification times). At start, re-read only new or changed folders and drop vanished ones. A full re-read only on demand.
- **Fast cache format.** Use a format that loads in about a second for thousands of mails. Keep the cache outside any cloud-synced folder: it is derived data.
- **Build in the background** when the panel starts, so pages are usable at once; the search API reports `index_ready: false` until it is done.

Search is for **finding** mails. It is never a substitute for reading the mail and its original attachments.

## Tests

Words in any order; accented and unaccented forms; transliteration in both directions; a capped attachment still matches on its first part; incremental refresh picks up an added, a changed and a removed folder without re-reading the others; an old cache format is ignored. See scenario S13.
