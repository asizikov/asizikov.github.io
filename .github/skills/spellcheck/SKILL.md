---
name: spellcheck
description: Fix English spelling, grammar, and punctuation in Markdown while preserving tone and formatting. Use when the user asks to spellcheck, proofread, or correct language mistakes in a document or changed Markdown files.
---

# Spellcheck a Markdown document

Find the relevant Markdown document in the supplied context or current changes. If the target is ambiguous, ask the user which document to check.

Check the document for English spelling, grammar, and punctuation errors and fix them.

Only change wording as needed to correct these mistakes. Do not otherwise rewrite or rephrase text, or change formatting. The tone and language must remain the same.

Never change any code blocks. Never change any front matter metadata except to fix spelling mistakes there.
Never rename tags or change dates.

Provide a summary of the changes you made.