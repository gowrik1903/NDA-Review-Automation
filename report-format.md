# Review report format

Each review is a Word document (landscape, Calibri) built from the skeleton in `word-template/`. You only write one file, `word/document.xml`, using the building blocks below. Everything else (fonts, heading styles, table borders) is already set up in the skeleton.

## Colour code (use exactly these)

| Colour | Label in Status cell | Used for | Cell fill | Text colour |
|---|---|---|---|---|
| Red | **ESCALATE** | Resist, Missing, Issue; Mutual checklist "No"; overall "Negotiate before signing" | `FFC7CE` | `9C0006` |
| Amber | **MIDDLE GROUND** | Fallback; Mutual checklist "Partly"; overall "Acceptable with minor comments" | `FFEB9C` | `7F6000` |
| Green | **OK** | Preferred, Consistent, Not applicable; Mutual checklist "Yes"; overall "Acceptable as drafted" | `C6EFCE` | `006100` |
| Grey | **FLAG** | Provisions outside the playbook (no rating) | `E7E6E6` | `3A3A3A` |

Other fills: table header row `1F3864` (white bold text, style `CellHeader`), label cells `DEEAF6`.

Status cells always have two lines: the bold coloured label (ESCALATE / MIDDLE GROUND / OK), then the playbook rating in normal text (Resist, Missing, Issue, Fallback, Preferred, Consistent, Not applicable).

## Report layout (in this order)

The page is landscape with 13680 twips of usable width. Every table's column widths must add up to 13680.

1. **Title**: `NDA Review: <NDA title> (<Party A> / <Party B>)`, style `Title`. Then a `Subtitle` line: `Recipient-side first-level review against the Recipient-Side NDA Review and Negotiation Playbook · Reviewed <date>`.
2. **At a glance**: a 4-column table (widths 4800 / 2960 / 2960 / 2960) with one row:
   - Overall assessment cell: label "OVERALL ASSESSMENT", then the verdict, filled with the matching colour.
   - Red cell: "ESCALATE", then the count of red topics, then "Resist / Missing / Issue".
   - Amber cell: "MIDDLE GROUND", then the count, then "Fallback".
   - Green cell: "OK", then the count, then "Preferred / Consistent".
   - The three counts must add up to 14.

   Below the table, add a `Note` paragraph with the colour key: "Colour key: Red = escalate to supervising senior (Resist, Missing or Issue) · Amber = acceptable middle ground (Fallback) · Green = OK (Preferred or Consistent) · Grey = outside the playbook, flagged only."
3. **Heading 1 "Deal details"**: a 4-column table (2200 / 4640 / 2200 / 4640), with label cells filled `DEEAF6`. Rows:
   - Our client (Recipient) | Counterparty
   - NDA type | Effective date
   - Purpose / transaction | File reviewed

   Give each party's name, entity type, location and defined term.
4. **Heading 1 "Issues for the supervising senior"**: one row per red topic, most important first. Columns (450 / 2200 / 1100 / 1350 / 4580 / 4000): # | Playbook topic | NDA § | Status | What's wrong (one or two sentences) | Recommended action (one sentence; full wording is in the next section). If there are no red topics, write one row saying "No items require escalation."
5. **Heading 1 "Clause-by-clause review"**: one row for **each** of the 14 playbook topics, in playbook order. Columns (450 / 1800 / 1350 / 3000 / 3080 / 4000):
   - **#**
   - **Playbook topic**: the topic name in bold, then the NDA § on a second line.
   - **Status**: colour-coded.
   - **What the NDA says**: a short exact quote, in italics and in quotation marks, or "Not found in the NDA".
   - **Issue**: plain English, tied to the playbook. Mention good elements too.
   - **Recommendation**:
     - Red rows: a bold "Proposed wording:" line, then the replacement or new wording in italics, then optionally one line of explanation.
     - Amber rows: a bold "Optional:" line with wording that would reach Preferred, or "Acceptable; no change needed."
     - Green rows: "No change needed." plus any optional clarification.

   The header row repeats on each page (`<w:tblHeader/>`).
6. **Heading 1 "Mutual NDA checklist"** (mutual NDAs only; omit the section entirely for one-way NDAs): the six playbook questions. Columns (6000 / 1300 / 6380): Question | Answer (colour-coded Yes / Partly / No) | Explanation.
7. **Heading 1 "Provisions outside the playbook"**: columns (1300 / 1350 / 2600 / 8430): NDA § | Status (grey FLAG) | Provision | Why it may matter. If there are none, write one row saying "None identified."
8. **Heading 1 "Final review check"**: columns (3000 / 10680), label cells filled `DEEAF6`. Rows: Defined terms | Interaction between proposed changes | Anything else for the senior.
9. **Closing `Note` paragraph**: "This is a preliminary, automated first-level review prepared against the firm's Recipient-side NDA playbook. It is not legal advice and must be reviewed by a supervising lawyer before it is relied on or shared with the client or counterparty."

