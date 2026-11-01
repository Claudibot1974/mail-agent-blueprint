# Acceptance scenarios

An implementation is ready for real mail only when every scenario for the modules it includes passes. Run them on a **temporary copy** of the system with the sample mails in [sample_mails/](sample_mails/), never on a real archive.

Report each scenario as `PASS` or `FAIL` with one line of evidence (a path, a count, an index entry).

All people, companies and addresses in the sample mails are invented.

## The sample mails

| File | What it is | Used in |
|---|---|---|
| `01_quote.eml` | Supplier quote with a CSV attachment; first mail of a thread | S01, S03, S05 |
| `02_quote_reply_SENT.eml` | Your reply to 01 (goes in `Sent/`) | S03, S07 |
| `03_quote_confirm.eml` | Supplier's answer, completes the thread, contains a condition with a date | S03, S16 |
| `04_out_of_office.eml` | Out-of-office (`Auto-Submitted: auto-replied`) | S02 |
| `05_helpdesk_ticket.eml` | Ticket notification (`Auto-Submitted: auto-generated`): real content | S02 |
| `06_newsletter.eml` | Promotion | S08 |
| `07_duplicate_of_01.eml` | Same Message-ID as 01 | S05 |
| `08_accountant_deadline.eml` | Single mail with a clear due date | S16 |
| `09_greek_meeting.eml` | Greek subject and body (Συνάντηση για το νέο έργο, *Meeting about the new project*) | S04, S13 |

## Core: ingest (module 01)

**S01. One mail, one folder.** Ingest `01_quote.eml`. → One folder in `Unsorted/` named `2026-03-02_0914_<sender>_quote-for-pump-replacement_<hash>`; the `.md` has the frontmatter fields of BLUEPRINT §3; the body text is present; `attachments/quote_AC-2291.csv` exists and its rows appear in the `.md` under the attachment heading; the original `.eml` has left `Inbox/`.

**S02. Auto-replies dropped, notifications kept.** Ingest `04` and `05`. → `04` produces no folder and is counted as an auto-reply in the run report. `05` produces a normal folder with the ticket text.

**S03. Threads.** Ingest `01` and `03` from `Inbox/` and `02` from `Sent/`. → All three folders (or two folders, if your build appends a sent reply to a thread still in `Unsorted/`, which is allowed: then the folder carries `contains_sent`) share the same `thread_id`; `02` has `direction: "sent"`.

**S04. Non-Latin text.** Ingest `09`. → The folder name is ASCII (transliterated or neutral slug); the `.md` keeps the Greek text intact in UTF-8.

**S05. Duplicate shield.** Ingest `01`, then `07`. → `07` is skipped as a duplicate; still exactly one folder. Then simulate a stale index: delete the folder of `01` by hand (outside the system) and ingest `07` again. → The run reports a stale index entry instead of silently re-creating the folder, or re-creates it **and** reports it. It never does so silently.

**S06. Spreadsheet cap.** Generate a workbook with 30 sheets × 1,000 rows × 50 columns and attach it to a test mail. → The `.md` holds at most the configured extract, ends with a truncation marker pointing to `attachments/`, and stays under ~150 KB; the original file is intact in `attachments/`.

**S07. Sent reply to a filed thread.** File the thread of `01`+`03` into `Projects/Plant_Maintenance`, then ingest `02` from `Sent/`. → `02` arrives in `Unsorted/` with `suggested_destination: "Projects/Plant_Maintenance/<thread folder>"`. Approving it merges it: the thread `.md` gains the message, the temporary folder is gone, both indexes point at the thread folder.

## Core: trash and indexes (modules 02, 03)

**S08. Delete and restore.** Ingest `06`; record both indexes; `trash --delete` its folder. → Folder in `Deleted/` with `.trash.json`; its thread key removed; its message-id repointed to `Deleted/`. `trash --restore` it. → Folder back in `Unsorted/`, both indexes identical to the recorded state.

**S09. Trash keeps the shield closed.** With `06` in the trash, ingest `06_newsletter.eml` again. → Skipped as a duplicate.

**S10. Purge preview.** `trash --purge --older-than 0 --dry-run`. → Lists `06`, changes nothing on disk or in the indexes.

**S11. Corrupt index.** Write `{not json` into the message-id index and start an ingest. → The run stops before touching any mail, the bad file is copied to `.corrupt_<timestamp>`, and the rebuild command is printed.

**S12. Rebuild.** Run the rebuild with `--dry-run`, then `--apply`. → The reconstructed indexes contain every mail folder in `Unsorted/` and `Projects/`; a second rebuild reports no differences.

## Core: search (module 04)

**S13. Search.** With all samples ingested: `search "pump quote"` finds the thread (words in any order); `search "synantisi"` and `search "συνάντηση"` both find `09` (transliteration, and accent-insensitive matching of `Συνάντηση`); `search "AP-400"` finds `01` through its attachment text. Restart the search process: the second start loads from cache in about a second. Add one mail: only that folder is read.

## Core: panel and triage (modules 05 + protocol)

**S14. Batch file.** In the panel, select the conversation of `03` only. → `state/triage_batch.json` lists the whole thread, with the unselected pieces flagged as not selected.

**S15. Triage presentation.** Ask the AI to triage the batch containing all samples still in `Unsorted/`. → One answer; it starts with the `<N> mails = <K> conversations …` line; the quote conversation is **one Thread block** with one timeline entry per mail and the sent piece marked `SENT by you`; single mails get Mail blocks; nothing is moved before an `OK`; `06` is proposed for deletion; `05` is proposed for filing, not deletion.

**S16. Deadlines proposed, not added.** During S15 → the AI proposes "send purchase order to Acme, 2026-03-10" (from `03`) and "documents to the accountant, 2026-03-20" (from `08`) as possible reminders; nothing is written to the deadline store until the user confirms. (Deadline module only.)

## Restructuring (BLUEPRINT §6)

**S17. Merge with preview.** Create projects `A` and `B` with one mail each. `restructure --merge A B --dry-run` → prints the plan, changes nothing. Without `--dry-run` → `A` is gone, both mails are in `B`, both indexes and every state file that named `A` now say `B`, and a backup exists.

**S18. Missing project warning.** Rename a project folder by hand, outside the tool, and start a session. → The session brief warns that state points at a missing project and prints the `--relabel` command.
