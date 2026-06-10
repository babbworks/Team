# newent — Data Model

> The content hierarchy, addressing scheme, and storage architecture.
> Read alongside ARCHITECTURE.md.
> Last revised: 2026-04-19

---

## 1. Core Hierarchy

```
Workspace
└── Business (slug, profile_short, profile_long?)
    ├── Plans
    │   └── Plan (id, title, type, framework, status, variant_of?)
    │       ├── Archive (deleted items, restorable)
    │       └── Sections (ordered, stable-addressed)
    │           └── Section (address_num, slug, title)
    │               └── Documents (ordered, stable-addressed)
    │                   └── Document (address_letter, slug, title, variant_of?)
    │                       └── Headings (ordered, stable-addressed)
    │                           └── Heading (address_num, title, variant_of?)
    │                               └── ContentItems (ordered, positional-label)
    │                                   └── ContentItem (label_letter, body, variant_of?)
    │                                       └── ContextItems (ordered, positional-label)
    │                                           └── ContextItem (label_numeral, body, variant_of?)
    ├── Notes (free-form, assignable to any level, optional title)
    ├── Warmup Sessions
    ├── QA Question-Answer Records
    └── Time Tracking Records
```

**Addressing stability:**
- Sections, Documents, Headings: stable address (assigned at creation, never changes)
- ContentItems, ContextItems: positional display label (reflects current order; resequences on deletion/reorder)

---

## 2. Entity Definitions

### Business
```
id                  UUID
slug                string (unique in workspace, e.g. "acme-corp")
name                string (display name, e.g. "Acme Corp")
profile_short       BusinessProfileShort (see Section 3)
profile_long        BusinessProfileLong? (see Section 3)
created_at          datetime
updated_at          datetime
```

### Plan
```
id                  UUID
business_id         UUID (FK)
title               string
type                enum: startup | operations | custom
framework_id        string (industry framework slug)
audience_framework  string? (set at export time)
status              enum: draft | active | archived
stage               enum: idea | pre-seed | seed | series-a | growth | established
target_audience     string? (brief description for this plan's intended audience)
created_at          datetime
updated_at          datetime
variant_of          UUID? (FK to parent plan — plan-level fork)
fork_date           datetime? (when this fork was created)
```

Plan variants (forks) are full independent copies. After forking, source and variant share no content. The `variant_of` reference is metadata only.

### Section
```
id                  UUID
plan_id             UUID (FK)
address_num         integer (stable, e.g. 3)
display_order       integer
slug                string (e.g. "market")
title               string (e.g. "Market Analysis")
created_at          datetime
```

### Document
```
id                  UUID
section_id          UUID (FK)
address_letter      string (stable uppercase, e.g. "B")
display_order       integer
slug                string
title               string
variant_of          UUID?
created_at          datetime
updated_at          datetime
```

### Heading
```
id                  UUID
document_id         UUID (FK)
address_num         integer (stable, e.g. 2)
display_order       integer
title               string
variant_of          UUID?
min_paragraphs      integer? (from framework definition)
is_complete         boolean (computed: content meets min_paragraphs threshold)
created_at          datetime
updated_at          datetime
```

### ContentItem
```
id                  UUID (internal, permanent)
heading_id          UUID (FK)
display_label       string (positional lowercase letter: "a", "b", "c" — resequences on change)
display_order       integer
body                text (markdown)
variant_of          UUID?
created_at          datetime
updated_at          datetime
```

### ContextItem
```
id                  UUID (internal, permanent)
content_item_id     UUID (FK)
display_label       string (positional roman numeral: "i", "ii", "iii" — resequences on change)
display_order       integer
body                text (markdown)
variant_of          UUID?
created_at          datetime
updated_at          datetime
```

**On ContentItem and ContextItem IDs:** The `display_label` is positional and resequences when items are deleted or reordered. This matches user intuition — the label reflects where something is in the current list. Internal `id` (UUID) tracks permanent identity for relations and history. Users interact with `display_label`; the system uses `id`.

---

