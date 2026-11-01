# Module 03. Trash · core

## Purpose

Make "delete" reversible (P2). One module owns the trash; it is the only lawful way to delete a mail, from the AI and from the panel.

## Commands

```
trash --delete "<folder>"         # Unsorted/<folder> → Deleted/<folder> + tombstone
trash --restore "<folder>"        # back to Unsorted/, indexes restored
trash --list                      # what is in the trash, newest first
trash --purge --older-than 30 [--dry-run]   # really delete old items; manual only
```

## The tombstone

`Deleted/<folder>/.trash.json`:

```json
{
  "deleted_at": "2026-03-05T10:12:00",
  "from": "Unsorted/2026-03-05_0801_promo_spring-sale_9f8e7d",
  "by": "panel",
  "thread_keys": ["<abc@mail.example.com>"],
  "message_ids": ["<def@mail.example.com>"]
}
```

## Rules

- Delete: move the folder, write the tombstone, remove thread keys, repoint message-ids (module 02 rules). Repoint any state file that stores the folder path (for example logs), with a backup.
- Restore: refuse if a live folder with the same name exists; otherwise move back, put the thread keys back, repoint message-ids to `Unsorted/`.
- Purge: removes the folders **and** their index entries. Never scheduled; the user runs it.
- A trashed mail is not a filing: nothing goes into `filed_log.json`, and no deadline is proposed.
- The panel's delete asks for confirmation (listing the subjects, with a warning line for mails the user sent) and offers an undo for a few seconds.

## Tests

Delete then restore returns the archive and both indexes to the identical state; restore refuses on a name clash; a reply to a trashed thread arrives as a new mail; the same `.eml` processed again while in the trash is a duplicate; purge with `--dry-run` changes nothing. See scenarios S08–S10.
