# Contributing

Thanks for wanting to improve Menagerie. It is a blueprint, so most contributions are changes to documents: clearer rules, better module sheets, new scenarios, lessons from your own build.

## Before you start

- **Small fixes** (typos, unclear sentences, broken links): open a pull request directly.
- **Bigger changes** (a new principle, a new module, a change to the triage format): open an issue first and describe the problem you want to solve. It saves you work if the idea doesn't fit.

## Rules for contributions

1. **Keep it generic.** No real names, companies, addresses, mail content or file paths from your own system. Examples use invented people and `example.com` addresses.
2. **Every rule comes with its reason.** Write why it exists, ideally from something that went wrong. The reason is what lets an AI agent apply the rule in situations the text does not cover.
3. **Every new rule or module comes with a test.** Add a scenario to [acceptance/SCENARIOS.md](acceptance/SCENARIOS.md), and a sample mail in [acceptance/sample_mails/](acceptance/sample_mails/) if needed.
4. **Respect the principles.** A change that weakens a principle in [PRINCIPLES.md](PRINCIPLES.md) (for example letting the system file or delete without approval) needs a strong case in an issue first.
5. **Plain English.** Short sentences, concrete words. Many readers are not native speakers, and AI agents follow precise text better.

## Sharing your build

Built your own version? Open an issue labelled `show-and-tell` with your stack, what you changed and what you learned. Real builds are the best feedback this blueprint can get.
