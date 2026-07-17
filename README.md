# write-like-me

A Claude Code plugin that learns your writing style from your real writing — GitHub, Jira, Confluence, or local files — and makes Claude write code comments, docstrings, commit messages, PR descriptions, and docs in *your* voice instead of generic AI prose.

It works in two steps: `/learn-style` distills your writing into a cached profile once, and a skill applies that profile automatically on every prose-writing task after that. No profile yet? It falls back to sensible humanizer defaults (no "delve", no bullet-point inflation, varied sentence length).

## Install

In Claude Code:

```
/plugin marketplace add Axel519/write-like-me
/plugin install write-like-me
```

Or from a local checkout:

```
git clone https://github.com/Axel519/write-like-me
claude --plugin-dir ./write-like-me
```

## Setup

Nothing is required up front — run `/learn-style` and it uses whatever sources are available. To widen coverage:

| Source | What it needs |
|--------|---------------|
| **GitHub** | `gh auth login` (uses your PR/issue/review text) |
| **Local git repo** | a folder path — samples your commit messages |
| **Local docs** | a folder path — reads your `.md`/`.txt` |
| **Jira + Confluence** | `export ATLASSIAN_TOKEN=<api-token>` ([create one](https://id.atlassian.com/manage-profile/security/api-tokens)), then pass your site URL once |

## Usage

```
/learn-style                                    # GitHub + anything already configured
/learn-style ~/code/my-project                  # + commit messages and docs from a repo
/learn-style https://yourco.atlassian.net       # + Jira/Confluence (first run saves your site)
```

The first Atlassian run asks for your account email and saves it to `~/.claude/write-like-me/atlassian.json` (site + email only — the token stays in the env var). Re-run any time to refresh; each run overwrites the profile.

Once the profile exists at `~/.claude/write-like-me/profile.md`, just work normally. Ask Claude to write a comment, a PR body, or a commit message and it matches your style — no command needed.

## What it captures

The profile is split by register, because you don't write a commit message the way you write a design doc:

- **General voice** — tone, sentence rhythm, vocabulary and punctuation habits
- **Code comments & commit messages**
- **PR / issue / chat text**
- **Docs & long-form**

Each section has observed rules, verbatim examples from your writing, and anti-patterns (things you never do).

## Scope

Style applies to prose only — never to code logic, identifiers, or APIs. Mandated formats always win: a repo style guide, conventional commits, or a PR template override your personal style.

## License

MIT
