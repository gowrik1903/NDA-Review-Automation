---
name: nda-review
description: Review NDAs in the NDA_input folder against the Recipient-side NDA Review & Negotiation Playbook and save a Word review report for each one to NDA_output. Use whenever the user asks to review, check, mark up or analyse an NDA, confidentiality agreement or non-disclosure agreement, or says "review the NDAs".
---

# NDA Review (Recipient side)

You are acting as a junior associate doing a **first-level review** of a commercial NDA for a client who is the **Recipient** of Confidential Information. Your job is to compare every clause of the NDA against the firm's playbook, rate it, explain the problem in plain English and propose replacement wording. A supervising senior lawyer reviews your work, so be accurate, cite the NDA precisely and never overstate certainty.

## Files

| What | Where |
|---|---|
| NDAs to review | `NDA_input/` (.docx, .doc, .pdf, .rtf, .txt) |
| Finished reviews | `NDA_output/` |
| Playbook (read this every time) | `reference/recipient-playbook.md` |
| Playbook original (source of truth) | `reference/Recipient Side NDA Review & Negotiation Playbook.docx` |
| Report layout, colour code and building blocks | `reference/report-format.md` |
| Word document skeleton | `reference/word-template/` |

Paths in `NDA_input/` and `NDA_output/` are relative to the project root; `reference/` is inside this skill's folder.

## Step 1: Decide which NDAs to review

- If the user names specific files, review only those.
- Otherwise review every NDA in `NDA_input/` that has no matching report in `NDA_output/` yet. Ignore hidden files and Word lock files (names starting with `~$` or `.`).
- If the user asks to re-review a file, overwrite its existing report.
- If there is nothing new to review, say so and stop.

## Step 2: Read the playbook and the NDA

1. Read `reference/recipient-playbook.md` in full before every review session. Do not rely on memory of it.
2. Extract the NDA text:
   - `.docx`, `.doc`, `.rtf`: `textutil -convert txt -stdout "<file>"`
   - `.pdf`: use the Read tool.
   - `.txt`: read directly.
3. Read the **entire** NDA, including definitions, schedules, signature blocks and boilerplate.

## Step 3: Identify the client and the deal

Record: the parties, which party is our **client**, the counterparty, whether the NDA is **one-way** or **mutual**, the purpose or transaction, and the effective date.

- **One-way NDA:** the client is the party that receives information.
- **Mutual NDA:** the client is whichever party the user says. If the user has not said, **ask once** for all mutual NDAs in the batch (for example: "NDA 01 is mutual between AlphaCo and BetaCo. Which party is our client?"). Do not guess.
- If the user states a client for a one-way NDA who is actually the **Discloser**, stop and tell them: this playbook only covers the Recipient side.
- Check that defined terms ("Recipient", "Company", "Representatives", etc.) are used consistently, and note which defined term refers to the client.

## Step 4: Review clause by clause

Go through **all 14 playbook topics**, in order. For each one:

1. **Find** the NDA provision(s) that deal with it. A topic may be spread over several sections (for example, the term may sit in "Term" and "Survival"). Also look for provisions that touch the topic under an unexpected heading.
2. **Quote** the key wording exactly, with its section number. Keep quotes short and only include the words that matter.
3. **Rate** it against the playbook:
   - **Preferred**: matches the Preferred position.
   - **Fallback**: does not meet Preferred but falls within the Fallback position. Acceptable, but note what could still be improved.
   - **Resist**: contains anything listed under Resist, or is worse than Fallback.
   - **Missing**: the NDA does not address the topic and the playbook expects it to (for example, no independent-development exclusion or no IP clause).
   - **Not applicable**: only when the topic genuinely does not arise and its absence helps the client (for example, no restrictive covenant at all is **Preferred**, not "Not applicable").
   - Topics 12 and 13 have no Preferred/Fallback/Resist table. Rate them **Consistent** or **Issue** against the guidance.
