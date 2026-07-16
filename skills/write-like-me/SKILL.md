---
name: write-like-me
description: Use when writing prose on the user's behalf — code comments, docstrings, commit messages, PR descriptions, issue or review text, READMEs, documentation, changelogs, or messages. Applies the user's learned writing style from their profile, or humanizer defaults when no profile exists.
---

# Write like the user

Before writing any prose for the user, do this:

1. Read `~/.claude/write-like-me/profile.md`.
2. **Profile exists** — pick the register section matching the task:
   - code comments, docstrings, commit messages → `## Code comments & commit messages`
   - PR descriptions, issue text, review comments, chat/messages → `## PR / issue / chat text`
   - READMEs, docs, changelogs, any long-form text → `## Docs & long-form`

   Apply `## General voice` plus that section's rules and anti-patterns. Use the examples to hear the voice — never copy an example verbatim into output. If the matching section is missing, use `## General voice` alone.
3. **No profile** — apply the humanizer defaults below, and (once per session) mention that `/learn-style` will personalize the style.

## Scope

- Prose only. Never let style change code logic, identifiers, APIs, or data.
- Mandated formats win: conventional commits, PR/issue templates, a repo style guide, or explicit user instructions always override the personal style.
- Don't announce that a style is being applied — just write that way.

## Humanizer defaults (used when no profile exists)

- Vary sentence length; short sentences are fine.
- Contractions are fine.
- No AI-isms: "delve", "crucial", "seamlessly", "leverage", "robust", "it's important to note", "in today's fast-paced world", rhetorical-question openers.
- Plain verbs over ornate ones: use, not utilize; help, not facilitate.
- Comments say why, not what; one idea each; no restating the code.
- Don't bulletize what reads better as a sentence.
- Em-dashes sparingly; no emoji unless the user uses them first.
