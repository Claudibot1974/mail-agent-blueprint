# Blueprint

This document describes what a Menagerie system is made of and how data flows through it. It deliberately says nothing about programming language or libraries: your AI agent chooses them with you (see [BUILD_PROMPT.md](BUILD_PROMPT.md)). The original system used Python with the standard library only, a small local web server for the panel, and plain JSON files for state. That combination works well, but it is a suggestion, not a requirement.

## 1. The flow

```
 mail client ──export .eml──▶ Inbox/ (drop folder)
                                   │
                         Otter, the Postmaster (ingest)
                                   │   - parse, extract attachment text
                                   │   - drop auto-replies
                                   │   - block duplicates (message-id index)
                                   ▼
                            Archive/Unsorted/<mail folder>
                                   │
                    Owl, the Sorter (triage, run by the AI agent)
                                   │   - reads every mail in full
                                   │   - proposes a destination
                                   │   - waits for the user's OK
                    ┌──────────────┴──────────────┐
                    ▼                             ▼
     Archive/Projects/<Project>/<mail>    Archive/Deleted/<mail> + tombstone
                    │
        Lion (search) · Ant (deadlines) · Wolf (recaps) · Raccoon (analyses)
```

The user's own sent mail follows the same path through a second drop folder (`Sent/`). A sent mail that answers a thread already filed in a project is proposed as a merge into that thread's folder, not as a new folder.

## 2. Folder layout

```
<root>/
  Inbox/                  drop folder: .eml files waiting for ingest
  Sent/                   drop folder: .eml of mails the user sent
  Archive/
    Unsorted/             ingested mails waiting for triage
    Projects/<Project>/   one folder per project, mail folders inside
    Deleted/              the trash (folders + tombstones)
    .threads_index.json   thread key → mail folder
    .message_ids_index.json  message-id → mail folder (duplicate shield)
  state/                  JSON state owned by the modules (deadlines, logs, registry…)
  protocols/              the run-time protocols the AI follows (copied from this repo, adapted)
  AGENTS.md / CLAUDE.md   the entry point your AI tool reads first
```

Names are suggestions. Pick names in the user's language if they prefer, but keep them ASCII and stable: every index stores paths.

## 3. The mail folder

One mail = one folder. The folder name sorts chronologically and is readable at a glance:

```
<YYYY-MM-DD>_<HHMM>_<sender-slug>_<subject-slug>_<6-char-hash>/
  <same name>.md          frontmatter + body + attachment text
  attachments/            the original attachment files
```

Example: `2026-03-02_0914_dreyes_quote-for-pump-replacement_a1b2c3/`

Slugs are lowercase ASCII, with non-Latin scripts transliterated. File names inside `attachments/` are shortened to a safe length (cloud-sync tools fail on long paths).

### The `.md` file

```markdown
---
thread_id: "<CA+abc123@mail.example.com>"
message_id: "<CA+def456@mail.example.com>"
subject: "Quote for pump replacement"
first_message_date: "2026-03-02"
last_message_date: "2026-03-02"
message_count: 1
direction: "received"          # received | sent | meeting
participants:
  - "Dana Reyes <dana.reyes@acme-pumps.example.com>"
  - "You <you@example.com>"
attachments:
  - "quote_AC-2291.pdf"
routing_hints: "Plant_Maintenance"   # optional, see 5.2
suggested_destination: ""            # sent mails only, see 4
---

# Quote for pump replacement

## Email 1 — 2026-03-02 09:14

**From:** Dana Reyes <dana.reyes@acme-pumps.example.com>
**To:** You <you@example.com>

### Body

Hello, please find attached our quote for the four replacement pumps…

### Attachments

#### quote_AC-2291.pdf

<extracted text of the PDF>
```

Rules for the extracted text:

- PDF, Word and plain text: full text, with a generous size cap.
- Spreadsheets: an **extract** only (for example at most 200 rows and 20,000 characters per sheet, 100,000 per file), ending with a clear marker such as `…[truncated — full file in attachments/]`. A whole workbook copied into Markdown makes files of tens of megabytes that nobody, human or AI, can read, and it slows search for everyone.
- Images: kept as files when they carry content (screenshots, photos); dropped when they are signature logos or icons (filter by pixel size and file size, not by name). The body keeps a marker such as `[image: attachments/screenshot_1.png]` where the image was.

