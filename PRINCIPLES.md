# Principles

These rules hold in every implementation of Menagerie, whatever the language, operating system or mail client. An implementation that breaks one of them is not a Menagerie implementation. Each rule has its reason next to it, because the reason tells you how to apply the rule in situations this document does not cover.

The acceptance scenarios in [acceptance/SCENARIOS.md](acceptance/SCENARIOS.md) test most of them.

## P1. The human approves every move

The AI reads and proposes. Only an explicit approval from the user can:

- move a mail out of the inbox folder (`Unsorted/`) into a project;
- create, rename, merge or freeze a project;
- add, change or close a deadline;
- send anything outside the machine.

A proposal shown together with others is still approved one by one, unless the user explicitly approves a group.

**Why:** an assistant that files by itself is fast until the day it files a contract in the wrong place, and nobody looks there again. The value of the system is that its archive can be trusted.

**Allowed exception:** narrowly scoped automatic filing rules that the user writes down explicitly (for example "newsletters from this exact address go to the trash"). Each rule names exact senders, is listed in one configuration file, logs every action and can be undone. The system never adds such a rule by itself.

## P2. Delete means trash

No module ever removes a mail for real. "Delete" moves the mail folder to `Deleted/` together with a tombstone file recording when, from where, and which index entries it had. Restoring puts everything back. Purging old trash is a manual command the user runs.

**Why:** deletion is the only operation you cannot fix afterwards. A tombstone costs a few kilobytes.

## P3. Read in full, decide by content

A destination is proposed only after reading the whole mail: body and extracted attachment text. The subject line and the sender are signals, never the basis of the decision. In an in-depth analysis, the original attachments are opened too, not just their text extracts.

**Why:** subjects lie ("RE: RE: FW: quick question" can be a contract change), and the decisive fact is often in an attachment or a screenshot.

## P4. One conversation, one unit

Mails that share a thread identifier are presented, decided and filed together. A conversation is never filed by halves, and never ends up split across two projects.

**Why:** a reply filed apart from its thread loses its context, and later searches find half a story.

## P5. Preview before bulk changes

Every command that changes many things at once (merging projects, moving a group of mails, purging the trash, rebuilding indexes) supports `--dry-run`. The dry run is shown to the user before the real run.

**Why:** a bulk mistake is a hundred small mistakes at once.

## P6. The indexes follow every move

The system keeps two indexes: thread → folder, and message-id → folder (the duplicate shield). Every operation that moves, merges or trashes a mail updates both, in the same step. A tool that moves mail folders without updating the indexes must not exist.

**Why:** a stale message-id entry makes the ingester believe a mail is new, and it archives it again as a duplicate. In the original system, 685 of 3,686 entries went stale before this rule existed.

## P7. One owner per state file

Every data file (indexes, deadlines, plans, logs) has exactly one module that writes it. Other modules read it, or ask the owner through a function or an API. Writes are atomic: write to a temporary file, then rename.

**Why:** two writers on one JSON file eventually produce a corrupt file, at the worst moment.

## P8. Never edit generated files

Human-readable mirrors (for example a `Deadlines.md` generated from `deadlines.json`) are rebuilt from the master data and never edited by hand, by the user or by the AI.

**Why:** an edit to a mirror is silently lost at the next rebuild.

## P9. Local first, nothing leaves by default

Mail, attachments, indexes and state live on the user's machine. Anything that leaves the machine (a summary pushed to a phone, a call to an AI API) is an explicit, previewed action, limited to an allow-list of fields.

**Why:** an email archive is the most sensitive dataset most people own.

## P10. Same format, every time

The triage output follows a fixed structure (see the triage protocol). Any AI tool working on the system uses the same structure.

**Why:** the user reads dozens of proposals a day. A stable format lets them scan instead of read, and spot a wrong proposal immediately.

## P11. Never guess a date

A deadline has a real calendar date or none at all. Vague timing ("end of the month", "ASAP") is stored as text next to an empty date and flagged as high priority if it is urgent.

**Why:** an invented date looks exactly like a real one, and someone will plan around it.

## P12. Guard the dangerous paths with tests

The ingest, the trash, the index updates and the project restructuring each have an automated test that runs on a throwaway copy, never on the real archive.

**Why:** these are the paths where a bug destroys data instead of just looking wrong.
