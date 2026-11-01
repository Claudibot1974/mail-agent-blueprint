# Module 02. Index store · core

## Purpose

Own the two archive indexes (BLUEPRINT §5.1) so that no other module ever reads or writes them directly.

## Interface

- `load_threads()` / `save_threads(data)`
- `load_message_ids()` / `save_message_ids(data)`
- `repoint(old_folder, new_folder)`: rewrite every entry of both indexes that points at `old_folder` (used by file, merge, trash, restore and restructure)
- `remove_thread_keys(folder)`: used when a mail goes to the trash
- a `rebuild` command: scan `Unsorted/` and `Projects/`, read each `.md` frontmatter, reconstruct both indexes; `--dry-run` prints the differences, `--apply` writes

## Rules

- **Atomic save:** write to `<file>.tmp`, flush, then rename over the original.
- **Corruption:** if a file cannot be parsed, copy it to `<file>.corrupt_<timestamp>`, stop the run **before touching any mail**, and print the rebuild command. Never start from an empty index silently: an empty duplicate shield means every old mail can be re-ingested.
- **Format stability:** the on-disk format is plain JSON that a human can inspect.
- **Trash semantics:** on delete, thread keys are **removed** (a reply to a trashed thread must come back as a new mail), message-id entries are **repointed** to `Deleted/…` (the same `.eml` processed again is still a duplicate while the mail is in the trash).

## Tests

Atomic save survives a simulated crash between write and rename; a corrupt file is quarantined and the run stops; `repoint` updates both files; rebuild on a known archive reproduces the expected indexes. See scenarios S05, S11, S12.
