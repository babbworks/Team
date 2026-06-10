# newent — System Architecture & Evolution

> Primary design document. Start here. Verbose by intent.
> Supersedes directional content in EVOLUTION-PLAN.md where they conflict.
> Last revised: 2026-04-19

---

## 1. Purpose & Vision

**newent** is a command-line tool for planning new enterprises. Its anchoring purpose is structured business plan creation (startup plan); its secondary purpose is operational planning. It is published and maintained as a standalone tool under the `newent` name, but its long-term home is as a first-class service within the **babb CLI** and **babb AI** platform.

The tool must serve two users simultaneously without compromising either:

- **The non-technical founder** who has an idea, some observations, and enough motivation to plan it seriously but no tolerance for jargon, nested command structures, or workflow scaffolding that feels like homework.
- **The power user** (technical founder, operator, AI agent) who needs piping, JSON output, deep addressing, automation hooks, and the ability to call any part of the system with precision.

These goals are not in tension if the design is right. The secret is progressive disclosure: the simplest path does something immediately useful; complexity is always available but never imposed.

---

## 2. Design Philosophy

### 2.1 Microservices Architecture

newent is structured as a collection of small, single-responsibility services with defined interfaces. In the current local implementation these are Python modules with clean API boundaries. As newent becomes part of the babb platform, each service is independently deployable. The discipline starts now.

**Rules:**
- Services do not import each other's internals. They communicate through defined data contracts.
- Each service owns its own data access. Nothing else reads its tables or files directly.
- Each service is independently testable in isolation.
- Service boundaries are the future network boundaries.

The services (see Section 4): **Store**, **Doctor**, **Coach**, **Warmup**, **QA**, **New**, **Ent**, **Develop**, **Document**, **Notes**, **Finance**, **Framework**, **Export**, **Spaces**, **Cloud**.

### 2.2 Natural-First CLI

Command grammar must feel like speech. A user should be able to describe what they want and have the command approximate that description.

```
newent acme-corp                    # "show me acme-corp"
newent acme-corp problems           # "take me to the problems section"
newent doctor acme-corp             # "run diagnostics on acme-corp"
newent warmup acme-corp             # "start a warmup session for acme-corp"
newent acme-corp problems new       # "create something new in problems"
newent acme-corp problems ent       # "show me problems in full context"
```

There is no required learning of "packs," "profiles," "assembly," or schema versioning for basic use. These are internal concerns, not user-facing vocabulary.

### 2.3 Modular Content Addressing

Every piece of content in a plan has a precise five-level address. The user is never required to know this address, but power users and agents can use it for surgical access. Periods between levels are optional — `3B2a` and `3.B.2.a` are equivalent. See DATA-MODEL.md for the full addressing scheme.

### 2.4 babb Evolution Path

When a service moves from local to the babb cloud:
1. The service's Python module becomes an HTTP microservice (FastAPI, likely).
2. The CLI becomes a thin client making authenticated API calls.
3. Data moves from local SQLite to a cloud store (PostgreSQL + object storage).
4. The content model is unchanged. The addressing scheme is unchanged.
5. The user experience is unchanged except that data is synced and shareable.

Every design decision should be evaluated against this evolution path. Avoid coupling to the local filesystem in service logic — keep filesystem access in the Store service only.

### 2.5 Document Versioning

Planning documents and READMEs evolve frequently. When any guidance document is substantially revised, the previous version is archived in `archive/` with the archival date in its filename and a brief header noting what changed and why. This preserves the design record without polluting the working document set.

---

## 3. Who We Serve: The User Spectrum

| Persona | Profile | Needs from newent |
|---|---|---|
| **Early Founder** | Has an idea, minimal planning experience | Warmup sessions, guided coach, no jargon, encouraging output |
| **Experienced Operator** | Has run businesses, building next one | Quick navigation, direct section access, variant management |
| **Investor-Prep Founder** | Preparing for funding round | Audience framework overlays, doctor fitness metrics, export |
| **Technical Founder** | Comfortable with CLI, wants automation | JSON output, piping, agent-accessible addressing, scripting |
| **babb AI Agent** | Automated agent drafting plan sections | Full addressable API, JSON I/O, notes assignment, batch ops |

Design decisions must not penalize any persona to benefit another.

---

## 4. Service Inventory

### 4.1 Store Service
**Responsibility:** All data persistence. The only service that reads from or writes to disk/database.

