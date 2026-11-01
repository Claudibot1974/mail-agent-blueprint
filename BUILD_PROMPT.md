# Build prompt

Copy everything below the line into your AI coding agent, opened in the empty folder where your system will live. Replace `<path or URL of this repository>` first.

---

You are going to build a personal email system following the Menagerie blueprint at `<path or URL of this repository>`.

## Step 0. Read before anything else

Read these files completely, in this order:

1. `PRINCIPLES.md`: the rules you must never break.
2. `BLUEPRINT.md`: the architecture and data formats.
3. `protocols/TRIAGE_PROTOCOL.md`: how triage works at run time.
4. The core module sheets: `modules/01_ingest.md` to `modules/05_panel.md`.
5. `acceptance/SCENARIOS.md`: the tests your build must pass.

Do not write any code yet.

## Step 1. Interview me

Ask me the following questions, a few at a time, and wait for my answers. Suggest a sensible default for each one. Then summarise my choices in a short table and ask me to confirm it.

**Environment**
1. Which operating system will this run on?
2. Which programming language do you prefer? If I have no preference, propose one that is easy to run on my machine without extra installs.
3. Where should the system live (folder path)? Warn me if I pick a cloud-synced folder: the archive can live there, but any git repository and cache should not.

**Mail**
4. Which mail client or provider do I use? How will mails reach the system: exported `.eml` files dropped into a folder (the blueprint's default), IMAP, the Gmail API, Microsoft Graph, or something else?
5. Do I want my own sent mails in the archive too?
6. Which languages do my mails come in? (This affects search: accents, transliteration of non-Latin scripts.)

**Organisation**
7. Give me 5–10 project names to start with, and optional themes to group them. If I don't know yet, start empty: projects get created during triage, with my approval.
8. Are there senders whose mails I always want trashed (newsletters, promotions) or always filed in one place? List exact addresses only. Start with none if unsure.

**Scope**
9. Core only (ingest, indexes, triage, trash, search, panel), or also optional modules (deadlines, weekly recap, meetings, analyses register)? Recommend core only for the first build.
10. Which AI tools will I use to run triage day to day? (Decides the name of the entry file: `CLAUDE.md`, `AGENTS.md`, or another.)
11. In which language and at what level should the AI write recaps and proposals for me?

## Step 2. Plan

Write a short build plan: components, files, the order you will build them in, and how each one maps to a module sheet. Show it to me and wait for my approval.

## Step 3. Build the core, in this order

1. **Index store** (`modules/02_indexes.md`): atomic saves, corruption quarantine, rebuild command. Tests first.
2. **Ingest** (`modules/01_ingest.md`): parse, mail folder, attachment extraction with caps, auto-reply filter, duplicate shield, threads. Tests with the sample mails in `acceptance/sample_mails/`.
3. **Trash** (`modules/03_trash.md`): delete, restore, list, purge, all updating both indexes. Tests.
4. **Restructuring tool** (`BLUEPRINT.md` §6) with `--dry-run` everywhere. Tests on a throwaway archive.
5. **Search** (`modules/04_search.md`): an index that loads fast and refreshes incrementally.
6. **Session brief**: prints the active projects by theme, the pending triage batch, and warnings about state that points at missing projects. Wire it to run at the start of each AI session if my tool supports that.
7. **Panel** (`modules/05_panel.md`): local only, `127.0.0.1`.
8. **Run-time files**: copy `protocols/TRIAGE_PROTOCOL.md` into the system, adapted to my folder names, language and tools. Write the entry file (`CLAUDE.md` / `AGENTS.md`): short, pointing at the protocols, stating the principles in one line each.

Rules while building:

- Every test runs on a temporary copy, never on my real archive.
- Never process my real mail until Step 4 is complete.
- When a module sheet and my answers disagree, ask me; don't guess.
- Keep every state file with a single owner (P7) and every write atomic.

## Step 4. Prove it

Run every scenario in `acceptance/SCENARIOS.md` that applies to the modules you built, using the sample mails. Report each one as PASS or FAIL with one line of evidence. Fix and re-run until all of them pass. Then show me how to do my first real ingest and my first triage, on a handful of mails.

## Step 5. Later

Optional modules are added one at a time, each with the same cycle: read its sheet, ask me its questions, plan, build with tests, prove with its scenarios.
