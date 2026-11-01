# Module 05. Panel · core

## Purpose

A small local web page that prepares the work for the AI and the human, and carries out the actions the human clicks. It never makes a triage decision.

## Rules

- Bound to `127.0.0.1` only. No login is needed because nothing is exposed to the network.
- Starts in a couple of seconds: heavy work (the search index) runs in the background.
- Every write goes through the module that owns the file (P7). The panel has no private copy of any rule.
- Restarting the panel always loads the newest code.

## Pages

**Dashboard.** Counters (mails waiting, conversations, trash items), the last ingest run (with its report), buttons to run the ingest for received and sent mail, links to the other pages.

**Triage.**
- The mails in `Unsorted/`, oldest first, grouped by conversation: one header row per thread with its pieces underneath; one checkbox selects the whole conversation.
- A proposed project per row (from routing hints), editable.
- Badges: `SENT`, `↩ my reply inside`, `meeting`, `suggested: <project>`.
- **Copy prompt:** writes `state/triage_batch.json` (selected conversations, their pieces, pieces of the same threads that were not selected, related units, proposed projects) and copies a short prompt such as "Triage the batch in state/triage_batch.json following protocols/TRIAGE_PROTOCOL.md" to the clipboard.
- **Delete selected:** confirmation dialog, then the trash command per piece, then an undo toast.

**Search.** A search box with grouped results; each result opens the mail's `.md` rendered as a page.

**Projects (optional, recommended).** A table of projects with mail count, last activity, a 12-month sparkline and open deadlines. Clicking a project opens its brief. A visual map is a nice extra: for example a radial chart where distance from the centre is days since the last mail and colour is recent momentum.

## Visual design

Content first: the list or the chart dominates; the header and the counters stay thin. Colour has a job (status, momentum, overdue), and there is a full-screen mode for charts. Dark and light themes both work.

## Tests

Copy prompt writes a batch file that matches the selection, including unselected pieces of selected threads; delete selected calls the trash module and undo restores; the panel answers its status endpoint within a few seconds of starting. See scenarios S14, S15.