## 3. Business Profile Models

### BusinessProfileShort
Captured at business creation. Prompts can be skipped and filled later.
```
industry_hint       string? (free text, used to suggest a framework)
stage               enum: idea | pre-seed | seed | series-a | growth | established
primary_location    LocationRecord?
primary_contact     ContactRecord?
website             string?
```

### BusinessProfileLong
Populated via warmup session (`profiling` tag) or directly via `newent acme-corp profile`.
```
founding_story      text?
vision              text?
mission             text?
key_challenges      list[text]?
problems_solving    list[text]?
early_team          text?
key_metrics         list[MetricRecord]?
notable_milestones  list[MilestoneRecord]?
other_narrative     text?
```

### LocationRecord
```
city        string?
region      string?
country     string
timezone    string?
```

### ContactRecord
```
name        string?
email       string?
phone       string?
role        string?
```

A business can have multiple contacts stored. The primary contact is flagged.

---

## 4. Addressing Scheme

Full five-level address: `<section>.<document>.<heading>.<content>.<context>`

```
3           → Section 3
3.B         → Document B in Section 3
3.B.2       → Heading 2 in Document B, Section 3
3.B.2.a     → Content item a
3.B.2.a.i   → Context item i of content item a
```

**Periods are optional.** `3B2ai` and `3.B.2.a.i` resolve identically.

**Identifier types by level:**
| Level | Type | Stability | Example |
|---|---|---|---|
| Section | Integer | Stable (creation) | `3` |
| Document | Uppercase letter | Stable (creation) | `B` |
| Heading | Integer | Stable (creation) | `2` |
| ContentItem | Lowercase letter | Positional (resequences) | `a` |
| ContextItem | Roman numeral | Positional (resequences) | `i` |

**Address scope:** Addresses are scoped to a plan. Two plans in the same business can both have `3.B.2.a.i`.

---

## 5. Variant Model

Variants are supported at: Plan, Document, Heading, ContentItem, ContextItem levels.

```
Document 3.B              ← original (active)
Document 3.B · v2         ← variant (variant_of = 3.B's UUID)
Document 3.B · v3         ← variant
```

Only one variant per address can be active at a time. The active variant is used in exports and Doctor scoring.

**Variant listing** shows relative numbers for selection and actions (make active, edit, archive, delete):
```
[1] Original  ← active
[2] Variant 2
[3] Variant 3
Actions: [r] replace active  [e] edit  [a] archive  [d] delete
```

---

## 6. Note Model

```
id              UUID (displayed as NOTE-XXXX)
business_id     UUID (FK) — notes belong to the business, not a specific plan
title           string? (optional; displayed if set, "(untitled)" if not)
body            text (markdown)
tags            string[]
assignment      AssignmentTarget?
display         boolean (default: true — renders as prologue when assigned to plan/section/document)
status          enum: active | used | archived
created_at      datetime
updated_at      datetime
source          enum: manual | warmup | imported | agent
```

**Notes as prologues:** Notes assigned to plan, section, or document level render as a preface above that element in views and exports. Toggle `display` per note, per element, or globally.

**AssignmentTarget:**
```
plan_id             UUID?
section_id          UUID?
document_id         UUID?
heading_id          UUID?
content_item_id     UUID?
context_item_id     UUID?
```

---

## 7. QA Question-Answer Model

```
Question:
id              UUID (globally unique, displayed as Q-XXXX)
text            text
follow_up       text?
tags            string[] (topic, plan element, stage, etc.)
source          string (which question set or framework it belongs to)
framework_id    string? (if associated with a specific framework element)
element_address string? (e.g. "3.B.2" — if tied to a plan element)
created_at      datetime

Answer:
id              UUID
question_id     UUID (FK)
business_id     UUID (FK)
plan_id         UUID? (FK)
session_id      UUID? (FK — warmup, coach, or QA session)
body            text
source_context  string (which service triggered this Q&A)
started_at      datetime (when prompt was shown)
submitted_at    datetime (when response was saved)
time_spent_sec  integer (computed: submitted_at - started_at)
```