## 4. Threads

A thread key is derived from the mail headers (`References`, `In-Reply-To`, `Message-ID`, with a normalized subject as a fallback). It goes into the `thread_id` frontmatter field.

- Each mail gets its own folder at ingest. The triage groups folders by `thread_id` and presents them as one unit (see the triage protocol).
- When a thread is filed, every piece goes into the same project folder.
- A **sent** reply to a thread whose folder is already in a project is ingested into `Unsorted/` with `suggested_destination: "Projects/<P>/<folder>"`. At triage, on `OK`, it is **merged**: its message is appended to the thread's `.md`, its attachments are moved into the thread's `attachments/`, both indexes are updated, and the temporary folder is removed.

## 5. Supporting data

### 5.1 The two indexes

| File | Key | Value | Used for |
|---|---|---|---|
| `.threads_index.json` | thread key | mail folder path | merging sent replies, grouping |
| `.message_ids_index.json` | message-id | `{ "folder": path, … }` | the duplicate shield: a known message-id is never ingested twice |

Both are owned by one small module (an "index store") that saves atomically and refuses to start a run if an index file is corrupt: it quarantines the bad file and tells the user to run a rebuild command, which reconstructs both indexes by scanning the archive.

### 5.2 Routing hints

The ingester can stamp `routing_hints` into the frontmatter: projects where earlier mails of the same sender or the same thread were filed. The triage reads hints first and calls the search module only when hints are missing or contradictory. Hints are suggestions; the decision still comes from reading the mail.

### 5.3 The project registry

`state/project_registry.json` maps each project folder to a theme (for example "Finance", "Suppliers", "Personal") and records frozen projects:

```json
{
  "version": 2,
  "themes": { "Plant_Maintenance": "Operations", "Tax_Returns": "Finance" },
  "frozen": { "Old_Office_Move": "2026-01-15" }
}
```

It is maintained by hand plus by the restructuring tool (rename, merge, freeze). At the start of every AI session, a short brief prints the active projects grouped by theme, so the agent never proposes a duplicate project.

### 5.4 Logs

- `state/filed_log.json`: append-only, one row per filing (project, mail folder, date, direction). It feeds the weekly recap.
- `state/autofile_log.json`: one row per automatic action under P1's exception, with what is needed to undo it.

## 6. Restructuring projects

Projects change: two turn out to be one, a name was wrong, a mail was filed in the wrong place. One tool owns all of this:

```
restructure --merge  <Src> <Target>   [--dry-run]
restructure --rename <Old> <New>      [--dry-run]
restructure --move-mail <mail path> <Target> [--dry-run]
restructure --freeze / --unfreeze <Project> [--dry-run]
restructure --relabel <Old> <New>     [--dry-run]   # folder already gone, fix state only
```

In one pass it moves folders, rewrites both indexes, every state file that stores a project name (deadlines, plans, logs, registry), and the project summary files, after taking a backup. A plain `mv` of a project folder is never allowed (P6). The session brief checks at every start that no state file points at a project that no longer exists, and prints the `--relabel` command if one does.

## 7. The panel

A small local web page (bound to `127.0.0.1` only) that shows:

- how many mails are waiting in `Unsorted/`, grouped by conversation, with a checkbox per conversation and a proposed project per row;
- a button that writes the selection to `state/triage_batch.json` and copies a ready-made prompt for the AI agent ("triage the batch");
- the trash with undo;
- search;
- optional module pages (deadlines, recaps, projects map).

The panel never makes a triage decision. It prepares the work for the AI and the human, and it executes actions the human clicks. See [modules/05_panel.md](modules/05_panel.md).

## 8. Where the AI fits

The AI agent is not a background service. It works when the user opens it and asks:

- "triage the batch" → it follows `protocols/TRIAGE_PROTOCOL.md`;
- "plan my week" → it follows the planner protocol (optional module);
- "what happened in project X last month?" → it uses the search module and reads the mails.

The run-time protocols are what makes the behaviour consistent. The entry file (`AGENTS.md`, `CLAUDE.md` or your tool's equivalent) is short and points at them. Keep it short: an agent reads it at the start of every session.