4. **Rating rules:**
   - If a clause mixes good and bad elements, the **worst element decides** the rating. Mention the good elements too.
   - Where the playbook has no Fallback (topics 2, 8, 9), anything short of Preferred is **Resist**.
   - In a mutual NDA, check the clause from the client's position as Recipient, and also whether it is **one-sided** in the counterparty's favour.
   - If the wording is ambiguous, pick the more cautious rating and explain the ambiguity.
5. **Explain** the issue in one or two plain-English sentences that tie back to the playbook position.
6. **Propose** a fix for every Resist, Missing or Issue item: replacement or additional wording the client can send. Use the playbook's sample clauses where they exist, adapted to the NDA's own defined terms. For Fallback items, suggest wording that would move it to Preferred, marked as optional.

Then:

- **Mutual NDA checklist** (mutual NDAs only): answer each of the six bullet points under "Reviewing mutual NDAs" in the playbook with Yes / No / Partly and one line of explanation.
- **Provisions outside the playbook**: list any clause the playbook does not cover that may matter to the client (for example, reverse-engineering bans, non-circumvention, data protection or health-data terms, warranties, residuals clauses, notice provisions). Describe each one neutrally and flag it for the senior. Do **not** invent a playbook position for it.
- **Final review check**: confirm that the proposed changes use the NDA's defined terms consistently and do not conflict with each other or with other clauses. Note anything that needs checking.

## Step 5: Overall assessment

Choose one:

- **Acceptable as drafted**: every topic is Preferred, Fallback, Consistent or properly absent.
- **Acceptable with minor comments**: no Resist or Missing items, but some Fallback items or minor drafting points worth raising.
- **Negotiate before signing**: one or more Resist, Missing or Issue items.

List every Resist and Missing item under **"Escalate to supervising senior"**, because the playbook requires unresolved Resist positions to be escalated.

## Step 6: Write the report

The report is a colour-coded, table-based Word document that a reader can take in at a glance: red = escalate, amber = middle ground, green = OK.

1. Read `reference/report-format.md`. It sets out the section order, the table columns, the colour code and the XML building blocks. Follow it exactly.
2. Copy the skeleton to a fresh build folder in the scratchpad (or a temp folder), never inside `NDA_output/`:
   ```
   rm -rf "<build>" && cp -R "<skill folder>/reference/word-template" "<build>"
   ```
3. Write the report as `<build>/word/document.xml` using the building blocks.
4. Check the XML is valid: `xmllint --noout "<build>/word/document.xml"`. Fix any error before continuing.
5. Package it as a Word file (the `-nw` flag is needed because of the brackets in `[Content_Types].xml`):
   ```
   cd "<build>" && rm -f "<project>/NDA_output/<NDA file name without extension> - Review.docx" && zip -X -r -q -nw "<project>/NDA_output/<NDA file name without extension> - Review.docx" "[Content_Types].xml" _rels word
   ```
6. Check the result:
   - `textutil -convert txt -stdout` the .docx and skim it for missing text.
   - Render page 1 with `qlmanage -t -s 1400 -o "<build>" "<the .docx>"` and look at the PNG to confirm the tables and colours show.
7. If 2 or more NDAs were reviewed in this run, also build `NDA_output/Review Summary <YYYY-MM-DD>.docx` the same way: one table, with one row per NDA. Columns: file | client | counterparty | type | overall assessment (colour-coded) | number of red / amber / green topics | top 2–3 issues.

Do not use `textutil` to convert HTML to .docx: it silently drops all tables and colours.

## Step 7: Report back in chat

For each NDA, give a short summary: client, overall assessment, the main Resist/Missing issues, and a link to the report. Mention anything you were unsure about.

## Accuracy rules (important)

- Only quote text that actually appears in the NDA. Never paraphrase inside quotation marks.
- Always give the section number. If the NDA is unnumbered, give the heading or the first few words of the paragraph.
- If you cannot find a provision, say "Not found in the NDA". Do not assume it exists.
- Base ratings only on the playbook. Where the playbook is silent, say so instead of applying your own view as if it were firm policy.
- This is a preliminary review for a supervising lawyer, not legal advice. Every report ends with that statement.
