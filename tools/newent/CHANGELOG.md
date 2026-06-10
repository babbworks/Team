# newent — Design & Architecture Changelog

> Ordered by service/component. Records step changes and major adjustments to tool function and design.
> Code-level changes belong in git history. This log tracks design decisions.

---

## Format

Each entry: `[YYYY-MM-DD] Description` — optionally followed by a brief note on the reason or impact.

---

## Global / Cross-Cutting

- [2026-04-18] Initial architecture established: microservices discipline, SQLite Store, Typer CLI, PyYAML frameworks, `rich` output library
- [2026-04-18] Document archiving policy established: superseded guidance docs archived to `archive/` with date suffix and archival header
- [2026-04-19] Migration strategy removed from ARCHITECTURE.md — rewrite targets v1 directly; no migration path needed
- [2026-04-19] Bare `newent` command defined as a designed experience (stats + orientation + inspiration), not a help screen
- [2026-04-19] Periods in content addresses made optional — `3B2ai` and `3.B.2.a.i` are equivalent
- [2026-04-19] Deletion flow defined: two-step (y/yes + Enter), content moves to archive not permanent delete
- [2026-04-19] Build order updated to reflect new services (QA, New, Ent, Spaces, Cloud)

---

## Store Service

- [2026-04-19] schema/001_initial.sql written — full DDL committed to v2-newent repo (git: 7714e47)
- [2026-04-19] schema/001_spaces.sql written — global spaces.db DDL
- [2026-04-18] Initial schema: Business, Plan, Section, Document, Heading, ContentItem, Note, CoachSession, WarmupSession
- [2026-04-19] ContextItem added as 5th addressing level (attached to ContentItem)
- [2026-04-19] BusinessProfileShort and BusinessProfileLong added to Business entity
- [2026-04-19] LocationRecord and ContactRecord models added
- [2026-04-19] QA Answer model added with time tracking fields (started_at, submitted_at, time_spent_sec)
- [2026-04-19] Archive model added: ArchiveItem with full entity snapshot and restore support
- [2026-04-19] Note reordering added: reorder_notes() method on Store
- [2026-04-19] ContentItem and ContextItem addressing clarified: display_label is positional (resequences), internal UUID is permanent identity
- [2026-04-19] Plan fork (plan-level variant) added: fork_plan() method; variant_of + fork_date fields on Plan

---

## Doctor Service

- [2026-04-18] Initial design: content coverage, framework fitness, word counts
- [2026-04-19] Lens architecture introduced: Doctor assembles reports from registered lenses; new lenses can be added without redesigning core
- [2026-04-19] TimeInvestmentLens added: time spent per section/document, avg time per content unit, completion projection
- [2026-04-19] Completion projection logic defined: heading is "complete" when content meets min_paragraphs threshold; projection calibrated from completed headings

---

## QA Service

- [2026-04-19] New service introduced: unified question-and-answer engine across the whole tool
- [2026-04-19] Questions get globally unique IDs (Q-XXXX format)
- [2026-04-19] Vibe session support: ad-hoc cross-topic question sets composed by tags
- [2026-04-19] Coach and Warmup services both delegate Q&A to QA service (not managing questions independently)
- [2026-04-19] Framework YAML population via Q&A: populate_via_qa() on Framework service calls QA service
- [2026-04-19] Time tracking: record_answer() captures started_at → submitted_at delta per Q&A interaction

---

## New Service

- [2026-04-19] New service introduced: context-sensitive creation at any hierarchy level
- [2026-04-19] Inline creation syntax defined: `1 3 new 2 4 new` sequence for any ordered set (notes, content items, context items)
- [2026-04-19] `new` command resolves to correct entity type based on navigation context

---

## Ent Service

- [2026-04-19] New service introduced: meta-contextual lens placing any element in plan/company context
- [2026-04-19] `ent` accessible as trailing command on any navigation: `newent acme-corp problems ent`
- [2026-04-19] EntView structure defined; ai_context field reserved for future Cloud/AI enrichment
- [2026-04-19] Name etymology: `ent` echoes the `new` in `newent` (new entity, placed in context)

---

## Coach Service

- [2026-04-18] Initial design: develop/document channel routing, three voices (quiet/partial/full), YAML hint files, stateful sessions
- [2026-04-19] Coach delegates Q&A to QA service — coach prompts are registered as QA questions with unique IDs
- [2026-04-19] Time tracking: answer timestamps flow through QA service → Store → Doctor

