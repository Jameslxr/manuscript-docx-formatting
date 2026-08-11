---
name: manuscript-docx-formatting
description: Convert scientific and biomedical DOCX drafts and submission-package documents into restrained, natural, submission-style Word files without scientific review. Use for format-only repair or audit of natural empty paragraphs, zero paragraph spacing, explicit line spacing, role-aware cover-letter and response-letter rhythm, left-aligned manuscript front matter, formal black typography, continuous Word line numbers, dynamic page numbers, margins, headings, or current target-journal layout. Apply a fail-closed whole-document gate to every modified DOCX. Ask for a target journal only when journal-specific compliance is requested.
---

# Manuscript DOCX Formatting

## Scope and routing

Perform Word-format repair and validation only. Preserve scientific wording,
claims, citations, reference order, figures, tables, and section order unless the
user explicitly authorizes content changes.

Do not run scientific review, evidence checking, language revision, journal
selection, or a scientific submission gate. Route those requests to
`manuscript-review-revision`.

Resolve this file's directory as `SKILL_ROOT`; never assume the current working
directory is the skill directory.

Select the smallest applicable lane:

| Lane | Trigger | Required work |
|---|---|---|
| `audit-only` | diagnose formatting without edits | inventory, applicable audits, full rendering |
| `baseline-fix` | a simple DOCX format edit | whole-document paragraph, spacing, typography, line/page-number gate |
| `manuscript-profile` | a manuscript draft must become natural and submission-style | baseline plus deterministic front-matter normalization and audit |
| `submission-package-profile` | cover letter, response letter, highlights, declaration, or editable package text | baseline plus role-aware package normalization and package audit |
| `journal-format` | named journal or supplied template | applicable lane plus current official source-linked overrides |

The lanes are modular, but the baseline gate is not optional after a write. A
one-property edit cannot bypass whole-document spacing and numbering checks.

In `journal-format`, make the target journal the first intake question only when
it is missing and cannot be inferred. Ask at most one concise follow-up at a
time, and only when the answer would materially change the output. Infer article
type, blinded/unblinded state, and submission stage from supplied files when
possible.

## Non-negotiable baseline after every DOCX write

- Body prose and empty separators have effective `spaceBefore=0 pt` and
  `spaceAfter=0 pt`; automatic before/after spacing is absent or disabled.
- Adjacent body-prose paragraphs are separated by exactly one structurally empty
  Word paragraph: `text¶`, `¶`, `next text¶`. Never use paragraph spacing or a
  manual line break as a substitute.
- Body prose and empty separators carry the exact resolved line-spacing token;
  default to `double` when no current source specifies another value.
- In a manuscript, the same resolved line-spacing token also applies to title,
  authors, affiliations, correspondence, Keywords, every heading/subheading,
  and declaration/CRediT paragraphs. Do not mix single-spaced front matter or
  headings with 1.5/double-spaced prose.
- In a submission-package file, one resolved token applies to every visible
  body and table-cell paragraph, including salutation, body, closing,
  signature, reviewer-comment, response, declaration, and empty separators.
  Use `single` as the package fallback when no current journal source or user
  instruction specifies another token.
- Every section has continuous Word-native line numbering (`countBy=1`), with no
  paragraph-level suppression.
- Every active page story has one dynamic `PAGE` field, continuous across
  sections. Default to the upper-right header; use another position only as a
  recorded official-template or user override.
- Every non-empty top-level paragraph style is classified as body prose or
  semantic non-body content for the structural audit.
- Typography is formal and black. Do not introduce report colors, banners,
  cards, decorative rules, or publication-facsimile styling.

## Journal-neutral manuscript profile

When no current official source resolves a different rule, normalize a
manuscript to:

- 1-inch margins and top vertical alignment;
- left-aligned title (Times New Roman 15 pt bold), plus authors, affiliations,
  correspondence, Keywords, headings, declaration text, and body prose all at
  the resolved body size (12 pt fallback), with one global resolved line-spacing
  token and 0/0 pt paragraph spacing;
- no table, text box, shape, centered display block, or decorative container for
  title-page content;
- title, authors, affiliations, correspondence, then one empty paragraph before
  `Abstract` in the integrated profile;
- a bold `Keywords:` label with regular keyword terms, followed by exactly one
  real empty paragraph before `Introduction` or the next main-text heading;
- exactly one real empty paragraph before every new section, subsection, and
  declaration/CRediT block; no empty paragraph between a heading and its first
  body paragraph; collapse missing or duplicated semantic separators;
- a dedicated non-body reference role after `References`/`Bibliography`; never
  insert body-prose separators between reference entries;
- required title, authors, affiliations, and corresponding-author details in an
  unblinded submission-ready manuscript.

