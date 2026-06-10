# newent — CLI Design

> Command grammar, UX patterns, addressing, and interaction design.
> Read alongside ARCHITECTURE.md.
> Last revised: 2026-04-19

---

## 1. Grammar Overview

newent uses a **subject-first** grammar. The business name is always the first positional argument in common use. System verbs come before the business name when explicitly invoked.

```
newent <business-name>                              # subject-first, common path
newent <business-name> <section>                   # navigate into section
newent <business-name> <section> <document>        # navigate into document
newent <business-name> <address>                   # navigate by address
newent <verb> <business-name>                      # explicit verb form
newent <business-name> <section> new               # create new in context
newent <business-name> <section> ent               # contextual view
newent <business-name> browser                     # open browser at business
```

The two entry forms for a business are equivalent:
```
newent acme-corp          ← preferred for typing
newent name acme-corp     ← explicit form, useful in scripts
```

---

## 2. Reserved Words

The following words cannot be used as business names. Checked at creation time only.

```
System verbs:   doctor  coach  warmup  warmups  develop  document  export
                status  migrate  update  browser
Utilities:      notes  finance  config  init  help  version  spaces  cloud
Navigation:     name  new  ent  list  plan  framework  archive
```

**On collision:**
```
$ newent new finance
⚠  "finance" is a reserved system word and cannot be used as a business name.
   Suggestions: finance-co  my-finance  financeapp
   Or enter a different name: _
```

Reserved word check uses exact match on the first positional argument only, after stripping hyphens and lowercasing.

---

## 3. Business Name Rules

- Lowercase, hyphens allowed, no spaces, no leading/trailing hyphens
- 2–60 characters
- Alphanumeric plus hyphens only
- Must be unique within the workspace
- Displayed as-is; no automatic slugging of user input

---

## 4. Single-Item Selection UX

When a navigation context has only one option, it is selected automatically. The user sees a brief confirmation line and presses Enter to proceed:

```
→ acme-corp / Startup Plan  [1 plan · press Enter to continue, or type "new" / "variant"]
```

If the user types `new`, the New service is invoked to create a sibling. If the user types `variant`, a variant of the single item is created. Hint data can surface a simple statistic inline: e.g., `[1 plan · 0 variants · press Enter to continue]`.

When multiple options exist, the selection list is shown with relative numbers. The user enters a number, `new`, or `variant`.

---

## 5. Plan Selection UX

When a business has more than one plan, any plan-context command triggers selection:

```
$ newent acme-corp problems

acme-corp — 3 plans:
  [1] Startup Plan          active · edited 2 days ago · 14 variants
  [2] Operations Q2 2026    draft · created Apr 10 · 0 variants
  [3] Series A Prep         draft · created Apr 18 · 2 variants
  [n] New standalone plan
  [v] New variant (fork) of an existing plan
→ _
```

Bypass with `--plan <id>`:
```
newent acme-corp problems --plan 2
```

Plan variants (forks) are full plan copies — they share no content with the source plan after forking. Variant metadata records the source plan and fork date.

---

## 6. Content Addressing

Every piece of content has a five-level address within a plan:

| Level | Identifier | Example |
|---|---|---|
| Section | Integer | `3` |
| Document | Uppercase letter | `B` |
| Heading | Integer | `2` |
| Content item | Lowercase letter | `a` |
| Context item | Roman numeral | `i` |

Full address: `3.B.2.a.i` — "Section 3, Document B, Heading 2, Content item a, Context item i."

**Periods are optional.** `3B2ai` and `3.B.2.a.i` are equivalent. Periods are encouraged for readability but never required.

**CLI use:**
```
newent acme-corp 3              # section 3
newent acme-corp 3B             # document B in section 3
newent acme-corp 3B2            # heading 2
newent acme-corp 3B2a           # content item a
newent acme-corp 3B2ai          # context item i
```

**Content item and context item IDs are positional display labels, not permanent identifiers.** The internal UUID tracks identity; the letter/numeral reflects current display position. If item `b` is deleted, item `c` becomes `b`. This matches user intuition — the address reflects where something is, not when it was created. Sections, Documents, and Headings use stable addresses (assigned at creation) because resequencing them is an explicit reorder operation.

### 6.1 Listing

```
newent acme-corp 3B --list          # list all headings in document 3.B
newent acme-corp 3 --list           # list all documents in section 3
newent acme-corp 3B2a --list        # list context items for content item a
```

### 6.2 Variant Listing

Any entity that supports variants can display a variant list with relative numbers:

