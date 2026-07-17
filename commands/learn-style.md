---
description: Learn your writing style from GitHub, Jira/Confluence, and/or local files, and save it as a reusable profile
argument-hint: "[local folder path] [Jira/Confluence URL]"
---

Build (or rebuild) the user's writing style profile at `~/.claude/write-like-me/profile.md`.

Arguments given: $ARGUMENTS

## 1. Detect available sources

Check each source below and collect the available ones:

- **GitHub** — available if `gh auth status` exits 0.
- **Atlassian (Jira + Confluence)** — available if the `ATLASSIAN_TOKEN` env var is set AND `~/.claude/write-like-me/atlassian.json` exists with `site` and `email` fields.
  - Token set but config missing, and the arguments contain an Atlassian URL (e.g. `https://acme.atlassian.net/browse/PROJ-1`): one-time setup. Take the URL's scheme+host as `site`, ask the user for their Atlassian account email, then verify credentials with `curl -sfS -K - <<< "user = \"EMAIL:$ATLASSIAN_TOKEN\"" "SITE/rest/api/3/myself"`. On success, `mkdir -p ~/.claude/write-like-me` and save `{"site": "...", "email": "..."}` to `~/.claude/write-like-me/atlassian.json`. On failure, show the curl error and skip Atlassian.
  - Token set but no URL ever given: tell the user — "Pass your Jira or Confluence URL once so I can save your site, e.g. `/learn-style https://yourcompany.atlassian.net`" — and continue without Atlassian.
  - Token unset: skip Atlassian silently.
- **Local files / git repo** — available if the arguments contain a folder path that exists. A git repo additionally yields the user's own commit messages.

If **no** source is available, print what to set up and stop:
- GitHub: run `gh auth login`
- Atlassian: export `ATLASSIAN_TOKEN` (API token from https://id.atlassian.com/manage-profile/security/api-tokens), then re-run with your site URL
- Local: `/learn-style ~/path/to/your/docs`

## 2. Fetch writing samples

Target 30–50 samples per register. Skip empty bodies, bodies under 10 words, and anything templated or bot-generated (dependabot, release notes, checklists). Truncate samples longer than ~200 words. If a source errors (401, 403, network), report it briefly, skip it, and continue with the rest.

### GitHub (via gh)

```bash
LOGIN=$(gh api user -q .login)
# PR and issue bodies (register: PR / issue / chat text)
gh search prs --author "$LOGIN" --limit 30 --json title,body
gh search issues --author "$LOGIN" --limit 30 --json title,body
# Recent activity — comments and commit messages (fetch once, reuse)
EVENTS=$(mktemp)
gh api "users/$LOGIN/events?per_page=100" --paginate --slurp > "$EVENTS"
jq '[.[][] | select(.type=="IssueCommentEvent") | .payload.comment.body]' "$EVENTS"
jq '[.[][] | select(.type=="PullRequestReviewCommentEvent") | .payload.comment.body]' "$EVENTS"
jq '[.[][] | select(.type=="PushEvent") | .payload.commits[].message]' "$EVENTS"  # register: Code comments & commit messages
```

If a `gh search` subcommand or flag is unavailable in the installed gh version, fall back to `gh api "search/issues?q=author:$LOGIN+type:pr&per_page=30"` (and `type:issue`).

Each snippet is self-contained — shell state does not persist between tool calls.

### Jira (via curl)

```bash
SITE=$(jq -r .site ~/.claude/write-like-me/atlassian.json)
EMAIL=$(jq -r .email ~/.claude/write-like-me/atlassian.json)
# Issues the user reported — summaries + descriptions
curl -sfS -K - <<< "user = \"$EMAIL:$ATLASSIAN_TOKEN\"" "$SITE/rest/api/3/search/jql?jql=reporter%3DcurrentUser()%20ORDER%20BY%20created%20DESC&maxResults=30&fields=summary,description"
```

If `/rest/api/3/search/jql` returns 404 (older deployments), retry with `/rest/api/3/search?jql=...`. Descriptions and comments arrive as Atlassian Document Format JSON — extract the plain text by collecting all `.text` string values in document order.

The user's comments (same SITE/EMAIL setup as above — re-derive them if this is a fresh shell): get their accountId from `curl -sfS -K - <<< "user = \"$EMAIL:$ATLASSIAN_TOKEN\"" "$SITE/rest/api/3/myself"` (field `.accountId`), then for the ~10 most recent issue keys from the search above run `curl -sfS -K - <<< "user = \"$EMAIL:$ATLASSIAN_TOKEN\"" "$SITE/rest/api/3/issue/KEY/comment"` and keep only comments whose `.author.accountId` matches. Register: PR / issue / chat text.

### Confluence (via curl)

```bash
SITE=$(jq -r .site ~/.claude/write-like-me/atlassian.json)
EMAIL=$(jq -r .email ~/.claude/write-like-me/atlassian.json)
curl -sfS -K - <<< "user = \"$EMAIL:$ATLASSIAN_TOKEN\"" "$SITE/wiki/rest/api/content/search?cql=creator%3DcurrentUser()%20and%20type%3Dpage&limit=20&expand=body.storage"
```

Strip the XML/HTML tags from each `body.storage.value` to get plain text. Register: Docs & long-form.

### Local files / git repo

Read `.md` and `.txt` files from the given folder (recursively, skip anything over 50 files — take the 50 most recently modified). Register: Docs & long-form.

If the folder is a git repo, also sample the user's own commit messages:

```bash
FOLDER=<path>
git -C "$FOLDER" log --no-merges --author="$(git -C "$FOLDER" config user.email)" --format='%B%x00' -n 50
```

Bodies are NUL-separated — split on `\0`. If that yields fewer than 5 (the repo uses a different commit email), re-run without `--author` and keep only messages whose voice plausibly matches. Register: Code comments & commit messages. If every message carries a conventional-commit prefix (`feat:`, `fix:`), that's a mandated format, not personal voice — judge the wording after the prefix.

## 3. Distill the profile

Analyze the corpus for habits you can actually observe: tone and formality, sentence length and rhythm, vocabulary quirks, punctuation habits (em-dashes? ellipses? exclamation points?), capitalization, emoji usage, language(s) used, how they open and close messages, how they explain vs. instruct. Every rule must be grounded in the samples; every example must be a verbatim quote from the corpus.

Overwrite `~/.claude/write-like-me/profile.md` using exactly this structure (omit a register section only if it got zero samples):

```markdown
# Writing style profile

Generated: <today's date> — Sources: <source: N samples, ...>

## General voice

<5-10 bullet rules covering tone, rhythm, vocabulary, punctuation, language>

## Code comments & commit messages

Rules:
<3-6 bullets of observed habits specific to this register>

Examples:
<3-5 short verbatim quotes>

Anti-patterns (never does):
<2-4 bullets>

## PR / issue / chat text

Rules:
<3-6 bullets>

Examples:
<3-5 short verbatim quotes>

Anti-patterns (never does):
<2-4 bullets>

## Docs & long-form

Rules:
<3-6 bullets>

Examples:
<3-5 short verbatim quotes>

Anti-patterns (never does):
<2-4 bullets>
```

## 4. Report

Print a short summary: sources used with sample counts per register, sources skipped and why, the profile path, and a note that prose-writing tasks will now use the profile automatically (via the write-like-me skill).
