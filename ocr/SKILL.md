---
name: OCR-Extraction-Skill
description: This skill defines how an AI agent must perform optical character recognition (OCR), text extraction, and document transcription from scanned PDFs, images, screenshots, photographs, and mixed text/image documents.
---

# OCR Extraction Skill

## Purpose

This skill defines how an AI agent must perform optical character recognition (OCR), text extraction, and document transcription from scanned PDFs, images, screenshots, photographs, and mixed text/image documents.

The primary objective is **faithful extraction**, not interpretation. The agent must preserve the source document as accurately as possible and must never invent, guess, complete, or silently correct text that is not visible or reliably extractable.

## Core Principles

1. **Extract, do not infer.**
   - Only output text, numbers, table entries, symbols, headings, dates, names, signatures, footnotes, and labels that are visible or reliably machine-extracted from the source.
   - Do not infer missing words from context.
   - Do not complete partial sentences unless the missing characters are visible.
   - Do not guess values in tables, totals, dates, references, names, account numbers, or legal clauses.

2. **Preserve structure.**
   - Maintain the original reading order, headings, paragraphs, lists, page breaks, table structure, captions, footnotes, and section numbering wherever possible.
   - Preserve tables as tables rather than converting them into prose.
   - Preserve meaningful whitespace, indentation, row grouping, and column relationships.

3. **Be explicit about uncertainty.**
   - Mark unclear, cropped, obscured, handwritten, faint, distorted, or unreadable content clearly.
   - Use consistent uncertainty markers.
   - Do not hide uncertainty by replacing unclear content with likely-looking text.

4. **Maintain auditability.**
   - Keep page references.
   - Where possible, preserve line, paragraph, table, row, column, and section context.
   - Make it possible for a reviewer to trace extracted content back to the source.

5. **Do not alter meaning.**
   - Do not modernise spelling, grammar, punctuation, currency symbols, dates, company names, legal references, or terminology unless explicitly asked.
   - Do not normalise dates, currencies, addresses, or numbers in the extraction output unless producing a separate normalised data layer.

## Permitted Agent Tasks

The agent may:

- Extract visible text from images, scans, PDFs, screenshots, and photographs.
- Extract embedded text from text-based PDFs before using OCR.
- OCR scanned pages where embedded text is absent, incomplete, corrupt, or unreliable.
- Preserve page order and document structure.
- Reconstruct tables where the source clearly shows rows and columns.
- Mark uncertain or illegible content.
- Provide a separate summary, interpretation, or data normalisation only when explicitly requested and clearly separated from the transcription.

The agent must not:

- Invent missing text.
- Guess unclear characters, words, numbers, dates, signatures, initials, names, addresses, tax references, company numbers, bank details, or table values.
- Recalculate totals and insert them into the extraction as though they appeared in the source.
- Treat handwritten content as certain unless it is clearly legible.
- Merge separate pages, tables, or sections in a way that obscures original structure.
- Remove footnotes, annotations, marginal notes, stamps, handwritten additions, or struck-through text if they are visible.
- Present an interpretation as OCR output.

## Source Handling Workflow

### 1. Identify Document Type

Before extraction, classify each source or page as one of the following:

- Text-based PDF with embedded selectable text.
- Scanned PDF page requiring OCR.
- Mixed PDF page containing both embedded text and scanned/image content.
- Photograph of a document.
- Screenshot.
- Form.
- Table-heavy document.
- Handwritten document.
- Poor-quality or partially unreadable document.

### 2. Prefer Embedded Text Where Reliable

For PDFs:

1. Attempt embedded text extraction first.
2. Check whether the extracted text is coherent, complete, and aligned with the visible page.
3. If embedded text is missing, corrupt, duplicated, garbled, or out of order, use OCR for that page.
4. For mixed pages, combine embedded text and OCR only where doing so improves fidelity and does not duplicate content.

### 3. Page-by-Page Extraction

Process documents page by page.

Each page should retain a clear page marker:

```text
--- Page 1 ---
```

For formal outputs, use:

```markdown
## Page 1
```

Do not remove blank pages without noting them. If a page appears blank, state:

```text
[Page appears blank]
```

If a page contains only images, stamps, signatures, diagrams, or tables, describe the visible non-text content briefly and objectively.

## Output Modes

The agent should choose the output mode that best matches the user's request. If no mode is specified, use **faithful Markdown transcription**.

### Mode A: Faithful Markdown Transcription

Use for general OCR and document extraction.

Requirements:

- Preserve headings with Markdown heading levels where the hierarchy is clear.
- Preserve paragraphs and line breaks where meaningful.
- Preserve bullet points and numbered lists.
- Preserve tables as Markdown tables where practical.
- Preserve page breaks.
- Use uncertainty markers for unclear content.