## Building blocks (copy and fill)

Text must be XML-escaped: `&` → `&amp;`, `<` → `&lt;`, `>` → `&gt;`. Straight and curly quotes, apostrophes and `§` can be used as they are.

**Document wrapper** (everything goes between `<w:body>` and `<w:sectPr>`):
```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<w:document xmlns:w="http://schemas.openxmlformats.org/wordprocessingml/2006/main"><w:body>
  ...content...
<w:sectPr><w:pgSz w:w="15840" w:h="12240" w:orient="landscape"/><w:pgMar w:top="1080" w:right="1080" w:bottom="1080" w:left="1080" w:header="540" w:footer="540" w:gutter="0"/></w:sectPr>
</w:body></w:document>
```

**Paragraph with a style** (`Title`, `Subtitle`, `Heading1`, `Note`, or leave out `<w:pPr>` for normal text):
```xml
<w:p><w:pPr><w:pStyle w:val="Heading1"/></w:pPr><w:r><w:t xml:space="preserve">Deal details</w:t></w:r></w:p>
```

**Runs inside a paragraph**: plain, bold, italic, and bold coloured:
```xml
<w:r><w:t xml:space="preserve">plain </w:t></w:r>
<w:r><w:rPr><w:b/></w:rPr><w:t xml:space="preserve">bold </w:t></w:r>
<w:r><w:rPr><w:i/></w:rPr><w:t xml:space="preserve">"italic quote"</w:t></w:r>
<w:r><w:rPr><w:b/><w:color w:val="9C0006"/></w:rPr><w:t xml:space="preserve">ESCALATE</w:t></w:r>
```

**Table start** (list one `gridCol` per column):
```xml
<w:tbl><w:tblPr><w:tblStyle w:val="ReviewTable"/><w:tblW w:w="13680" w:type="dxa"/><w:tblBorders><w:top w:val="single" w:sz="4" w:space="0" w:color="A6A6A6"/><w:left w:val="single" w:sz="4" w:space="0" w:color="A6A6A6"/><w:bottom w:val="single" w:sz="4" w:space="0" w:color="A6A6A6"/><w:right w:val="single" w:sz="4" w:space="0" w:color="A6A6A6"/><w:insideH w:val="single" w:sz="4" w:space="0" w:color="A6A6A6"/><w:insideV w:val="single" w:sz="4" w:space="0" w:color="A6A6A6"/></w:tblBorders><w:tblLayout w:type="fixed"/></w:tblPr>
<w:tblGrid><w:gridCol w:w="450"/><w:gridCol w:w="1800"/>...</w:tblGrid>
```
End every table with `</w:tbl>`, then add an empty paragraph `<w:p/>` after it so tables never touch.

**Header row** (one cell shown; repeat for each column):
```xml
<w:tr><w:trPr><w:tblHeader/></w:trPr>
<w:tc><w:tcPr><w:tcW w:w="450" w:type="dxa"/><w:shd w:val="clear" w:color="auto" w:fill="1F3864"/></w:tcPr><w:p><w:pPr><w:pStyle w:val="CellHeader"/></w:pPr><w:r><w:t xml:space="preserve">#</w:t></w:r></w:p></w:tc>
</w:tr>
```

**Ordinary cell** (leave out `<w:shd>` for no fill; add more `<w:p>` blocks for extra lines):
```xml
<w:tc><w:tcPr><w:tcW w:w="3000" w:type="dxa"/></w:tcPr><w:p><w:pPr><w:pStyle w:val="Cell"/></w:pPr><w:r><w:t xml:space="preserve">text</w:t></w:r></w:p></w:tc>
```

**Status cell** (red example; swap the fill, text colour and labels for amber, green or grey):
```xml
<w:tc><w:tcPr><w:tcW w:w="1350" w:type="dxa"/><w:shd w:val="clear" w:color="auto" w:fill="FFC7CE"/></w:tcPr>
<w:p><w:pPr><w:pStyle w:val="Cell"/></w:pPr><w:r><w:rPr><w:b/><w:color w:val="9C0006"/></w:rPr><w:t xml:space="preserve">ESCALATE</w:t></w:r></w:p>
<w:p><w:pPr><w:pStyle w:val="Cell"/></w:pPr><w:r><w:rPr><w:color w:val="9C0006"/></w:rPr><w:t xml:space="preserve">Resist</w:t></w:r></w:p></w:tc>
```

**Body row**: `<w:tr>` + one `<w:tc>` per column + `</w:tr>`. Every row must have exactly as many cells as the table has `gridCol` entries.