Owns: business registry (including profile data), plan registry, section/document/heading/content/context hierarchy, notes, time tracking records, QA question/answer store, framework metadata, session state.

Storage: SQLite per workspace (local), cloud DB (babb). Content bodies stored as markdown text within the DB; exportable to files on demand.

Time tracking is owned by Store: every Q&A interaction that leads to manual content input records start and end timestamps. The delta is stored per heading interaction and surfaced by Doctor.

### 4.2 Doctor Service
**Responsibility:** Diagnostic analysis of any business or plan. Functions as a repeatable dashboard — run `newent doctor acme-corp` at any point and get a current health snapshot.

Doctor is designed to be extended with **lenses**: distinct analytical views that can be added over time without redesigning the core. The initial lens set is content completeness and framework fitness. Future lenses may include: financial health, narrative coherence, time investment, competitive positioning, investor readiness. Each lens produces a typed report; Doctor assembles them into a unified dashboard.

Outputs (initial lens set):
- Section coverage percentage
- Document completion estimates (headings filled vs. total)
- Word/substance counts per section
- Framework fitness score against selected industry framework
- Active notes count (unassigned, assigned, used)
- Warmup sessions complete vs. pending
- Time spent per section / per document (from Store time tracking)
- Estimated time to completion (based on avg time per content unit × remaining units)
- Running completion projections as plan fills in

Key design constraint: Doctor is read-only. It never modifies state. It is always safe to run.

### 4.3 Coach Service
**Responsibility:** Guided workflow through any develop or document channel. The tool's primary concierge.

Coach has three voices:
- **quiet**: prompt only. Minimal output. For experienced users who know what they're doing.
- **partial**: condensed guidance + key references at each input point. Good default.
- **full**: rich contextual content — explanatory text, examples, references, links, motivating framing.

Coach delegates all Q&A interactions to the **QA Service**. Coach does not manage questions directly — it calls QA to fetch the right question for each heading, record responses, and track session progress. This means Coach prompts are first-class QA questions with unique IDs, and any answered question is referenceable across the system.

At each input point, the relevant context is loaded from framework hint YAML files and presented per the active voice.

Coach routing maps the nine intents to service channels:
- **Develop channel**: orient, discover, decide, plan, expand, review
- **Document channel**: narrate, prove, capital

Sessions are stateful and resumable. `newent coach acme-corp resume` returns to the exact heading where the user left off.

### 4.4 Warmup Service
**Responsibility:** Informal, low-friction elicitation. Warmup is the tool's answer to the blank-page problem — a place to think out loud and have it organized for you.

`warmups` (plural command) is the Warmup landing experience: a browseable, hangout-friendly space showing all available question sets, in-progress sessions, completed sessions, and session summaries. The user can drift through it without committing to a formal plan session.

Warmup delegates all Q&A interactions to the **QA Service**. Warmup question sets are registered in the QA service's question library. This means a warmup question can be surfaced during a coach session or a standalone QA session — questions are portable across contexts.

Outputs of a warmup session:
- A structured summary note stored in the Notes service (with unique ID, auto-titled)
- Optional direct assignment to a plan section
- A prompt to proceed to a related coach intent

Warmup has no dependency on a plan existing. Multiple warmup sessions per business are expected.

### 4.5 QA Service
**Responsibility:** The unified question-and-answer engine for the whole tool. Manages a library of questions with unique IDs across all contexts: warmup sets, coach prompts, framework population, business profiling.

QA enables **vibed sessions**: a user can pull questions from different plan elements based on what they're in the mood to think about, rather than following a prescribed sequence. The QA service composes question sets on demand from tags, topics, or explicit question IDs.

Questions have unique IDs, are tagged by topic and plan element, and can be grouped into named sets or requested ad-hoc. Every answered question is stored with its response, question ID, source context, and timestamps. Time tracking for engagement (when a prompt led to manual input) is captured here and fed to Doctor via Store.

QA also powers framework YAML file population: instead of hand-editing YAML, a user can run `newent framework populate <id>` and be walked through the fields as a guided Q&A session.

### 4.6 New Service
**Responsibility:** Context-sensitive creation at any level of the hierarchy. `new` resolves to "create a new [thing] at the current navigation level."