### Mode B: Plain Text Transcription

Use where the user needs raw extracted text.

Requirements:

- Preserve page breaks.
- Preserve line breaks where useful.
- Do not use Markdown tables unless specifically requested.
- Do not add explanatory commentary inside the transcription.

### Mode C: Structured JSON Extraction

Use where the user requests machine-readable output.

Requirements:

- Include page numbers.
- Include field names only where visible or explicitly requested.
- Use `null` for missing values.
- Use an `uncertain` flag where confidence is low.
- Include source context for each extracted item where possible.
- Do not invent schema fields that imply facts not present in the source.

Example:

```json
{
  "pages": [
    {
      "page_number": 1,
      "text_blocks": [
        {
          "text": "Invoice Number: INV-1027",
          "uncertain": false
        },
        {
          "text": "Account Ref: [illegible]",
          "uncertain": true
        }
      ]
    }
  ]
}
```

### Mode D: Table Extraction

Use where the document is primarily table-based or the user specifically asks for table extraction.

Requirements:

- Preserve table rows and columns.
- Do not collapse multiple columns into one field.
- Do not infer missing cells.
- Do not fill blank cells unless the source visibly uses ditto marks, row grouping, or repeated labels.
- Mark blank cells as empty.
- Mark unclear cells as `[unclear]` or `[illegible]`.
- Preserve column headings exactly as shown.
- Preserve footnotes and notes associated with the table.
- Preserve totals as shown, but do not recalculate them unless asked separately.

## Tables

### General Table Rules

When a table appears in the source:

- Recreate it as a Markdown table, CSV, TSV, HTML table, or JSON array depending on the requested output.
- Keep the same column order as the source.
- Keep the same row order as the source.
- Keep multi-line cell content inside the same cell where possible.
- Preserve merged-cell meaning using repeated labels only if clearly required for machine readability. If repeated labels are inserted, state that this was done in a separate note.
- Preserve blank cells as blank cells.
- Preserve currency symbols, commas, decimal places, percentages, brackets, minus signs, dashes, and footnote markers exactly as shown.

### Markdown Table Requirements

Use Markdown tables only when the table is simple enough that Markdown will not distort it.

Example:

```markdown
| Date | Description | Amount |
|---|---:|---:|
| 01/04/2026 | Bank interest | £124.50 |
| 02/04/2026 | [illegible] | £75.00 |
```

Use right alignment markers for numeric columns only where helpful:

```markdown
| Description | Amount |
|---|---:|
```

### Complex Tables

For complex tables with merged cells, nested headings, multiple header rows, schedules, or subtotals:

- Prefer HTML table or structured JSON if Markdown would lose meaning.
- Preserve multiple header rows.
- Preserve section rows and subtotal rows.
- Do not flatten the table into prose.
- Add a note if table structure could not be fully preserved.

Example note:

```text
[Table note: The source table contains merged header cells. The table below preserves the apparent column hierarchy as closely as possible.]
```

### Table Uncertainty

Use cell-level uncertainty markers.

Examples:

```markdown
| Name | Amount |
|---|---:|
| A. Smith | £1,240 |
| [unclear: possibly B. Jones] | £950 |
| C. Patel | [illegible] |
```

Do not write:

```markdown
| B. Jones | £950 |
```

unless the source clearly says `B. Jones`.

## Uncertainty Markers

Use the following standard markers consistently.

| Marker | Meaning |
|---|---|
| `[illegible]` | Text is present but cannot be read. |
| `[unclear]` | Text is partly readable but uncertain. |
| `[unclear: possible text]` | Best-effort reading, but not certain. |
| `[cropped]` | Content appears cut off by the image boundary. |
| `[obscured]` | Content is hidden by a mark, stamp, fold, shadow, object, or overlap. |
| `[blank]` | A field or table cell is visibly blank. |
| `[not visible]` | Expected content is not visible in the supplied image/page. |
| `[handwritten: illegible]` | Handwritten text is present but unreadable. |
| `[signature]` | A signature mark is visible but not transcribed. |

Do not use vague wording such as:

- `probably`
- `looks like`
- `I think`
- `seems to say`

inside the transcription unless using the formal marker:

```text
[unclear: possibly ...]
```

## Handwriting

Handwritten content requires stricter uncertainty handling.

Rules:

- Transcribe handwriting only where it is clearly legible.
- Mark unclear handwriting as `[handwritten: illegible]` or `[unclear: possible text]`.
- Do not infer names, initials, dates, prescription details, clinical findings, tax references, addresses, or monetary amounts from handwriting.
- For signatures, use `[signature]` unless the name is clearly printed or legibly written.
- If handwritten text overlaps printed text, preserve both where possible and mark any ambiguity.

