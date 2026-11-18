# NDA Review Automation

## What this project is

This project is part of a workshop for **Columbia Law School** students, developed with a professor. In the workshop, students automate a legal task with Claude Code or a similar tool. **The students have no coding background**, so everything here should stay simple, readable and explainable in plain English. Prefer plain-language instructions (skills, markdown) over scripts and code.

The first task is **NDA review**. Students get a legal playbook, the guidance a junior associate would use to review an NDA, and automate the first-level review against it.

## Roadmap

1. **Now:** automate first-level review of NDAs against the **Recipient-side** playbook. This is the version being presented to the professor.
2. **Later:** add a **Discloser-side** playbook, so NDAs can be reviewed for either side. Also support different NDA types: mutual vs one-way, simple vs complex, and different kinds of business transactions.
3. **Later:** students build **legal benchmarks** to test the automated reviews. This covers creating a benchmark, annotating data and judging whether results are correct.

Design choices should leave room for these steps. For example, keep playbooks as separate reference files so another side or NDA type can be added next to them, and keep the review output structured and consistent so it can be scored against annotated answers later.

## Layout

| Path | Purpose |
|---|---|
| `NDA_input/` | NDAs waiting to be reviewed |
| `NDA_output/` | Finished review reports (.docx), one per NDA, plus a batch summary when several are reviewed |
| `.claude/skills/nda-review/` | The review automation: `SKILL.md` (workflow), `reference/recipient-playbook.md` (playbook text), the original playbook .docx, `reference/report-format.md` (report layout and colour code) and `reference/word-template/` (Word skeleton) |
| `.claude/skills/nda-redline/` | Step two: applies a review's red items to a copy of the NDA as Word tracked changes with margin comments. `SKILL.md` (workflow) and `reference/redline-format.md` (how to write tracked changes) |
| `NDA Review Skill/`, `NDA Redline Skill/` | Readable copies of the skills' text files for viewing and sharing. The working skills are in `.claude/skills/`; resync the copies after any change |

## How to run a review

Put NDAs in `NDA_input/` and ask Claude to "review the NDAs" (or run `/nda-review`). For mutual NDAs, say which party is the client, or Claude will ask.

To mark up a reviewed NDA, ask Claude to "redline NDA 02" (or run `/nda-redline`). The result is `NDA_output/<NDA name> - Redline.docx`. Its margin comments are internal notes for the supervising lawyer and must be removed before anything goes to the other side.

## Notes

- The playbook .docx in the skill folder is the source of truth. If it changes, regenerate `recipient-playbook.md` from it.
- Reports are colour-coded, table-based Word documents (red = escalate, amber = middle ground, green = OK). They are built by filling in a Word skeleton and zipping it with built-in macOS tools (`xmllint`, `zip`), so no extra software is needed. Do not use `textutil` to make .docx files: it drops all tables and colours. This Mac has no Python or Node installed; don't depend on them.