`time_spent_sec` is the time tracking unit. Doctor aggregates these by heading, section, and plan to show time investment and generate completion projections.

---

## 8. Warmup Session Model

```
id                  UUID
business_id         UUID (FK)
question_set_id     string
status              enum: in_progress | complete | abandoned
qa_session_id       UUID (FK → QA session that backs this warmup)
summary_note_id     UUID? (FK to Note created on completion)
started_at          datetime
completed_at        datetime?
```

Warmup sessions are backed by QA sessions. The Warmup service creates a QA session and delegates question/answer management to the QA service.

---

## 9. Coach Session Model

```
id                  UUID
business_id         UUID (FK)
plan_id             UUID (FK)
intent              enum: orient | discover | decide | plan | narrate | prove | capital | expand | review
voice               enum: quiet | partial | full
status              enum: in_progress | complete | paused
qa_session_id       UUID (FK → QA session that backs this coach session)
current_heading_id  UUID? (resume point)
completed_headings  UUID[]
started_at          datetime
updated_at          datetime
```

---

## 10. Archive Model

```
ArchiveItem:
id              UUID
plan_id         UUID (FK)
archived_at     datetime
archived_by     enum: user | system
entity_type     enum: section | document | heading | content_item | context_item
entity_id       UUID (the original internal ID)
entity_data     json (complete snapshot of the entity at archival time)
address_at_archival string (e.g. "3.B.2.a")
restore_target  string? (address where it would be restored)
```

Archive items are viewable and restorable. The archive itself can be cleared, which is the only permanent deletion operation (requires two-step confirmation: `yes` + Enter).

---

## 11. Time Tracking

Time tracking is handled via the QA Answer model (`started_at`, `submitted_at`, `time_spent_sec`). Every prompt that leads to manual user input records this data.

Doctor aggregates time data to produce:
- Time spent per heading
- Time spent per document, per section, per plan
- Average time per completed content unit
- Estimated time to complete remaining content (avg × remaining units)
- Running completion projection as the plan fills in

Completion projection threshold: a heading is considered "complete" once its content meets `min_paragraphs` from the framework definition. Doctor uses completed headings to calibrate the time-per-unit estimate.

---

## 12. Storage Architecture

### Local (v1)
Single SQLite file per workspace: `<workspace>/.newent/store.db`

All entities in one file. Content bodies stored as TEXT in SQLite — no separate content files. Workspace is portable (copy `.newent/store.db`). Framework YAML files live alongside the DB.

### Workspace Layout
```
<workspace>/
├── .newent/
│   ├── store.db              ← all content and metadata
│   ├── config.toml           ← workspace settings
│   └── frameworks/           ← local framework overrides and installs
│       └── <id>/
│           ├── framework.yaml
│           └── hints/
│               └── <section-slug>.yaml
└── exports/                  ← generated output (gitignore-able)
    └── <business-slug>/
        └── <plan-title>/
```

**Spaces** are a separate global concept and are not stored per-workspace. Spaces metadata lives in a global config location (`~/.newent/spaces.db` or babb cloud when connected).

### Export Layout (generated, not source of truth)
```
exports/<business-slug>/<plan-title>/<audience-framework>/
    index.md
    sections/
        01-problem/
            A-problem-statement.md
            ...
    appendix.md
```

Export output is always regenerable from `store.db`. It is never the source of truth.

### Cloud (babb, future)
Schema maps 1:1 to PostgreSQL. Framework YAML files move to object storage. CLI becomes thin client; all reads/writes go to the babb API. The SQLite → PostgreSQL migration is a straight table copy via the Cloud service's sync operation.

---

## 13. Schema Versioning

`store.db` contains a `schema_version` table (single row). On startup, the tool compares the DB version against the current tool version and runs any pending migration scripts. Migrations are numbered and additive where possible.

Framework YAML files carry a `schema_version` field. Framework service validates compatibility at load time. Existing plans retain their framework version; new plans use the current version.
