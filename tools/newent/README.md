# newent v2

`newent` is a command-line tool for planning new enterprises with a local-first, service-oriented architecture.  
This folder (`tools/v2-newent`) is the active implementation workspace.

## Canonical Design References

The project is implemented against these primary design docs:

- `tools/ARCHITECTURE.md`
- `tools/CHANGELOG.md`
- `tools/CLI-DESIGN.md`
- `tools/DATA-MODEL.md`
- `tools/SERVICES.md`

These define the intended behavior, service boundaries, data contracts, and command grammar.  
When implementation details conflict with older notes, these references win.

## Current Implementation Scope

The current focus is **Store service foundation**:

- `newent/models.py` — dataclasses used at service boundaries
- `newent/services/store.py` — SQLite-backed Store service with broad CRUD surface
- `schema/001_initial.sql` — primary workspace schema
- `schema/001_spaces.sql` — global Spaces schema (future Spaces service)

## High-Level Architecture

- **Microservice discipline:** services are isolated by responsibility and communicate through typed contracts.
- **Store ownership rule:** only Store reads/writes SQLite directly.
- **Local-first:** data is stored in SQLite; cloud integration is planned but deferred.
- **Natural-first CLI:** command grammar favors subject-first, human-readable commands with progressive disclosure.

## Data Model Highlights

- Core hierarchy: Business -> Plan -> Section -> Document -> Heading -> ContentItem -> ContextItem
- Addressing supports compact and dotted forms (for example `3B2ai` and `3.B.2.a.i`)
- Stable addresses at upper levels, positional labels for content/context display
- Notes, archive, QA sessions/answers, and time tracking are first-class entities

## Service Inventory (Design Target)

Primary services include:

- Store
- Doctor
- QA
- Coach
- Warmup
- New
- Ent
- Notes
- Framework
- Export
- Spaces
- Cloud (stub)
- Finance (stub until dedicated design session)

## Developer Setup

From `tools/v2-newent`:

```bash
python3 -m pytest tests -q
python3 -m newent
```

## Working Rules for This Folder

- All implementation work for this phase should remain inside `tools/v2-newent`.
- Keep changes aligned to the five canonical docs listed above.
- Prioritize correctness and test coverage over speed, especially for Store behavior.
