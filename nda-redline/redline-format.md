# Writing tracked changes and comments in a Word file

A .docx is a zip of XML files. The text is in `word/document.xml`, as paragraphs (`<w:p>`) containing runs (`<w:r>`) of text (`<w:t>`). A tracked change wraps runs in `<w:del>` (deleted) or `<w:ins>` (inserted). Word shows these as red strike-through and underline, which the other side can accept or reject.

Use the values below everywhere:
- `A` = `Recipient counsel (draft)` (author)
- `D` = today as `YYYY-MM-DDT00:00:00Z` (date)
- Every `<w:ins>`, `<w:del>` and comment needs a `w:id` that is unique in the document. Use 0, 1, 2… for comments and 100, 101, 102… for insertions and deletions.

Text must be XML-escaped: `&` → `&amp;`, `<` → `&lt;`, `>` → `&gt;`. Quotes and apostrophes can stay as they appear in the original.

If the original run has formatting (`<w:rPr>…</w:rPr>` right after `<w:r>`), copy that `<w:rPr>` into every run you create from it, including inside `<w:del>` and `<w:ins>`.

## 1. Change words inside a paragraph

Split the original run into pieces: unchanged text, a deletion and an insertion. Example: replacing "within twenty-four (24) hours" with "promptly".

Original:
```xml
<w:r><w:t>… in writing within twenty-four (24) hours after becoming aware …</w:t></w:r>
```
Redlined:
```xml
<w:r><w:t xml:space="preserve">… in writing </w:t></w:r>
<w:del w:id="105" w:author="A" w:date="D"><w:r><w:delText xml:space="preserve">within twenty-four (24) hours</w:delText></w:r></w:del>
<w:ins w:id="106" w:author="A" w:date="D"><w:r><w:t xml:space="preserve">promptly</w:t></w:r></w:ins>
<w:r><w:t xml:space="preserve"> after becoming aware …</w:t></w:r>
```
Rules:
- Inside `<w:del>`, text goes in `<w:delText>`, never `<w:t>`.
- Always add `xml:space="preserve"` so spaces at the edges are kept.
- Spaces belong to the unchanged run where possible, so the result reads correctly whether the change is accepted or rejected.

## 2. Delete a whole paragraph

Mark both the text and the paragraph mark as deleted, so the empty line disappears when accepted. Add `<w:rPr><w:del …/></w:rPr>` inside the paragraph's `<w:pPr>` (create `<w:pPr>` if there is none; if it has a `<w:pStyle>`, put `<w:rPr>` after it):
```xml
<w:p …><w:pPr><w:rPr><w:del w:id="110" w:author="A" w:date="D"/></w:rPr></w:pPr>
<w:del w:id="111" w:author="A" w:date="D"><w:r><w:delText xml:space="preserve">3.2 Confidential Information shall not be deemed …</w:delText></w:r></w:del></w:p>
```

## 3. Insert a new paragraph

Add a new `<w:p>` after the related paragraph. Mark its paragraph mark and its text as inserted:
```xml
<w:p><w:pPr><w:rPr><w:ins w:id="120" w:author="A" w:date="D"/></w:rPr></w:pPr>
<w:ins w:id="121" w:author="A" w:date="D"><w:r><w:t xml:space="preserve">6.2 Nothing in this Agreement transfers ownership of …</w:t></w:r></w:ins></w:p>
```
Copy the `<w:pStyle>` of the neighbouring body paragraphs into the new `<w:pPr>` if they have one.

## 4. Attach a comment

Wrap the first change for a topic in a comment range, then add the reference run right after the range end:
```xml
<w:commentRangeStart w:id="0"/>
<w:del …>…</w:del><w:ins …>…</w:ins>
<w:commentRangeEnd w:id="0"/><w:r><w:commentReference w:id="0"/></w:r>
```
Each comment's text goes in `word/comments.xml` (create it):
```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<w:comments xmlns:w="http://schemas.openxmlformats.org/wordprocessingml/2006/main">
<w:comment w:id="0" w:author="A" w:date="D" w:initials="RC"><w:p><w:r><w:t xml:space="preserve">Playbook 7. Term and survival: confidentiality should run 1 year (at most 3 years) from disclosure, not in perpetuity.</w:t></w:r></w:p></w:comment>
</w:comments>
```

Register the comments file:
- In `word/_rels/document.xml.rels`, add before `</Relationships>`:
  `<Relationship Id="rIdRedlineComments" Type="http://schemas.openxmlformats.org/officeDocument/2006/relationships/comments" Target="comments.xml"/>`
- In `[Content_Types].xml`, add before `</Types>`:
  `<Override PartName="/word/comments.xml" ContentType="application/vnd.openxmlformats-officedocument.wordprocessingml.comments+xml"/>`

## 5. Checks before packaging

1. **Valid XML:** `xmllint --noout` on `word/document.xml`, `word/comments.xml`, `word/_rels/document.xml.rels` and `[Content_Types].xml`.
2. **Matching ids:** every `commentRangeStart` id has a matching `commentRangeEnd`, a `commentReference` and a `<w:comment>`. Count them with `grep -o '<w:commentRangeStart' … | wc -l` and the same for the others.
3. **Nothing lost:** rejecting every change must give back the original word for word. Extract the text of the original and of the redline with all insertions removed, then compare (macOS's built-in `perl` does the extraction):
   ```
   perl -0ne 's#</w:p>#</w:p>\n#g; for (split /\n/) { my $t=""; while (/<w:(?:t|delText)(?: [^>]*)?>([^<]*)</g) { $t.=$1 } print "$t\n" if length $t }' "<unpacked original>/word/document.xml" > orig.txt
   perl -0ne 's#<w:ins [^>]*>.*?</w:ins>##gs; for (split /\n/) { my $t=""; while (/<w:(?:t|delText)(?: [^>]*)?>([^<]*)</g) { $t.=$1 } print "$t\n" if length $t }' "<build>/word/document.xml" > rejected.txt
   diff orig.txt rejected.txt && echo "Reject-all matches the original"
   ```
   Unzip the original into its own folder in the scratchpad for this comparison.
4. **Reads well when accepted:** print the accepted version and read every changed clause:
   ```
   perl -0ne 's#<w:del [^>]*>.*?</w:del>##gs; for (split /\n/) { my $t=""; while (/<w:t(?: [^>]*)?>([^<]*)</g) { $t.=$1 } print "$t\n" if length $t }' "<build>/word/document.xml"
   ```
5. **Preview:** Quick Look (`qlmanage`) shows the accepted text only, not the red markup. That is expected; tracked changes show in Word.