```
$ newent acme-corp 3B --variants

Document 3.B — "Target Market Definition"  (3 variants)
  [1] Original  ← active · edited Apr 15
  [2] Variant   created Apr 17
  [3] Variant   created Apr 18
  Actions: [r] replace active  [e] edit  [a] archive  [d] delete
→ _
```

Relative numbers are display-only. The user selects by number and then chooses an action. Listing variants is available for: Plans, Documents, Headings, ContentItems, ContextItems.

---

## 7. Section Names as Aliases

Section slugs can be used interchangeably with section addresses:

```
newent acme-corp problems           # same as section 3 (if problems = 3)
newent acme-corp market             # same as section 5
```

Section slug aliases are defined by the industry framework. The `generic` framework uses: `executive-summary`, `problem`, `solution`, `market`, `business-model`, `go-to-market`, `competition`, `team`, `financials`, `operations`, `risk`, `appendix`.

---

## 8. The `new` Command

`new` is a first-class command and microservice. It resolves contextually to create the appropriate thing at the current navigation level:

```
newent new                              → new business (interactive)
newent acme-corp new                    → new plan for acme-corp
newent acme-corp problems new           → new document in problems section
newent acme-corp 3B new                 → new heading in document 3.B
newent acme-corp 3B2 new                → new content item
newent acme-corp 3B2a new               → new context item for content item a
```

### 8.1 Inline Creation Syntax

For any ordered set (notes, content items, context items), the user can issue a combined reorder + create sequence by typing space-separated relative numbers with `new` inserted at insertion points:

```
→ Reorder (and optionally add): 1 3 2 new 4 5 new
```

This means: position 1 stays, then position 3, then position 2, then create a new item here, then position 4, then position 5, then create another new item. The New service processes the sequence and opens inline Q&A (via QA service) for each `new` trigger.

Items created mid-sequence are immediately assigned a display label for the rest of the sequence. The user does not need to separately reorder afterward.

---

## 9. The `ent` Command

`ent` generates a situated, contextual view of any plan element. It is a first-class service that grows richer over time.

```
newent acme-corp ent                    → company context view (profile + plan summary)
newent acme-corp problems ent           → problems section in plan context
newent acme-corp 3B ent                 → document with section context + related notes
newent acme-corp 3B2 ent                → heading with full surrounding context
```

The `ent` view includes: the element's content, its parent hierarchy, sibling elements at the same level, assigned notes (as prologue if applicable), and a brief "how this fits" summary. Future AI-enhanced `ent` will add narrative coherence analysis and audience lens overlay.

---

## 10. The `warmups` Command

`warmups` (plural) is the Warmup landing experience — a browseable place to hang out and explore question sets without committing to a formal session:

```
newent warmups                          → warmup home (all sets, recent sessions)
newent warmups acme-corp                → warmup home for acme-corp
newent warmup acme-corp                 → start or resume a warmup session
newent warmups list                     → list all available question sets
newent warmups history acme-corp        → completed sessions for acme-corp
```

Warmup home shows: available question sets (with tags, estimated time, count), in-progress sessions, completed sessions with their summary notes, and suggestions based on plan state.

---

## 11. Browser Access

```
newent browser                          → workspace-level browser
newent acme-corp browser                → business browser view
newent acme-corp problems browser       → browser at problems section
newent acme-corp 3B2 browser            → browser at a specific heading
```

Browser deferred. Data model supports it from the start.

---

## 12. Output Modes

**Default (human):** Formatted output with color and layout. Uses `rich` for tables, panels, progress indicators. Color conveys meaning: green=good, yellow=warning, red=error/missing.

**JSON:** `--json` on all commands. Machine-readable, includes `schema_version`. No color or formatting. Suitable for piping to `jq`, scripts, AI agents.

**Quiet:** `--quiet` / `-q`. Suppresses informational output; shows only errors and explicit results.

**NO_COLOR:** Respected when `NO_COLOR` env var is set or when output is not a TTY.

---

## 13. Piping and External Input

```bash
cat research-notes.md | newent notes acme-corp add --stdin
cat assumptions.csv | newent finance acme-corp import --stdin --format csv
cat interview-transcript.txt | newent warmup acme-corp load --stdin
```

When stdin is detected and a command supports it, interactive prompts are suppressed. Unrecognized file types return a clear error listing supported formats.

---

## 14. Deletion and Archive Flow

Deletion is always two-step. Content is never permanently deleted in the first pass — it goes to archive.

1. User issues delete command (e.g., `newent acme-corp 3B delete`)
2. System shows exactly what will be affected
3. Prompt: `Type "yes" then press Enter to move to archive: _`
4. User types `y` or `yes`
5. User presses Enter
6. Content moved to plan archive

`--force` skips the confirmation for scripted use but still moves to archive (not permanent delete).

