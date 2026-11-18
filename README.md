# NDA Review Automation

Automating a junior associate's first-level NDA review with [Claude Code](https://claude.com/claude-code). This is part of a Columbia Law School workshop in which students with no coding background automate legal tasks.

The automation reviews an NDA for a client who is **receiving** confidential information. It checks each clause against a Recipient-side playbook and can then mark up the NDA the way a lawyer would.

## What's here

| File | What it is |
|---|---|
| `SKILL.md` | **Review skill.** Reads an NDA, rates each of the 14 playbook topics (Preferred / Fallback / Resist / Missing), proposes wording, and produces a colour-coded Word report. |
| `recipient-playbook.md` | The Recipient-side NDA Review & Negotiation Playbook the review is measured against. |
| `report-format.md` | Layout and colour code of the review report: red = escalate, amber = middle ground, green = OK. |
| `nda-redline/` | **Redline skill.** Applies the review's red items to a copy of the NDA as Word tracked changes with margin notes, editing *within* the other side's language rather than replacing clauses. |
| `CLAUDE.md` | Project notes Claude Code reads automatically: purpose, roadmap and folder layout. |

## How it works

1. Put an NDA (.docx) in an `NDA_input/` folder in your project.
2. Ask Claude Code to **"review this NDA"**. For a mutual NDA, say which party is your client. The report appears in `NDA_output/` as `<NDA name> - Review.docx`.
3. Ask Claude Code to **"redline this NDA"**. A marked-up copy appears as `<NDA name> - Redline.docx`. Open it in Word (Review → All Markup) to see the tracked changes.

## Setting it up in Claude Code

Claude Code loads skills from a project's `.claude/skills/<skill-name>/` folder:

- `.claude/skills/nda-review/`: `SKILL.md`, with `recipient-playbook.md` and `report-format.md` in a `reference/` subfolder
- `.claude/skills/nda-redline/`: `SKILL.md`, with `redline-format.md` and `redlining-principles.md` in a `reference/` subfolder

The review skill also uses a small Word template (`reference/word-template/`) to build its report. That template is not included in this repository yet.

## Important

- Every report and redline is a **preliminary first-level review for a supervising lawyer**, not legal advice.
- Use **fictitious NDAs** only. A review sends the full text of the NDA to Claude.

## Roadmap

1. Recipient-side review and redline (this version)
2. A Discloser-side playbook and more NDA types (mutual vs one-way, simple vs complex)
3. Benchmarks: students annotate expected answers and score the automated reviews