`new` is aware of context:
```
newent new                              → new business
newent acme-corp new                    → new plan for acme-corp
newent acme-corp problems new           → new document in problems section
newent acme-corp problems 3.B new       → new heading in document 3.B
newent acme-corp problems 3.B.2 new     → new content item in heading 3.B.2
newent acme-corp problems 3.B.2.a new   → new context item for content item 3.B.2.a
```

`new` also handles the inline creation syntax for reordering + inserting within any ordered set:
```
1 3 2 new 4 5 new
```
This sequence means: reorder to positions 1, 3, 2, then create a new item, then continue with 4, 5, then create another new item. The New service processes this sequence, calling QA service for each `new` trigger to collect the new item's content.

### 4.7 Ent Service
**Responsibility:** Meta-contextual lens. Places any plan element — section, document, heading, content item — within its full plan and company context. `ent` is a viewing and understanding service, not a content creation service.

`ent` generates a **situated view**: the element plus its relationship to everything around it — parent section purpose, sibling elements, assigned notes, related warmup responses, and (in future AI-enhanced mode) an analytical framing of how this element contributes to or diverges from the plan's overall argument.

```
newent acme-corp ent                    → company context view
newent acme-corp problems ent           → problems section in plan context
newent acme-corp 3.B ent                → document in section + plan context
newent acme-corp 3.B.2 ent              → heading with full surrounding context
```

The name `ent` echoes the `new` in `newent` — a new entity, placed in its context. It is a first-class service because context-aware views will become richer over time (AI integration, cross-plan comparison, audience lens overlay). The service interface is designed to accommodate that growth.

### 4.8 Develop Service
**Responsibility:** Tracks and routes progress through develop-channel intents (orient, discover, decide, plan, expand, review). Acts as a coordinator, not a generator.

### 4.9 Document Service
**Responsibility:** Manages narrate, prove, capital channels. Prepares documents for export. Bridges content hierarchy (Store) and the Export service.

### 4.10 Notes Service
**Responsibility:** Free-form content capture and assignment. Each note has a unique ID, optional title, timestamp, and markdown body.

Notes assigned to a plan, section, or document level become **prologues** — they render as a preface above that element in any view or export. Note display (as prologue or as floating annotation) can be toggled per element or globally.

Notes are listed for any element as a relatively numbered list. The user can reorder notes by responding with a new number sequence. The inline creation syntax (`1 3 new 4 2`) is also supported for note sets.

### 4.11 Finance Service
**Responsibility:** Deferred pending dedicated design session.

Scope for that session: financial model structure per industry framework; guided P&L, cash flow, and cap table input; seed vs. Series A vs. bank-loan financial depth; non-SaaS model support; Finance output integration with audience framework overlays.

Do not design or implement until the Finance design session is completed.

### 4.12 Framework Service
**Responsibility:** Manages industry frameworks and audience frameworks. Provides structural templates. Handles framework installation and updates.

Two framework dimensions:
- **Industry framework**: shapes the section/document/heading structure of a plan. Includes vocabulary, expected elements, and completeness thresholds.
- **Audience framework**: shapes how existing plan content is packaged for export. Applied at export time only.

Framework YAML files use fully declared field schemas with `optional:` markers inline or in a dedicated `optional_fields:` section. Every possible field is listed in the YAML — no implicit fields. This makes the files self-documenting and enables QA-driven population.

Framework updates:
- `newent framework update` — pull updates from babb registry (when connected)
- `newent framework install <path>` — install from local YAML file
- `newent framework install <url>` — install from URL (when cloud enabled)
- Bundled framework updates arrive with tool version upgrades (breaking changes handled by schema migration)

### 4.13 Export Service
**Responsibility:** Generates output files from plan content. Applies audience framework overlays. Non-destructive.

### 4.14 Spaces Service
**Responsibility:** Optional organizational grouping for businesses and plans. Spaces are a global microservice — they exist independently of any workspace directory. A business or plan does not need to belong to a space, but can be added to one or more.

Spaces differ from workspaces:
- A **workspace** is a directory-level concept (where `.newent/store.db` lives).
- A **space** is a named logical collection that can span multiple businesses, plans, or workspaces.

```
newent spaces                           → list all spaces
newent spaces create "Portfolio 2026"   → new space
newent spaces add acme-corp             → add a business to a space
newent spaces show "Portfolio 2026"     → show contents and summary
```

Spaces are designed for: portfolio views, project groupings, team-level organization, and (in babb) shared workspaces across collaborators.

