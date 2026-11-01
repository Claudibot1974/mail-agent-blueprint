# Triage protocol (Owl, the Sorter)

This is a run-time protocol: the AI agent follows it every time the user asks for a triage. Copy it into your system and adapt the folder names, the language and the commands to your build. Keep the structure and the rules.

**Any AI tool working on this system follows this file exactly. Do not invent another format and do not change the order of sections.** The user must see the same presentation every time, from every tool.

---

## What triage is

Move each mail folder from `Archive/Unsorted/` into the right project folder under `Archive/Projects/`, or into the trash if it is of no interest. Each mail folder contains a `.md` file (frontmatter, body, extracted attachment text) and an `attachments/` folder with the originals.

Spreadsheet attachments appear in the `.md` only as an extract. When a decision depends on figures beyond the extract, open the original file.

## Which mails

1. Read `state/triage_batch.json` if it exists: it holds the mails the user selected in the panel, grouped by conversation (`threads`), each with a proposed project. Work on the ones still in `Unsorted/`.
2. If the file is missing or already done, take the oldest mails in `Unsorted/` (the folder names start with the date), at most 15, and say how many you took.
3. Never ask the user to list folders again.

### Conversations

- Mails with the same `thread_id` are **one unit**. A mail with an empty `thread_id` is its own unit.
- If other pieces of the same conversation are still in `Unsorted/` but were not selected, include them anyway and say so in the header: `(+2 pieces not selected in the panel)`. A conversation is never triaged by halves.
- Two units with the same subject but different `thread_id` (typically a forward) stay separate. Add a `Linked to:` line; propose the same project only if the content says so.
- A piece with `suggested_destination` means the conversation already lives in that project folder: that project is Option A for the whole unit.

## Hard rules

1. **Analyse each mail, present each conversation once, deliver everything in one answer.**
   - Analysis is per mail: open every `.md` of the unit and read it whole. Never skip a piece because a later reply quotes it.
   - Presentation is per conversation: a single mail gets a **Mail block**, a thread gets one **Thread block** with a timeline entry per mail.
   - Delivery is one answer covering every unit of the batch. Do not stop after each block to wait for an `OK`.
   - The batch never shortens the work: every mail gets its own block or timeline entry, built from a full read.
2. **Decide by content.** Read the full `.md`, body and attachment text, before proposing. The subject is a signal, not a reason.
3. **Reuse existing projects.** Check the list of projects (printed by the session brief) before proposing a new one. Check the frozen projects too: if a frozen project covers the topic, propose to unfreeze it instead of creating a duplicate.
   - Never create a project folder the user did not explicitly approve.
   - After the user approves a new project, add it to `state/project_registry.json` with a theme (propose one; the user confirms).
   - Never rename or merge project folders with a plain move: use the restructuring tool, dry run first.
4. **Propose aggregations.** If two or more different conversations in `Unsorted/` look related (same topic, same counterpart), add a `Related (aggregation)` line suggesting one shared project, so you don't create many single-mail projects. You may glance at folder names to spot candidates, but you classify each mail only by reading it.
5. **Oldest first.**
6. **Wait for the user's confirmation before moving anything.** Showing proposals together does not mean moving them together.
7. **Language:** write for the user in the language and level set in the entry file.
8. **Project folder names:** ASCII, `Underscore_Separated`, consistent with existing folders.
9. **Automatic replies** (out-of-office, bounces, read receipts) are trashed. The ingester should already drop them; this rule is a backstop. Ticket notifications from helpdesk systems are **not** automatic replies: they carry real content.

## Output format

Start with one line: `<N> mails = <K> conversations (<T> threads with 2+ mails, <S> single mails).`

Number the blocks per conversation: `Mail n of K` or `Thread n of K`.

### Mail block (single mail)

```
Mail n of K: <folder name>

## Recap
- **From / To**: <sender> / <recipients>
- **Date**: <date>
- **Subject**: <subject>
- **Body (key points)**:
  - <point>
  - <point>
- **Purpose / evaluation**: <who asks whom to do what, and why; your assessment of its value or status. Do not invent missing facts>
- **Attachments**: <real attachments only; "none" if only signature images>
- **Signals**: <people, companies, keywords that drive the classification>
- **Proposed deadline**: <only if the mail contains a due date: what + YYYY-MM-DD>

## Proposed destination
- **Option A (existing)**: `Projects/<Name>`: <why it fits>
- **Option B (new)**: `Projects/<New_Name>`: <why a new project is needed>
- **Related (aggregation)**: <only if other conversations look related>

## My choice
→ <single recommendation>

Confirm with `OK` / type a different folder / `skip` / `delete` / `stop`.
```

### Thread block (2+ mails of one conversation)

```
Thread n of K: "<subject>", <m> mails, <first date> → <last date>   [(+x pieces not selected in the panel)]

## Thread recap
- **People**: <in order of first appearance>
- **What it is about**: <2–4 points: topic and where it stands now>
- **Purpose / evaluation**: <the concrete request or decision, who owns it, your assessment>
- **Timeline** (oldest first, one entry per mail):
  1. <date time>, <sender>, `<folder>`
     - <what this mail adds; "only an acknowledgement" is a valid entry>
  2. …
- **Attachments**: <each with the timeline number it came in>
- **Signals**: <…>
- **Linked to**: <only for related units>
- **Proposed deadline**: <only if a due date appears anywhere in the thread>

## Proposed destination (one for the whole thread)
- **Option A (existing)**: …
- **Option B (new)**: …

## My choice
→ <single recommendation>

Confirm with `OK` (files all <m> mails) / type a different folder / `skip` / `delete` (all <m>) / `stop`.
```

A mail the user sent is marked `**SENT by you**` in its timeline entry, never dropped.

After all blocks, always add:

```
## Possible reminders
- Mail/Thread n: <short follow-up>, <date if the mail gives one, else "date to choose">

Tell me which reminders to add. I will add only the ones you confirm.
```

## After the user's `OK` (or a folder name)

For each mail of the unit, oldest first:

1. If the project folder is new, create it and register it with a theme (rule 3).
2. Move the mail folder with the system's **file command** (never a plain move), which updates both indexes in the same step. A sent piece with `suggested_destination`, when the user kept that option, is **merged** into the thread folder instead.
3. Log the filing in `state/filed_log.json` (through its owner module).
4. Confirm in one line: `Thread → Projects/<Name>/ (3 folders)`.

If the user typed a different folder than the suggested one, every piece goes there and nothing is merged: one conversation never ends up in two projects.

**`delete`:** run the trash command once per piece. Never remove files directly, never edit the indexes by hand for a delete. Confirm: `Trashed → Deleted/<folder> (undo: trash --restore "<folder>")`.

**`skip`:** leave it in `Unsorted/`. **`stop`:** end the session.

## After filing: deadlines (only if the deadline module is installed)

Scan each filed mail for due dates and action items. **Propose** them; add only what the user confirms. Never guess a date: vague timing goes into a text field with an empty date.

For a mail the user **sent**, check whether it completes an open deadline (the deadline module can pre-compute candidates). Ask: *"This sent mail may complete `<deadline>` (due `<date>`). Close it?"* Close it only on `OK`.

## Token discipline

- Don't re-read files you already read in this session.
- Don't re-list the projects for every mail; refresh the list only after creating a project.
- Query large state files through their command-line tools; never load them whole.
