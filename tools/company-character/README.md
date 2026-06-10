# company-character

A system for giving your organisation a terminal voice. Point it at your GitHub org and it builds a CLI companion that knows your tools, your status, and where you're headed — and speaks in your brand's voice.

Works offline by default. Optionally powered by Claude for natural language responses. No database. No server. Just a compiled knowledge file that you control.

---

## How it works

Three components work together:

```
company-state/     content and config — what your assistant knows and how it speaks
      +
GitHub org         your repos, scanned via API
      ↓
cc-admin           internal tool — compiles everything into dist/knowledge.json
      ↓
company-cli        public CLI — ships to users with the compiled knowledge inside
```

**`company-state`** is a folder of plain files: section mappings, personality, NOW.md, roadmap. You edit these directly or via `cc-admin`.

**`cc-admin`** is your private tool. It scans your GitHub org, parses READMEs according to your section map, applies overrides, and compiles a `dist/knowledge.json`. It also manages Q&A pairs, lore, announcements, aliases, and easter eggs.

**`company-cli`** is what users install. It reads the compiled knowledge and responds as your brand. Rename the entry point in `pyproject.toml` to match your brand (e.g. `heybabb`, `hey-acme`, `askwidget`).

---

## Quick start

```bash
cd cc-admin
python3 -m venv .venv && source .venv/bin/activate
pip install -e .

# Scaffold your company-state
cc-admin init
```

`cc-admin init` prompts for your org name, brand name, and topic prefix, then writes all the `company-state` files and updates `.cc-admin.yml`.

---

## GitHub topics

Tag repos in your GitHub org so the scanner knows what to include:

| Topic | What it means |
|---|---|
| `{prefix}-tool` | A product or tool — full deep scan |
| `{prefix}-featured` | Lead with this when listing |
| `{prefix}-vision` | Strategic work, not a product yet |
| `{prefix}-internal` | Infrastructure — brief mention only |

Replace `{prefix}` with your org prefix (set during `cc-admin init`).

---

## README conventions

Each opted-in repo should have these section headers (configurable in `company-state/section-map.yml`):

```markdown
## What is it?
## The Problem
## How it Works
## Current Status
## The Vision
## Industry Context
```

The scanner parses these into structured knowledge slots. Any header can be renamed or removed — edit `section-map.yml` or run `cc-admin repo section-map --add`.

---

## cc-admin commands

```
cc-admin init                    scaffold a new company-state/
cc-admin scan                    scan GitHub org and parse repos
cc-admin build                   compile dist/knowledge.json
cc-admin diff                    show what changed before publishing
cc-admin publish                 push dist/ to company-cli
cc-admin preview "query"         simulate a CLI response before publishing
cc-admin audit                   show build log

cc-admin qa list|add|edit|delete|test|import|export
cc-admin override list|set|suppress|embargo|tone|clear
cc-admin lore list|add|edit|delete|preview
cc-admin announce list|set|clear|expire
cc-admin alias list|add|remove|xref
cc-admin persona show|edit|easter-egg
cc-admin repo list|ignore|pin|section-map
```

---

## company-cli commands

```
companycli                       greeting
companycli tools                 list all indexed tools
companycli tool <name>           full breakdown of a specific tool
companycli now                   what the team is working on
companycli vision                roadmap and direction
companycli version               build info
companycli ask "<question>"      natural language, offline
companycli ask --ai "<question>" natural language, Claude-powered
companycli sync                  refresh GitHub signals
```

Rename `companycli` in `company-cli/pyproject.toml` to your brand name.

---

## AI mode

`ask --ai` uses the Claude API. Set your key:

```bash
export ANTHROPIC_API_KEY=your_key_here
companycli ask --ai "what makes your approach different?"
```

The entire compiled knowledge base is passed as context. Your assistant only speaks from what's in it.

---

## Overrides

`cc-admin override` lets you curate what the CLI says about any tool without touching the repo:

```bash
cc-admin override set bitpads status "Stable. Ships next week."
cc-admin override suppress bitpads context
cc-admin override embargo bitpads vision --until 2026-09-01
cc-admin override tone bitpads technical
```

Override types: `replace`, `suppress`, `embargo`, `confidence` (marks a slot as speculative), `tone`.

---

## Knowledge enrichment

Beyond what's in READMEs:

- **Q&A pairs** (`cc-admin qa`) — hand-craft answers to anticipated questions
- **Lore** (`cc-admin lore`) — inject company history, mission, culture
- **Announcements** (`cc-admin announce`) — pin a statement to the `now` command
- **Aliases** (`cc-admin alias`) — alternate names and cross-references between tools
- **Easter eggs** (`cc-admin persona easter-egg`) — trigger personality moments on specific inputs

---

## Open source

company-character is a reference implementation. Fork it, rename it, and build your own branded CLI companion. The architecture is intentionally generic — no hardcoded org names, no hardcoded brand text. Everything flows from config and `company-state`.

Built by [Babb Works](https://babb.works). The Babb CLI (`heybabb`) is the reference deployment.