### 4.15 Cloud Service
**Responsibility:** Reserved. Not implemented in local v1. Stub module with `NotImplementedError` pointing to the babb roadmap.

When implemented: authentication, sync, collaboration, AI feature access, framework registry, and remote backup. All other services remain unchanged when Cloud is activated — they route through Cloud's network layer without changing their own logic.

---

## 5. The Framework System

### 5.1 Industry Frameworks

An industry framework defines:
- The ordered set of plan sections
- Which documents belong in each section, and their ordering
- Which document headings are expected, which are optional
- Industry-specific vocabulary substitutions
- Minimum content thresholds per heading (minimum recommended paragraphs)
- Coach hint associations per heading

Bundled industry frameworks for v1.0 (minimum):
- `generic` — neutral baseline; current 18-domain registry is the seed
- `saas` — ARR/MRR, unit economics, product-led or sales-led motions
- `consumer-product` — DTC/retail/wholesale, SKU economics, channel mix
- `professional-services` — utilization model, retainer/project mix
- `manufacturing` — BOM, CapEx, supply chain, certification
- `nonprofit` — mission-driven, grant/donation revenue model

Framework development beyond this minimum set should be done with web research and real-world investor/lender expectations analysis per industry at coding time.

### 5.2 Audience Frameworks

An audience framework defines:
- Which sections and documents are included in the export (filter, reorder)
- Framing and tone guidance per section
- Required vs. optional elements for this audience
- Export format preferences

Bundled audience frameworks for v1.0 (minimum):
- `investor-seed` — YC/a16z/Sequoia style, traction-forward, concise
- `investor-growth` — Series A+, three-statement financials, cohort analysis
- `hbs-plan` — full HBS format, operations depth, risk section, 20-50 pages
- `bank-loan` — lender-facing, downside analysis, collateral, three-statement
- `internal-ops` — operational plan, team-facing, no investor framing
- `grant-application` — nonprofit/research, mission framing

### 5.3 Framework Fitness

Doctor reports framework fitness: the percentage of the selected industry framework's required elements that are present and substantive. Each heading has a minimum content threshold (minimum recommended paragraphs); once met, Doctor marks it complete and updates the running completion projection.

---

## 6. Coach Service: Detailed Design

### 6.1 Voice Modes

Voice mode is set per session and persists as a user preference. Default is `partial`.

| Voice | What prints at each input prompt |
|---|---|
| quiet | The bare prompt. Nothing else. |
| partial | Prompt + 2-4 line condensed guidance + one key reference or example |
| full | Prompt + explanatory context + examples + references + links + motivating framing |

### 6.2 Context Files (YAML Hint System)

For every document heading, there is a corresponding context entry in the framework's hint YAML. Hint files live at:
```
frameworks/<framework-id>/hints/<section-slug>/<document-slug>.yaml
```

Each entry contains fully declared fields:
```yaml
heading_slug: target-market-definition
quiet_label: "Who is your primary customer?"
partial_hint: |
  Define your primary customer with enough specificity to be useful.
  Avoid: "everyone." Target: a segment you can name, reach, and measure.
full_context: |
  ...multi-paragraph explanatory text...
examples:
  - "B2B SaaS: mid-market HR teams at companies with 200-2000 employees"
  - "Consumer: urban renters aged 28-42 with household income $80K+"
references:
  - title: "YC's advice on customer definition"
    description: "Paul Graham on doing things that don't scale first"
    url: ~  # filled in by content update or AI agent
tags: [market, customer, required]
optional: false
min_paragraphs: 2
```

All fields are declared. Optional fields are marked `optional: true` or listed in the file's `optional_fields:` section. This makes hint files self-documenting and QA-populatable.

### 6.3 Session State and Time Tracking

Coach session state includes:
- Current intent/channel
- Current heading (resume point)
- Completed headings with timestamps
- Time spent per heading (start_time captured when prompt displays, end_time when response submitted)

Time data is stored via Store and surfaced by Doctor.

---

## 7. The Bare `newent` Command

Running `newent` with no arguments is a designed experience — not a help screen. It is an evolving entry point combining stats, orientation, and inspiration.

Initial form:
- A brief welcome or return greeting (first run vs. returning user)
- Quick stats: businesses in workspace, plans active, recent activity
- A suggestion or prompt: the next meaningful action (from Develop or Doctor)
- A brief inspirational element (evolving — could be a question, a quote, a planning prompt)
- Soft navigation hints (not a full command list)

