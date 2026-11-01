# Module 01. Ingest (Otter, the Postmaster) · core

## Purpose

Turn raw mail files into mail folders in `Archive/Unsorted/`, once each, with everything the triage needs and nothing it doesn't.

## Inputs and outputs

| In | Out |
|---|---|
| `.eml` files in `Inbox/` (received) and `Sent/` (sent by the user) | one mail folder per mail in `Archive/Unsorted/` (format: BLUEPRINT §3) |
| both indexes | both indexes updated |
| | processed `.eml` moved to `Inbox/_done/` (or deleted, if the user prefers) |
| | a run report: ingested, duplicates skipped, auto-replies dropped, errors |

The ingester runs when the user starts it (a launcher or a panel button). It is not a background service unless the user explicitly asks for one.

## Behaviour

1. **Parse** headers and body. Prefer the plain-text part; convert HTML to readable text when that is all there is. Decode encoded headers and every charset you meet.
2. **Duplicate shield:** if the `Message-ID` is already in the message-id index and its folder still exists, skip the mail and count it as a duplicate. If the entry points at a folder that no longer exists, the index is stale: treat this as an error to report, not as permission to re-ingest silently (see scenario S05).
3. **Auto-replies:** drop out-of-office replies, delivery failures and read receipts. Identify them with the `Auto-Submitted: auto-replied` header, delivery-status content types and well-known subject patterns. Do **not** treat `Auto-Submitted: auto-generated` or `X-Auto-Response-Suppress` as auto-replies: helpdesk tickets and system notifications use them and carry real content.
4. **Thread key:** compute it from `References` / `In-Reply-To` / `Message-ID`, falling back to a normalized subject (strip `Re:`, `Fwd:` and their local-language equivalents).
5. **Folder name:** `<date>_<time>_<sender-slug>_<subject-slug>_<hash>` (BLUEPRINT §3). For sent mail, use `sent-to-<first-recipient>` as the sender slug.
6. **Attachments:** save the originals in `attachments/` with shortened safe names; extract text into the `.md` with caps (BLUEPRINT §3). Keep content images, drop signature icons.
7. **Routing hints:** look up earlier folders of the same thread and the same sender in the indexes; stamp up to three project names in `routing_hints`, tagging frozen projects as `(FROZEN)`.
8. **Sent replies to filed threads:** if the thread index points into `Projects/`, stamp `suggested_destination` with that folder (BLUEPRINT §4). If it points to a folder still in `Unsorted/`, append the sent message to that folder and stamp `contains_sent: "<date>"` so the panel can show it.
9. **Update both indexes** through the index store, atomically, at the end of each mail.

## Automatic filing rules (optional, P1 exception)

A configuration file (for example `autofile_rules.yaml`) may list:

- `file:` exact sender addresses → project (for example a press-review service → `Press`);
- `trash:` exact sender addresses → trash (newsletters, promotions).

Rules match **exact addresses**, never words such as "info" or "news". Replies and forwards from those senders are held for normal triage. Every action is logged with what is needed to undo it, and an `--undo <folder>` command reverses it. The system never adds a rule by itself.

## Must not

- Move anything into `Projects/` (except under explicit automatic filing rules).
- Copy whole spreadsheets into the `.md`.
- Re-ingest a mail whose message-id is known.

## Tests

On a temporary archive with the sample mails: normal mail, three-mail thread, auto-reply dropped, helpdesk notification kept, duplicate skipped, huge spreadsheet capped, sent reply to a filed thread gets `suggested_destination`. See scenarios S01–S07.
