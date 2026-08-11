# Manuscript DOCX Formatting Skill

[Chinese](README.md)

An independent Codex / Agent Skill that converts scientific and biomedical DOCX drafts into restrained, natural, author-prepared submission manuscripts. It performs format-only repair and does not automatically change scientific claims, citations, or article structure.

[![Validate skill](https://github.com/Jameslxr/manuscript-docx-formatting/actions/workflows/validate.yml/badge.svg)](https://github.com/Jameslxr/manuscript-docx-formatting/actions/workflows/validate.yml)
![Version](https://img.shields.io/badge/version-v1.0.1-2563eb)
[![License: MIT](https://img.shields.io/badge/license-MIT-2ea44f)](LICENSE)

## Features

- Enforces effective `0 pt` paragraph spacing and removes hidden Word auto-spacing.
- Inserts exactly one real empty Word paragraph between adjacent body paragraphs instead of simulating a blank line with paragraph spacing.
- Uses explicit line spacing, defaulting to double spacing when no current source specifies another value.
- Applies the same resolved line spacing to authors, affiliations,
  correspondence, Keywords, headings/subheadings, body text, and CRediT or
  declaration sections; these roles default to the 12 pt body size.
- Bolds only recognized `Keywords:` and inline declaration labels; requires one
  real empty paragraph after Keywords and before every new section, subsection,
  or declaration block, with none between a heading and its first body paragraph.
- Keeps entries after `References/Bibliography` in a dedicated non-body role and
  never inserts body-prose blank separators between reference entries.
- Adds continuous Word-native line numbering and dynamic `PAGE` fields.
- Normalizes title, authors, affiliations, and correspondence into restrained left-aligned manuscript front matter unless an exact current journal source overrides it.
- Supports journal-, article-type-, and submission-stage-specific official formatting overrides.
- Separately verifies structure, front matter, semantic vertical rhythm, content preservation, journal rules, and every rendered page; only a complete pass returns `FORMAT_RELEASE_PASS`.

## Installation

Ask Codex to install the public Skill:

```text
Use $skill-installer to install this Skill:
https://github.com/Jameslxr/manuscript-docx-formatting/tree/main/manuscript-docx-formatting
```

Or run:

```bash
python3 ~/.codex/skills/.system/skill-installer/scripts/install-skill-from-github.py \
  --repo Jameslxr/manuscript-docx-formatting \
  --path manuscript-docx-formatting
```

For Claude Code or another compatible Agent Skills host:

```bash
git clone https://github.com/Jameslxr/manuscript-docx-formatting.git
cp -R manuscript-docx-formatting/manuscript-docx-formatting \
  "$HOME/.claude/skills/manuscript-docx-formatting"
python3 -m pip install -r manuscript-docx-formatting/requirements.txt
```

## Usage

```text
Use $manuscript-docx-formatting to convert this DOCX into a natural submission-style manuscript. Preserve all wording, citations, figures, tables, and section order. Deliver a new DOCX and every applicable format-audit result.
```

For journal-specific formatting:

```text
Use $manuscript-docx-formatting.
Target journal: [exact journal]
Article type: [article type]
Submission stage: [initial / revision / final]
Inspect the current official requirements, apply them, and report each source, implementation, and verification result.
```

## Boundary

The Skill preserves the source and writes a distinct output. It does not perform scientific peer review, evidence checking, language revision, or citation-integrity review; use `manuscript-review-revision` for those tasks. `FORMAT_RELEASE_PASS` verifies the resolved DOCX formatting contract only. It does not establish scientific correctness, editorial acceptance, or successful submission.

## Validation

```bash
python3 -m pip install -r requirements.txt
python3 -m py_compile manuscript-docx-formatting/scripts/*.py
python3 -m unittest discover -s manuscript-docx-formatting/tests -v
```

The current 17-test regression suite adds coverage for author/affiliation size, global double and 1.5 line spacing, Keywords labels, section/subsection boundaries, and missing or duplicated Author Contributions/CRediT blanks. It also retains coverage for literal blank paragraphs, hidden auto-spacing, line/page numbering, adversarial title pages, idempotent repair, text preservation, and fail-closed release composition.

## License

[MIT](LICENSE)
