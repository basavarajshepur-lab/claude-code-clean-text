# clean-text — Claude Code Skill

A Claude Code slash command that strips AI-generated and typographic special characters from any markdown or text document, replacing them with clean, conventional ASCII punctuation.

Invoke it as `/clean-text` from anywhere in Claude Code.

---

## The Problem

AI writing tools and copy-paste from Word, Google Docs, or Notion routinely smuggle in Unicode characters that look fine on screen but break plain-text pipelines, AI detectors, linters, and version control diffs:

- Em dashes `—` instead of commas or colons
- En dashes `–` in date ranges instead of hyphens
- Curly quotes `"` `"` instead of straight quotes
- Arrow glyphs `->` or middots `·` as decorative separators
- Non-breaking spaces that look like regular spaces
- Ellipsis glyphs `...` instead of three dots

This skill finds and replaces all of them in one command.

---

## Installation

1. Copy `SKILL.md` into your Claude Code project at:

```
.claude/skills/clean-text/SKILL.md
```

2. That is it. Claude Code auto-discovers skills in that directory. The `/clean-text` command is immediately available in your session.

---

## Usage

### Clean a file in place

```
/clean-text

File: outputs/prds/my-document.md
```

### Audit without changing anything

```
/clean-text audit

File: outputs/prds/my-document.md
```

Reports every special character found and its proposed replacement. Makes no changes.

### Clean and regenerate a Word doc

```
/clean-text export

File: outputs/prds/my-document.md
```

Cleans the markdown, then runs `generate_docx.py` in the same folder if one exists.

### Clean pasted text

```
/clean-text

Today's meeting — a quick sync — confirmed the 6–12 month timeline...
```

Returns the cleaned text inline.

---

## What Gets Cleaned

### Tier 1 — AI Signature Characters (always replaced)

| Character | Name | Replaced with |
|-----------|------|--------------|
| `—` | Em dash | `,` `:` `.` or `( )` depending on context |
| `–` | En dash | `-` in number/date ranges; `,` elsewhere |
| `->` | Right arrow | `->` in code; `to` in prose |
| `·` | Middle dot | ` \| ` |
| `...` | Ellipsis glyph | `...` (three literal dots) |
| `•` | Bullet | `-` outside markdown lists |

### Tier 2 — Smart Punctuation (always replaced)

| Character | Name | Replaced with |
|-----------|------|--------------|
| `"` `"` | Curly double quotes | `"` straight double quotes |
| `'` `'` | Curly single quotes | `'` straight apostrophe |
| Non-breaking space | U+00A0 | Regular space |
| Non-breaking hyphen | U+2010 | `-` |

### Tier 3 — Context-Aware (flagged for review, not silently changed)

`x` `>=` `<=` `(tm)` `(r)` `(c)` `Section symbol`

---

## Em Dash Intelligence

The skill does not do a blunt global replace on em dashes. It applies context rules in order:

1. `word — Capital` (two independent clauses) - replaced with `. Capital`
2. `longword — lowercase` (introducing a list or appositive) - replaced with `longword: lowercase`
3. ` — short aside — ` (parenthetical under 8 words) - replaced with ` (aside)`
4. Any remaining ` — ` - replaced with `, `
5. Bare `—` with no surrounding spaces - replaced with `-`

This prevents the double-colon artefacts (`::`) and awkward phrasing that naive replacements produce.

---

## Output

After cleaning, the skill prints a full replacement report:

```
Replacement report:
  Em dashes:      12 replaced
  En dashes:      8 replaced
  Arrows:         2 replaced
  Smart quotes:   6 replaced
  Total:          28 characters cleaned

  Spot-fixes: 2 double-colon artefacts corrected, 1 sentence capitalised

  File saved: outputs/prds/my-document.md
```

---

## Works With

- `.md` markdown files (primary use case)
- `.txt` plain text files
- Pasted text (inline mode)
- Companion Word doc regeneration (if `generate_docx.py` is present)

---

## Related Skills

Pair this with other Claude Code skills:

- After `/prd-draft` — clean the output before sharing
- After `/meeting-notes` — clean pasted transcript artefacts
- After `/status-update` — ensure executive comms are clean

---

## Contributing

Found a character this skill misses? Open an issue or pull request with:

1. The Unicode character (name and code point)
2. Where you encountered it (AI tool, app, OS)
3. What it should be replaced with

---

## License

MIT