## Forms

For forms, preserve label-value relationships.

Preferred format:

```text
Name: Jane Smith
Date of birth: 12/04/1980
Reference: [illegible]
```

If a checkbox is visible, represent it as:

```text
☑ Yes
☐ No
```

or:

```markdown
- [x] Yes
- [ ] No
```

Do not mark a checkbox as selected unless the mark is clearly present.

If a field is blank, use:

```text
Field name: [blank]
```

If the field is not visible, use:

```text
Field name: [not visible]
```

## Layout Preservation

### Headings

Preserve headings and section hierarchy.

Example:

```markdown
# Agreement

## 1. Definitions

### 1.1 Parties
```

Do not invent heading levels where the structure is unclear. Use bold text or plain text if hierarchy is uncertain.

### Paragraphs

Preserve paragraph breaks.

Do not join unrelated paragraphs.

Do not split a paragraph merely because a line wraps in the scan, unless line breaks are meaningful to the document type, such as poetry, code, addresses, tables, invoices, or statutory wording.

### Lists

Preserve list markers exactly where possible:

- Numbered lists.
- Lettered lists.
- Roman numerals.
- Bullet points.
- Nested lists.

Do not renumber lists unless the source numbering is clearly continuous and OCR spacing has merely separated it.

### Footnotes and Endnotes

Preserve footnote markers and note text.

Example:

```text
The amount is payable on completion.¹

¹ Subject to clause 4.2.
```

If a footnote marker is visible but the footnote text is not, state:

```text
[Footnote marker visible; footnote text not visible on supplied page]
```

### Marginalia, Stamps, Annotations, and Strikethrough

Preserve visible annotations.

Use labels where needed:

```text
[Stamp: RECEIVED 12 JUN 2026]
[Handwritten note in margin: [unclear: possible text]]
[Struck-through text: ...]
```

Do not silently omit stamps, handwritten notes, redactions, highlights, ticks, initials, or struck-through wording.

## Numbers, Dates, Currency, and References

These are high-risk OCR fields and must be handled conservatively.

Rules:

- Preserve exact formatting from the source.
- Do not change `1` to `I`, `0` to `O`, `5` to `S`, or vice versa unless visually clear.
- Do not normalise dates from `01/02/26` to `1 February 2026` unless explicitly asked in a separate normalised output.
- Preserve decimal places.
- Preserve thousands separators.
- Preserve negative number formats, including minus signs and brackets.
- Preserve currency symbols.
- Preserve references, codes, UTRs, company numbers, VAT numbers, NI numbers, account numbers, invoice numbers, and case references exactly as shown.
- If a character is uncertain, mark it.

Example:

```text
Company number: 12[unclear: possible 8]45678
```

Do not silently convert:

```text
£1,OOO.OO
```

to:

```text
£1,000.00
```

unless the source is visually clear or the correction is provided in a separate normalisation note.

## Spelling, Grammar, and Punctuation

The agent must preserve source spelling and punctuation.

Do not:

- Correct spelling mistakes.
- Correct grammar.
- Add missing punctuation.
- Remove repeated words.
- Standardise capitalisation.
- Convert British spelling to American spelling or vice versa.
- Change quotation marks, apostrophes, hyphens, en dashes, em dashes, or mathematical symbols unless clearly required by encoding limitations.

If an OCR engine returns text that appears wrong but the source is unclear, mark it as uncertain rather than correcting it.

## Images, Diagrams, Logos, and Non-Text Content

If the page contains meaningful non-text content, include a concise objective note.

Examples:

```text
[Logo at top left: Pierce]
[Diagram showing three companies connected by ownership arrows]
[Photograph of damaged vehicle]
[Chart with x-axis labelled "Year" and y-axis labelled "Revenue"]
```

Do not interpret diagrams unless asked separately.

For charts, extract visible labels, legends, axis titles, data labels, and captions where possible. Do not estimate plotted values unless exact values are labelled.

## Redactions and Missing Content

Preserve redactions.

Examples:

```text
Name: [redacted]
Address: [redacted]
```

If content is cut off or outside the image boundary:

```text
[cropped]
```

Do not attempt to reconstruct redacted, cropped, or missing content.

## Quality Control Checklist

Before returning OCR output, the agent must check:

- Every page has been considered.
- Page order is preserved.
- Tables remain tables where possible.
- Column and row relationships are preserved.
- Blank cells are not filled by inference.
- Unclear text is marked with uncertainty markers.
- No values have been guessed.
- Dates, numbers, currency amounts, references, and names are not silently corrected.
- Footnotes, stamps, signatures, handwritten notes, and annotations are included or noted.
- Any omitted content is explicitly identified.
- The output format matches the user's request.