Use explicit role styles or `--<role>-paragraph` arguments after inventory. Do
not guess identities from appearance when a draft uses one style for every
paragraph. In blinded mode, audit identity leakage; do not delete identifying
content without explicit authorization.

An exact current journal template may override alignment, font, title-page
location, anonymization, or page-number placement. Record the official source,
article type, stage, access date, implementation, and verification. Do not infer
editable-manuscript layout from a published PDF.

## Required workflow

1. Preserve the source. Record path, modification time, SHA-256, and output path.
2. Inventory paragraphs/styles, role paragraph numbers, sections,
   headers/footers, tables, figures, fields, tracked changes/comments, and any
   template. Stop before writing if the runtime cannot safely preserve a feature.
3. Read [references/formatting-contract.md](references/formatting-contract.md).
   For manuscripts, also read
   [references/front-matter-contract.md](references/front-matter-contract.md).
   For cover letters, response letters, or editable package text, instead load
   [references/submission-package-contract.md](references/submission-package-contract.md).
4. Resolve artifact lane, body/non-body styles, line spacing, front-matter roles,
   anonymization state, page-number position, and any official journal override.
5. Edit a distinct copy. Run the applicable manuscript or package profile
   normalizer before the numbering enforcer. Never overwrite the source.
6. Compare extracted text before and after. Any unauthorized text-node change is
   a failure.
7. For a manuscript, run the structural, front-matter, and semantic-rhythm
   audits. For a submission package, run its whole-document package audit. Fix
   and rerun until all applicable audits pass.
8. Render every page after the last layout-sensitive change. Inspect page 1 and
   every remaining page at readable zoom, then rerender after corrections.
9. Combine manuscript gates with `validate_format_release.py` or package gates
   with `validate_submission_package_release.py`. Do not call the work fully
   verified without the corresponding release-pass status.

## Command sequence

Use the workspace document Python runtime when default Python lacks
`python-docx`. Repeat role or style arguments as needed.

```bash
python3 "$SKILL_ROOT/scripts/apply_manuscript_profile.py" input.docx \
  --out normalized.docx --line-spacing double \
  --body-style <body-style> \
  --title-paragraph <n> --authors-paragraph <n> \
  --affiliation-paragraph <n> --correspondence-paragraph <n>

python3 "$SKILL_ROOT/scripts/enforce_docx_line_page_numbers.py" \
  normalized.docx --out release.docx \
  --page-number-position upper-right

python3 "$SKILL_ROOT/scripts/audit_docx_manuscript_style.py" \
  release.docx --paragraph-separation literal-blank \
  --expected-line-spacing <resolved-token> \
  --body-style <each-prose-style> \
  --exclude-style <each-semantic-nonbody-style> \
  --output-json release.structural.json

python3 "$SKILL_ROOT/scripts/audit_docx_front_matter.py" release.docx \
  --mode unblinded --front-matter-alignment left \
  --expected-page-number-position upper-right \
  --expected-line-spacing <resolved-token> \
  --output-json release.front-matter.json

python3 "$SKILL_ROOT/scripts/audit_docx_semantic_rhythm.py" release.docx \
  --expected-line-spacing <resolved-token> \
  --expected-body-font-size 12 \
  --output-json release.semantic-rhythm.json

python3 "$SKILL_ROOT/scripts/validate_format_release.py" \
  release.structural.json release.front-matter.json \
  release.semantic-rhythm.json \
  --content-preservation-status PASS \
  --journal-status NOT_APPLICABLE --render-status PASS \
  --output-json release.format-release.json
```

Accepted line-spacing tokens include `single`, `1.15`, `1.5`, `double`, a
positive multiple, `exact:24pt`, and `at-least:14pt`.

For a non-manuscript artifact, the front-matter gate is not applicable. Report
the package, preservation, journal, and render gates; do not fabricate a
manuscript title block. Use the separate sequence in
[references/submission-package-contract.md](references/submission-package-contract.md).

## Delivery and claim boundary

Deliver a new DOCX, the applicable JSON audit reports, and a concise record of
rules, source-linked overrides, conflicts, content-preservation status, and
visual QA. `FORMAT_RELEASE_PASS` verifies the resolved formatting contract; it
does not verify scientific accuracy, citation validity, editorial acceptance,
or successful submission. For a package file, the equivalent status is
`PACKAGE_FORMAT_RELEASE_PASS`.

## Maintenance boundary

This is a runtime-independent copy of the formatting lane retained in
`manuscript-review-revision`; never import from that sibling at runtime. When a
baseline or semantic-rhythm invariant changes, update and retest both copies.
Front-matter profile logic can evolve here independently and should remain modular.
