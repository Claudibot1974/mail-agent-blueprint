# Module 08. Meetings (Bear, the Interpreter) · optional

## Purpose

Bring meeting transcripts into the same triage flow as mail, so a meeting's decisions are filed in the right project next to the mails about it.

## Behaviour

- **Drop folder:** `Meetings/` receives transcripts: subtitle files (`.vtt`) or documents exported from the meeting tool, or files written by a local recorder.
- **Processor:** turns each transcript into a folder in `Unsorted/` with `direction: "meeting"`, the transcript as `[hh:mm:ss] **Speaker**: text`, and the original file in `attachments/`. It writes no notes and files nothing.
- **At triage:** the AI writes the meeting note (decisions, actions with owners, open questions) in the user's language, then proposes a project like any other mail.
- **Editing view (nice to have):** a panel page to correct lines, rename speakers ("Speaker 2" → a real name), add the user's own notes and delete lines before processing, with backups.

## Recording, if you build a recorder

- **Consent first.** The recorder starts only after the user confirms they told everyone the meeting is recorded. No secret recording, ever.
- Keep only the text; don't store the audio.
- Prefer on-device speech recognition. Check what quality it reaches for each language you need before relying on it.
- Telling speakers apart reliably needs a proper neural model; simple voice fingerprints don't work. Plain "Me / Remote" channels (microphone vs. computer audio) are honest and useful.

## Tests

A transcript becomes one folder with `direction: "meeting"`; the original moves to `Meetings/_done/`; the same file processed twice is a duplicate.
