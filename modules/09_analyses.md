# Module 09. Analyses register (Raccoon, the Archivist) · optional

## Purpose

When a topic needs real investigation (a dispute, a technical problem, a decision to prepare), the work lives in a folder inside the project. The register makes sure you can find it again, see where it stands, and resume it weeks later without starting over.

## Conventions

- An analysis is a folder `Projects/<Project>/_Analysis_<topic>/` (or a single `_Analysis_<topic>.md`).
- Inside: the notes and deliverables; helper scripts in `_workings/`; an optional `HANDOFF_<date>.md` written at the end of a working session.
- Each analysis has a **card**, `ANALYSIS_CARD.md`:

```markdown
---
status: waiting          # open | waiting | closed
updated: 2026-03-12
waiting_for: "Acme's reply on warranty terms"
---
## Question
Does the warranty cover the failed pump seals?
## Conclusion so far
Likely yes: clause 7.2 covers seals for 24 months; the failure is at month 14.
## Next step
Send the clause reference to Acme and ask for a free replacement.
```

## Behaviour

- **Scan:** the register reads every `_Analysis_*` folder on each request (nothing is copied into a database): files, first and last change, deliverables, the latest handoff, the card.
- **Page:** lists every analysis with status, last change and next step; opens the folder or the main document; **Resume** copies a prompt (card + latest handoff) for the AI.
- **Rule for the AI:** before starting an analysis, check the register for an existing one and build on it. At the end of any session that worked in an analysis folder, create or update its card.
- **Rule for in-depth work:** open and read every real attachment of the relevant mails (originals, not only the text extracts in the `.md`), and say which ones were read.

## Tests

A folder without a card is listed as "missing summary"; a card's status is read correctly; folders in `_workings/` are not counted as deliverables.