**Archive access:**
```
newent acme-corp archive                → view archived items for acme-corp's active plan
newent acme-corp archive --plan 2       → view archive for a specific plan
newent acme-corp archive restore 3B     → restore an archived item
newent acme-corp archive clear          → permanently delete all archived items
```

`archive clear` requires the same two-step confirmation. It is the only command that permanently destroys content.

---

## 15. Note Display and Reordering

Notes are listed for any element as a relatively numbered list:

```
Notes for: Section 3 — Problem
  [1] NOTE-0042  "Market gap observation"  · Apr 12 · active
  [2] NOTE-0051  "Customer interview summary"  · Apr 15 · warmup
  [3] NOTE-0067  (untitled)  · Apr 18 · manual

Options: [a] add note  [t] toggle display  [r] reorder  [n] new
→ _
```

**Reordering:** User types `r`, then enters a space-separated number sequence: `2 3 1` to reorder. Can include `new` to insert new notes inline: `1 new 2 3`.

**Display toggle:** Notes at plan, section, or document level render as prologues in views and exports. Toggle is per-element (default: on) or global (`newent config notes-display off`).

**Titles:** Notes can have optional titles set on creation or via edit. Untitled notes are listed as "(untitled)".

---

## 16. Interactive Prompts

1. Missing required arguments trigger guided prompts, not bare errors.
2. Prompts include available options with clear labels.
3. Numeric selection is always available; partial name matching also works.
4. `--dry-run` is available on all mutating commands.

Prompts are suppressed when: `--json` is active, stdin is not a TTY, or `--no-interactive` is passed. In non-interactive mode, missing required arguments return a structured error.

---

## 17. Error Message Standards

Pattern: `✗ [what happened] — [why] — [what to do]`

```
✗ Business "finance" not found — no matching business in this workspace
  Run `newent list` to see all businesses, or `newent new` to create one.

✗ "doctor" is a reserved word — cannot be used as a business name
  Try "doctor-app", "my-doctor", or another name.

✗ Plan selection required — acme-corp has 3 plans
  Use `--plan <id>` or run `newent acme-corp` to choose interactively.
```

Errors always exit with a non-zero code.

---

## 18. Global Flags

| Flag | Short | Description |
|---|---|---|
| `--json` | | Machine-readable JSON output |
| `--quiet` | `-q` | Suppress informational output |
| `--dry-run` | | Show what would happen without doing it |
| `--force` | | Skip interactive confirmations (still archives, never hard-deletes) |
| `--no-interactive` | | Fail on missing args instead of prompting |
| `--workspace <path>` | | Override workspace directory |
| `--plan <id>` | | Specify plan when multiple exist |
| `--voice <mode>` | | Override coach voice for this invocation |
| `--version` | | Print version and exit |
| `--help` | `-h` | Show help and exit |

---

## 19. Command Inventory

```
newent                               Special entry experience (stats + orientation)
newent <business>                    Orient printout or creation prompt
newent <business> <section>          Navigate to section
newent <business> <address>          Navigate by address (periods optional)
newent <business> browser            Open browser for business
newent <address> browser             Open browser at specific address
newent <business> ent                Contextual view of business
newent <business> <address> ent      Contextual view of element
newent <business> new                New plan for business
newent <business> <section> new      New document in section
newent <business> <address> new      New item at addressed level

newent new                           Create new business interactively
newent list                          List all businesses in workspace
newent status [<business>]           Summary dashboard

newent doctor [<business>]           Diagnostic dashboard
newent coach <business>              Start or resume coached session
newent warmup <business>             Start or resume warmup session
newent warmups [<business>]          Warmup home / hangout view

newent notes <business>              List notes for business
newent notes <business> add          Add note
newent notes <business> <id>         View/manage specific note

newent develop <business>            Develop channel entry
newent document <business>           Document channel entry

newent export <business>             Export plan
newent export <business> --audience <framework>   Export with audience overlay

newent framework list                List available frameworks
newent framework show <id>           Show framework details
newent framework install <path>      Install framework from file
newent framework update              Pull framework updates (requires cloud)
newent framework populate <id>       Populate framework YAML via Q&A

newent spaces                        List all spaces
newent spaces create <name>          Create space
newent spaces add <business>         Add business to space
newent spaces show <name>            Show space contents

newent archive [<business>]          View archived items
newent archive restore <address>     Restore archived item
newent archive clear                 Permanently delete archive (confirmed)

newent browser                       Open workspace-level browser

newent config                        Show/edit workspace configuration
newent help [<topic>]                Help and documentation
newent version                       Show version and check for updates
newent update                        Update newent
```