---

## Warmup Service

- [2026-04-18] Initial design: question-set based elicitation, summary notes output
- [2026-04-19] `warmups` (plural) command introduced as hangout landing experience — browseable, not transactional
- [2026-04-19] Warmup delegates all Q&A to QA service — warmup question sets registered in QA library
- [2026-04-19] Vibe session support inherited from QA service

---

## Notes Service

- [2026-04-18] Initial design: CRUD, assignment to hierarchy levels, source tagging
- [2026-04-19] Optional titles added to notes
- [2026-04-19] Prologue behavior defined: notes assigned to plan/section/document level render as prefaces in views and exports
- [2026-04-19] Display toggle added: per-note and global
- [2026-04-19] Note reordering via relative number sequence
- [2026-04-19] Inline note creation syntax: `1 3 new 2` supported for any note set
- [2026-04-19] Notes assignable to ContextItem level (added with ContextItem entity)

---

## Framework Service

- [2026-04-18] Initial design: industry + audience framework dimensions, YAML-defined, bundled defaults
- [2026-04-19] Framework YAML schema discipline: all fields fully declared, optional fields marked inline or in optional_fields section
- [2026-04-19] QA-driven framework population: populate_via_qa() method added
- [2026-04-19] Framework update/install commands defined: install from file, update from registry (requires Cloud)
- [2026-04-19] min_paragraphs field added per heading in framework definition (used by Doctor for completion threshold)
- [2026-04-19] Framework development guidance added: initial set is minimum; full industry-specific development to be done with web research at coding time

---

## CLI Surface

- [2026-04-18] Initial grammar: business-name-first, verb-first as alternative, reserved word detection
- [2026-04-18] Four-level addressing: Section(int) · Document(letter) · Heading(int) · ContentItem(letter)
- [2026-04-19] Fifth addressing level added: ContextItem (roman numeral)
- [2026-04-19] Periods in addresses made optional
- [2026-04-19] `new` and `ent` added as first-class commands
- [2026-04-19] Browser access pattern defined: `newent browser`, `newent <business> browser`, etc.
- [2026-04-19] `warmups` (plural) added as Warmup landing command
- [2026-04-19] Bare `newent` defined as designed entry experience
- [2026-04-19] Deletion flow defined: two-step confirmation, archive-first, clear-archive as only permanent delete
- [2026-04-19] Single-item UX defined: auto-select with Enter-to-confirm, inline stats hint, new/variant options
- [2026-04-19] Variant listing defined: relative numbers, action menu (replace active, edit, archive, delete)
- [2026-04-19] Note reordering and inline creation syntax added to CLI surface
- [2026-04-19] Reserved words updated: added warmups, browser, ent, spaces, cloud, archive

---

## Data Model

- [2026-04-18] Core hierarchy: Business → Plan → Section → Document → Heading → ContentItem
- [2026-04-18] Note model with AssignmentTarget
- [2026-04-18] SQLite per workspace in `.newent/store.db`
- [2026-04-19] ContextItem added as ContentItem child (fifth level)
- [2026-04-19] ContentItem and ContextItem addressing changed to positional display labels (not stable)
- [2026-04-19] BusinessProfileShort and BusinessProfileLong added
- [2026-04-19] LocationRecord and ContactRecord added
- [2026-04-19] Plan-level variant (fork) supported: variant_of + fork_date on Plan
- [2026-04-19] Archive model added per plan
- [2026-04-19] QA Question and Answer entities added
- [2026-04-19] Time tracking fields added to Answer
- [2026-04-19] Spaces stored globally at ~/.newent/spaces.db, separate from workspace store.db

---

## Spaces Service

- [2026-04-19] New service introduced: optional global organizational grouping for businesses and plans
- [2026-04-19] Spaces are separate from workspaces — businesses and plans do not require a space
- [2026-04-19] Spaces stored globally, sync to babb cloud when Cloud service is connected

---

## Cloud Service

- [2026-04-19] Reserved stub introduced: not implemented in local v1, designed for babb platform integration
- [2026-04-19] Scope defined: auth, sync, collaboration, AI features (Ent enrichment, QA AI curation), remote framework registry

---

## Finance Service

- [2026-04-18] Reserved stub introduced: pending dedicated design session
- [2026-04-19] Scope flags documented in ARCHITECTURE.md; no implementation until design session completed