This is designed to be pleasant to land on repeatedly. It grows more personalized as the tool accumulates data about the user's plan state.

---

## 8. Browser Service

The browser experience is a correlated visual interface to the CLI. Access is:
```
newent browser                          → open workspace-level browser
newent acme-corp browser                → open browser for acme-corp
newent acme-corp problems browser       → open browser at the problems section
```

Browser renders the same data from Store. The URL structure mirrors CLI addresses. The Doctor dashboard is a natural browser-first view. Export preview renders in browser before file output.

Browser implementation is deferred. The data model and addressing scheme support it from the start.

---

## 9. Business Profile Data

Every business captures profile data in two forms:

**Short form** (captured at creation, or on first run if skipped):
- Business name (display), slug, industry (framework selection hint)
- Primary location (city, country)
- Primary contact (name, email, phone, website)
- Stage (idea, pre-seed, seed, series-a, growth, established)

**Long form** (accessible via warmup or directly via `newent acme-corp profile`):
- Founding story
- Vision and mission statements
- Key challenges and problems being solved
- Early team snapshot
- Key metrics or milestones to date
- Any other grand-picture narrative

Long-form profile elements are stored as structured fields but feel seamless — they can be populated through a warmup session tagged `profiling` or directly through coach's `orient` intent. Long-form data feeds into the `executive-summary` section of any plan.

Plan profiles similarly capture plan-level details: plan title, type, target audience, creation date, parent plan (if variant/fork).

---

## 10. Deletion and Archiving

Deletion in newent is a two-step confirmation requiring explicit text plus an extra Enter:

1. User issues delete command
2. System shows what will be affected
3. Prompt: `Type "yes" to move this to archive, then press Enter: _`
4. User types `y` or `yes`
5. User presses Enter
6. Content moved to plan archive, not permanently deleted

**Plan Archive:**
- Every plan has an archive within its data
- Deleted sections, documents, headings, content items, and context items are moved there
- Archive is viewable: `newent acme-corp archive`
- Archive can be cleared: `newent acme-corp archive clear` (requires same two-step confirmation)
- Archive clearing is permanent

---

## 11. Tool Updates and Framework Updates

**Tool updates:**
- `newent version` shows installed version and whether an update is available
- `newent update` updates the tool (delegates to the package manager: pip, pipx, brew)
- Schema migrations run automatically on startup when `store.db` version is behind tool version
- Migrations are additive where possible; breaking changes are documented in the changelog

**Framework updates:**
- Bundled frameworks update with the tool
- Custom/local frameworks: `newent framework install <path>`
- Remote framework registry (babb): `newent framework update` (requires Cloud service)
- Framework version is stored per plan; old framework versions remain valid for existing plans

---

## 12. Dependency and Technology Decisions

| Decision | Choice | Reason |
|---|---|---|
| CLI framework | Typer | Clean, maintained, good for nested commands and typed args |
| Data store (local) | SQLite via stdlib `sqlite3` | No new deps, local-first, trivially portable |
| YAML | PyYAML | Hint files, framework files |
| Rich output | `rich` library | Doctor dashboard, coach voices, selection UIs, color |
| Type hints | Dataclasses throughout | Service boundary discipline; JSON-serializable interfaces |
| Testing | pytest + golden fixtures | Per-service unit tests + CLI integration tests |
| Browser (deferred) | TBD | Likely a lightweight local server; data model is browser-ready |

---

## 13. Build Order

1. **Store service + data model** — schema, all entities, time tracking tables
2. **Business and plan CRUD** — `newent acme-corp`, reserved word detection, plan selection UX, profile data (short form)
3. **New service** — context-sensitive creation, inline creation syntax
4. **Framework service + generic framework** — full field schema YAML, scaffold generation
5. **Doctor service** — lens architecture, initial lens set, time tracking display
6. **QA service** — question library, session management, time tracking hooks
7. **Notes service** — CRUD, assignment, prologue behavior, reorder syntax
8. **Warmup service** — delegates to QA, hangout experience, session summaries
9. **Coach service** — voices, delegates to QA, session state, hint loading
10. **Ent service** — contextual view generation
11. **Spaces service** — global collections
12. **Export service** — markdown first, then audience framework overlays
13. **Browser** — after core CLI stable
14. **Finance service** — after dedicated design session
15. **Cloud service** — after babb platform is ready
