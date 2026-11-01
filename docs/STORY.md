# Why Menagerie exists

*A short write-up of how this blueprint came about and what building it taught me.*

## The problem

My work inbox gets more than a hundred mails a day, in several languages, about dozens of parallel projects: suppliers, regulators, internal committees, field operations. Folders and rules in the mail client never kept up. Rules match on sender and subject, and the subject of the mail that matters is usually "RE: RE: FW: quick question".

What I wanted was not a smarter inbox. I wanted an assistant that **reads** every mail, attachments included, tells me in three lines what it is about and where it belongs, and then **waits for me to say yes**.

## What I built

A set of plain folders and a few small scripts on my own machine:

- every mail becomes a folder with a Markdown file I can read in any editor;
- an AI coding agent (I use Claude Code) triages the new ones following a written protocol and proposes a destination for each;
- I answer `OK`, a folder name, or `delete`;
- a small local web page shows what is waiting, lets me select a batch, and copies the prompt for the agent.

Over the months it grew: deadlines proposed from mail, a weekly narrative per project, meeting transcripts going through the same triage, a register of in-depth analyses. Each part got an animal name, mostly because "ask the Librarian" is easier to say than "run the search module", and partly because a system you enjoy using is a system you keep using.

It now holds more than 7,000 mails across about fifty projects.

## What I learned

**1. Write the rules down, and make the AI follow the file.** The breakthrough was not a better model. It was a protocol file the agent reads every time: the same output format, the same checks, the same order. Without it, every session invented its own way of presenting things, and I could not scan the proposals quickly.

**2. The human approval is the product.** I tried letting the system file the "obvious" mails by itself. It worked until it didn't, and a mail filed in the wrong place is a mail you never find again. Now nothing moves without my `OK`, except a short, explicit list of exact newsletter and press-review senders.

**3. Delete must mean trash.** One careless `rm -rf` is enough to make you rebuild a whole part of the system around undo.

**4. Indexes rot quietly.** At one point 685 of 3,686 duplicate-shield entries pointed to folders that had moved, and the same supplier thread got archived nine times. Every move now updates the indexes in the same step, and a test guards it.

**5. A conversation is one thing.** Showing replies as separate mails hid the link between them. Grouping by thread and filing a conversation as a unit fixed a whole class of mistakes.

**6. Read the attachments.** In one investigation the decisive evidence existed only in a screenshot: a supplier's label and a specification table. Text search alone had led me, and the AI, to the wrong conclusion.

**7. Performance problems hide in data, not code.** The panel got slower every week. The cause was spreadsheets copied whole into Markdown, one mail at 38 MB, making the search cache 155 MB and the start-up a minute long. Capping the extracts and making the cache incremental brought start-up back to three seconds.

**8. Keep it local.** An email archive is the most sensitive dataset most of us own. Everything here lives on one machine; anything that leaves it is an explicit, previewed action.

## Why a blueprint and not an app

My system is shaped around my job: my mail client, my languages, my projects. Packaging it as an installable app would mean either shipping all of that or spending months making it generic.

But coding agents are now good enough to build software from a precise description. So I wrote down what matters (the architecture, the rules, the protocols, and tests that prove an implementation is safe) and left the code to the agent. Your agent interviews you, builds your version, and has to pass the same acceptance scenarios mine does.

If you build one, I'd like to hear what you changed.
