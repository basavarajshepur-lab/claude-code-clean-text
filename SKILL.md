---
name: clean-text
description: Strip AI-generated and typographic special characters from markdown or text documents, replacing them with conventional ASCII punctuation. Covers em dashes, en dashes, arrows, middots, smart quotes, ellipses, bullets, and other non-standard glyphs.
disable-model-invocation: false
user-invocable: true
---

# Clean Text: Strip AI-Generated Characters

**When to use:** Any time a document contains em dashes, en dashes, curly quotes, arrow glyphs, middots, or other typographic/Unicode characters that should be plain ASCII. Common after AI drafting, copy-paste from Word or Google Docs, or importing from design tools.

**Invoke as:** `/clean-text` then provide the file path, or paste text directly.

---

## What Gets Cleaned

### Tier 1 - Always Replace (AI Signature Characters)

| Character | Unicode | Name | Replacement Rule |
|-----------|---------|------|-----------------|
| — | U+2014 | Em dash | Context-aware: `, ` mid-sentence, `: ` before a list, `. ` between clauses, `( )` for parentheticals |
| – | U+2013 | En dash | `-` in number/date ranges (e.g. "6-12 months"); `, ` elsewhere |
| → | U+2192 | Right arrow | `->` in code/diagrams; "to" in prose |
| · | U+00B7 | Middle dot / interpunct | ` \| ` as separator |
| … | U+2026 | Ellipsis | `...` |
| • | U+2022 | Bullet | `-` (only outside markdown lists) |

### Tier 2 - Always Replace (Smart Punctuation)

| Character | Unicode | Name | Replacement |
|-----------|---------|------|-------------|
| " " | U+201C/D | Curly double quotes | `"` straight double quotes |
| ' ' | U+2018/9 | Curly single quotes / apostrophes | `'` straight apostrophe |
| ‐ | U+2010 | Non-breaking hyphen | `-` |
|   | U+00A0 | Non-breaking space | regular space |

### Tier 3 - Context-Aware (Flag and Suggest)

| Character | Unicode | Name | Action |
|-----------|---------|------|--------|
| × | U+00D7 | Multiplication sign | `x` (unless in mathematical formulas) |
| ≥ ≤ | U+2265/4 | Greater/less-than or equal | Keep in technical docs; flag in prose |
| ™ ® © | various | Trademark symbols | Keep; flag for review |
| § | U+00A7 | Section symbol | `Section` in legal; `#` in markdown |

---

## How to Use

### Mode 1: Clean a File

```
/clean-text

File: outputs/prds/CrewVault_PRD_v1.0.md
```

I will:
1. Read the file
2. Apply all Tier 1 and Tier 2 replacements
3. Apply context-aware em dash logic (not a blunt replace-all)
4. Save the cleaned file in place
5. Print a replacement report (character, count, what it became)
6. If a `generate_docx.py` exists in the same folder, offer to regenerate the Word doc

### Mode 2: Clean Pasted Text

```
/clean-text

[paste any text here]
```

I will return the cleaned text inline so you can copy it back.

### Mode 3: Clean and Export

```
/clean-text export

File: outputs/prds/CrewVault_PRD_v1.0.md
```

Cleans the markdown file and immediately regenerates the Word doc if `generate_docx.py` is present in the same directory.

### Mode 4: Audit Only (No Changes)

```
/clean-text audit

File: outputs/prds/CrewVault_PRD_v1.0.md
```

Reports every special character found and its proposed replacement, but makes no changes. Use this to preview before committing.

---

## Execution Steps

When invoked, follow this exact sequence:

### Step 1: Identify Input

- If a file path is given, read it with the Read tool (markdown) or extract text via PowerShell (docx)
- If text is pasted, work on that directly
- If neither, ask: "Which file should I clean? Provide the full path, or paste the text."

### Step 2: Audit Pass

Count all Tier 1, Tier 2, and Tier 3 characters before making any change. Report:

```
Audit complete — characters found:
  Em dashes (—):        [count]
  En dashes (–):        [count]
  Arrows (→):           [count]
  Middots (·):          [count]
  Ellipses (…):         [count]
  Curly double quotes:  [count]
  Curly single quotes:  [count]
  Non-breaking spaces:  [count]
  Other:                [list any Tier 3 found]
```

If mode is `audit`, stop here and show the report with proposed replacements. Do not write anything.

### Step 3: Apply Replacements (Em Dash Logic)

Em dashes need context-aware replacement, not a blunt global swap. Apply in this order:

1. ` — [Capital letter]` (em dash between two independent clauses) → `. [Capital letter]`
2. `[word 5+ chars] — [lowercase]` (em dash introducing a list or appositive) → `[word]: [lowercase]`
3. ` — ` with the same subject on both sides (parenthetical aside) → ` (` ... `)` — only if the aside is short (<8 words); otherwise `, `
4. Any remaining ` — ` → `, `
5. Any bare `—` with no surrounding spaces → `-`

En dash rule:
- `[digit]–[digit]` → `[digit]-[digit]`
- All other `–` → `, `

All other characters use the flat replacement table above.

### Step 4: Spot-Fix Pass

After automated replacement, check for artifacts:
- Double colons `::` → `: ` (can occur when em dash preceded an existing colon)
- Double commas `,,` → `,`
- Period + lowercase at start of continuation (`. we`) → `. We` (capitalise)
- `: :` → `:`
- Sentence-start apostrophe artefacts

### Step 5: Write and Report

- Write the cleaned content back to the same file (in place)
- Print the replacement report:

```
Replacement report:
  Em dashes:      [n] replaced ([breakdown of which rule fired])
  En dashes:      [n] replaced
  Arrows:         [n] replaced
  Middots:        [n] replaced
  Smart quotes:   [n] replaced
  [other]:        [n] replaced
  ─────────────────────────────
  Total:          [n] characters cleaned

  Spot-fixes applied: [list any double-colon, capitalisation fixes, etc.]

  File saved: [path]
```

### Step 6: Word Doc (If Applicable)

If the file directory contains a `generate_docx.py` script:
- In `export` mode: run it automatically via `python generate_docx.py`
- In default mode: ask "A Word doc generator was found. Regenerate the .docx now? (yes/no)"

---

## Context Routing

When `/clean-text` is invoked, check:

1. **File type:** If `.md`, use Read + Edit tools. If `.docx`, use PowerShell to extract and re-inject text (only if no markdown source exists). If `.txt`, use Read + Write.
2. **Companion Word doc:** If a `generate_docx.py` is in the same directory, offer to regenerate after cleaning.
3. **CLAUDE.md voice rules:** After cleaning, verify the document still conforms to the voice rules in CLAUDE.md (no em dashes remaining, no forbidden words introduced by the replacement).
4. **Multiple files:** If the user says "clean all PRDs" or "clean the whole outputs folder", glob for `.md` files and process each one, reporting a combined summary.

---

## Character Reference Card

Quick lookup for manual fixes or edge cases:

```
Em dash      —   U+2014   → comma / colon / period / parens
En dash      –   U+2013   → hyphen (ranges) or comma
Figure dash  ‒   U+2012   → hyphen
Hyphen       ‐   U+2010   → hyphen
Minus sign   −   U+2212   → hyphen or minus
Right arrow  →   U+2192   → -> or "to"
Left arrow   ←   U+2190   → <- or "from"
Up arrow     ↑   U+2191   → "above" or "increase"
Down arrow   ↓   U+2193   → "below" or "decrease"
Middot       ·   U+00B7   → | (separator) or , (list)
Bullet       •   U+2022   → - (list item)
Ellipsis     …   U+2026   → ...
L dquote     "   U+201C   → "
R dquote     "   U+201D   → "
L squote     '   U+2018   → '
R squote     '   U+2019   → '
Apostrophe   ʼ   U+02BC   → '
NBSP             U+00A0   → space
Thin space       U+2009   → space
Hair space       U+200A   → space (or remove)
Zero-width   ​   U+200B   → remove
Word joiner  ⁠   U+2060   → remove
Section      §   U+00A7   → "Section" or "#"
Multiplication × U+00D7  → x
```

---

## Output Quality Self-Check

Before reporting done, verify:

- [ ] Zero em dashes (—) remaining in the file
- [ ] Zero en dashes (–) remaining in the file
- [ ] Zero curly quotes remaining
- [ ] No double-colon artefacts (`::`)
- [ ] No double-comma artefacts (`,,`)
- [ ] Document reads naturally — no replacement created awkward phrasing
- [ ] If a Word doc was regenerated, file size is within 20% of the pre-clean version (sanity check for corruption)
- [ ] Tier 3 characters flagged for human review, not silently changed

---

## Related Skills

- `/prd-draft` — creates PRDs that should be run through `/clean-text` before sharing
- `/meeting-notes` — meeting notes often contain pasted text with smart quotes
- `/status-update` — executive updates where clean punctuation matters for credibility
