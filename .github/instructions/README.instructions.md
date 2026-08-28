# AI Coding Rules - Contribution Guide

AI-optimized coding rules for AL development. One markdown file per category (see [_index.instructions.md](_index.instructions.md) for the list).

## Adding rules

1. Pick the matching category file, or create a new one.
2. Keep the format terse — instruction files are loaded into the AI context, so every line costs tokens:

```markdown
---
description: Short description
applyTo: "*.al"        # omit to make the file documentation-only
---

# AL <Category> Rules

## Rule N: <Title>
- Imperative bullet points, one rule per line.
- Add a code snippet only when the rule cannot be expressed in words.
```

3. Update `al-guidelines-rules.instructions.md` (`@file` reference) and `_index.instructions.md`.
4. Test with an AI assistant before submitting a PR.

## Writing guidelines

- Prefer imperative bullets over prose; drop "Intent" paragraphs.
- Include a bad example only when the mistake is common and non-obvious.
- Never duplicate content across files — cross-reference instead.
- Only files with an `applyTo` pattern are auto-loaded; keep pure documentation without it.
