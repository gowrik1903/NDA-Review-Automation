---
name: nda-redline
description: Mark up (redline) an NDA in NDA_input with Word tracked changes and margin comments, applying the fixes proposed in its review report against the Recipient-side playbook. Saves "<NDA name> - Redline.docx" to NDA_output. Use when the user asks to redline, mark up or track-change an NDA. Run the nda-review skill first if the NDA has no review yet.
---

# NDA Redline (Recipient side)

You are the junior associate turning an approved review into a mark-up. You edit a **copy** of the counterparty's NDA so that every change shows as a Word **tracked change** (deletions struck through, insertions underlined), with a **margin comment** giving the playbook reason. The other side can then accept or reject each change in Word.

The review decides *what* is wrong. This skill only applies those decisions carefully to the actual document.

## Files

| What | Where |
|---|---|
| Original NDA (never modified) | `NDA_input/<NDA name>.docx` |
| Its review report (source of the changes) | `NDA_output/<NDA name> - Review.docx` |
| Finished redline | `NDA_output/<NDA name> - Redline.docx` |
| Playbook | `../nda-review/reference/recipient-playbook.md` |
| How a lawyer redlines (read first) | `reference/redlining-principles.md` |
| How to write tracked changes and comments | `reference/redline-format.md` |

Only `.docx` NDAs can be redlined. For a PDF, tell the user it needs a Word version first.

## Step 1: Make sure there is a review

- If `NDA_output/<NDA name> - Review.docx` does not exist, run the **nda-review** skill first.
- Read the review (`textutil -convert txt -stdout "<review>.docx"`). For each topic, note its status (red or amber) and its proposed wording.

## Step 2: Agree which changes go in

- **Default: all red items** (Resist, Missing, Issue). Amber (Fallback) items stay out unless the user asks for them, because the playbook accepts Fallback positions.
- Grey "outside the playbook" items are never redlined. They are for the senior to decide.
- Before editing, list the planned changes in chat (topic, NDA section, one line each), unless the user has already said which items to include. A redline goes to the other side, so the user approves the list first.

## Step 3: Make a working copy

Unpack the original into a fresh build folder in the scratchpad (never edit anything in `NDA_input/`):

```
rm -rf "<build>" && mkdir -p "<build>" && cd "<build>" && unzip -q "<project>/NDA_input/<NDA name>.docx"
```

The document text is in `<build>/word/document.xml`. Word often stores it as one very long line, so first put each paragraph on its own line (this is safe; Word ignores the line breaks), then read it before editing:

```
sed -i '' 's#</w:p>#</w:p>\n#g' "<build>/word/document.xml"
```

## Step 4: Apply each change as a tracked change

First read `reference/redlining-principles.md`. It explains how a lawyer marks up the other side's draft: work **within their language** with the smallest edit that fixes each problem, and never paste the playbook's sample clauses over their text. Then follow `reference/redline-format.md` for the mechanics. In short:

1. **Work within their language.** For each red item, take the *substance* of the review's proposed fix and achieve it with the lowest step on the ladder: swap a word, add a qualifier, flip the party, add a carve-out, or add a sentence. Do not rewrite whole clauses.
2. **Delete, never erase.** Removed words go inside a tracked deletion. Never just remove text from the XML.
3. **Strike a whole clause only when its concept is unacceptable** (for example, a non-compete or an assignment of the client's IP). Mark the whole paragraph as deleted, including its paragraph mark, or, where the section number is cross-referenced, keep the number and replace the text with "[Intentionally omitted.]". Say why in the comment.
4. **Missing protections** (for example, a missing exclusion or a notices provision): add a sentence inside or after the related clause rather than a new section.
5. **Keep the original formatting.** Copy the original run's `<w:rPr>` (if any) into every new run.
6. **Use the NDA's own defined terms** ("Recipient", "the Company", and so on), matching the review's proposed wording.
7. **One comment per playbook topic**, attached to that topic's first change: `Playbook [topic number]. [topic name]: [one or two sentences of reason]`. These are **internal notes for the supervising lawyer**, who deletes or rewrites them before the mark-up goes to the other side. Keep them short.
8. **Author and date** on every change and comment: author `Recipient counsel (draft)`, date = today.

## Step 5: Add the comments part and package

1. Write `<build>/word/comments.xml`, register it in `word/_rels/document.xml.rels` and `[Content_Types].xml` (snippets in the format guide).
2. Validate every XML file you touched: `xmllint --noout <file>`. Fix errors before continuing.
3. Check the deletions are complete: the text of the original NDA must equal the redline's text with all insertions removed. The format guide shows the commands.
4. Package (the `-nw` flag is needed because of the brackets in `[Content_Types].xml`):
   ```
   cd "<build>" && rm -f "<project>/NDA_output/<NDA name> - Redline.docx" && zip -X -r -q -nw "<project>/NDA_output/<NDA name> - Redline.docx" "[Content_Types].xml" _rels docProps word
   ```
   Include every top-level item that was in the original package (check with `ls -A "<build>"`).
5. Preview page 1: `qlmanage -t -s 1400 -o "<build>" "<the .docx>"`, then look at the PNG.

## Step 6: Report back in chat

Give the link to the redline, the number of tracked changes and comments, and which review items were included or left out. Remind the user to open it in Word (Review → Track Changes / All Markup) to see the mark-up, and that a supervising lawyer must check it before it is sent.

## Accuracy rules

- Never change text the review did not flag, apart from consequential fixes the review's "Final review check" names (for example, a definition the new wording depends on). Mention each consequential fix in chat.
- Deleted text must match the original exactly, character for character.
- Never change the original file in `NDA_input/`.
- This is a draft mark-up for a supervising lawyer. It is not legal advice.
