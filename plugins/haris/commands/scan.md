---
description: Scan text or a file for prompt injection and Saudi PII (Arabic, Arabizi, English)
argument-hint: <text to scan, or a file path>
allowed-tools: mcp__haris__scan_text, Read, Glob
---

Scan what follows with Haris and report what it found: `$ARGUMENTS`

How to do it:

1. If the argument looks like a path that exists, read the file and scan its contents.
   Otherwise treat the argument itself as the text to scan. If the argument is empty, ask
   what to scan rather than guessing.
2. Call `mcp__haris__scan_text` on it. For a file over ~20,000 characters, scan it in
   sections and report per section.
3. Report, in this order:
   - the verdict and score, and the threshold it was compared against;
   - each finding: its category, the signature or detector that matched, and the offsets;
   - for PII, the **redacted** label only (`[IQAMA]`, `[IBAN]`), **never** the identifier
     itself, even though you can see it in the file you read.
4. Say plainly what Haris does **not** tell you: it flags, it does not block, and a clean
   verdict is not proof that text is safe. The rules layer only knows the phrasings it has
   signatures for.

If the scan comes back clean, say so in one line. Do not pad it.