## Confidence Reporting

When appropriate, include a short extraction quality note after the transcription, not inside it.

Example:

```markdown
## Extraction Notes

- Pages 1-3 were clear and appear fully extractable.
- Page 4 contains a faint handwritten note marked as `[handwritten: illegible]`.
- The table on page 5 has merged headings; the Markdown version preserves the apparent structure as closely as possible.
```

Do not overstate confidence.

Use cautious wording:

```text
The extraction appears complete except for the marked uncertain areas.
```

Avoid:

```text
The OCR is 100% accurate.
```

## Separate Transcription from Analysis

If the user asks for both OCR and explanation, keep them separate.

Recommended structure:

```markdown
# OCR Transcription

[Faithful extracted text]

# Extraction Notes

[Uncertainty and quality notes]

# Summary / Analysis

[Separate interpretation, only if requested]
```

Never blend interpretation into the OCR transcription.

## Security and Privacy

OCR often involves sensitive information. The agent must:

- Treat documents as confidential.
- Avoid unnecessary repetition of sensitive personal data in summaries.
- Avoid exposing hidden metadata unless explicitly requested.
- Not infer protected characteristics from documents.
- Not reconstruct redacted information.
- Not guess identities from signatures, partial names, addresses, or reference numbers.
- Avoid producing executable code or links from OCR content unless the user explicitly asks and the content is verified.

For passwords, access codes, API keys, bank details, tax references, health data, or legal documents, apply strict fidelity and uncertainty handling.

## OCR Pre-Processing Guidance

Where the agent controls OCR processing, it should consider:

- Deskewing rotated pages.
- Correcting page orientation.
- Increasing contrast for faint scans.
- Removing noise only where it does not remove content.
- Splitting double-page scans into separate pages where appropriate.
- Cropping borders only where this does not remove marginalia or page numbers.
- Using an OCR resolution appropriate to the document.
- Avoiding excessive compression that harms small text or table lines.

Pre-processing must not alter or remove substantive content.

## Multi-Language Documents

For multilingual documents:

- Preserve the original language in the OCR transcription.
- Do not translate unless explicitly requested.
- If translation is requested, provide it separately from the source transcription.
- Preserve accents, diacritics, non-Latin scripts, currency symbols, punctuation, and directionality where possible.
- Mark uncertain characters rather than substituting approximate Latin equivalents.

## File and Batch Processing

For multiple files:

- Process each file separately.
- Preserve original file order or state the order used.
- Use clear file headings.
- Preserve page numbering within each file.
- Do not merge tables or sections from different files unless explicitly requested.

Recommended structure:

```markdown
# File: example.pdf

## Page 1

...

# File: appendix.pdf

## Page 1

...
```

## Required Refusal or Escalation Cases

The agent should clearly state limitations where:

- The image resolution is too low.
- The document is too blurred, cropped, dark, skewed, or obstructed.
- The handwriting is not reliably legible.
- The page is missing.
- The table structure cannot be reconstructed safely.
- The OCR result is incomplete or corrupted.

Use:

```text
I cannot reliably extract this section because [reason]. I have marked it as `[illegible]` rather than guessing.
```

Do not provide a guessed transcription to satisfy completeness.

## Examples

### Good OCR Output

```markdown
## Page 1

Invoice

Invoice No: INV-2041
Date: 14/06/2026
Customer Ref: [unclear: possible AB-17]

| Description | Quantity | Unit Price | Total |
|---|---:|---:|---:|
| Consultancy services | 3 | £750.00 | £2,250.00 |
| Disbursements | [blank] | [blank] | £45.00 |
| Total | | | £2,295.00 |

[Signature]
```

Why this is good:

- The uncertain reference is marked.
- Blank cells remain blank.
- The table is preserved.
- The signature is noted but not guessed.

### Bad OCR Output

```markdown
Invoice No: INV-2041
Date: 14 June 2026
Customer Ref: AB-17
Total due: £2,295.00
Signed by John Smith
```

Why this is bad:

- The date format was changed.
- The uncertain customer reference was guessed.
- The table was collapsed.
- The signature name was invented.

## Agent Completion Rules

Before completing an OCR task, the agent must be able to confirm:

- I did not guess missing or unclear text.
- I preserved the original structure as far as possible.
- I kept tables as tables where practical.
- I marked uncertainty clearly.
- I kept extraction separate from analysis.
- I included page references.
- I noted any unreadable, cropped, blank, or missing areas.

If these conditions are not met, the agent must revise the output before returning it.
