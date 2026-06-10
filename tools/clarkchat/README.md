# Clark Chat

Clark Chat is the conversation layer for CLARK: a persistent, type-aware chat system where work, learning, and automation all attach to continuous conversations.

This repository currently contains the product requirements and early project scaffolding for the next version of Clark Chat.

## Why This Exists

Traditional chat tools treat messages as isolated streams. Clark Chat treats a conversation as a durable, structured object with:

- A semantic type (for example: Job, Room, Task, Course)
- Reusable attached elements (for example: Members, Context, Files, AI)
- Persistent history and state across sessions
- A surface for humans and microservices to collaborate in the same thread

The objective is to make conversation the system of record for operations, learning, and automation.

## Current Repository Status

This repo is in architecture + requirements phase:

- Primary source of truth: `chat-centric-requirements.md`
- Frontend folders (`src`, `dist`) exist as placeholders
- Tooling files for running the app locally are not yet committed in this workspace snapshot

If you are implementing the app, start from the requirements document first, then layer in frontend/runtime setup.

## Core Domain Model

The requirements define three key concepts.

### 1) Conversation Types (13)

Conversation types describe what a conversation *is* in the business domain.

- Work groups: `Rooms`, `Locations`, `Stations`, `Tools`
- Work types: `Jobs`, `Tasks`, `Reviews`
- Work materials: `Inputs`, `Outputs`
- Learning contexts: `Classes`, `Courses`, `Teams`

### 2) Conversation Elements (13)

Elements are reusable capabilities attachable to any conversation type.

- `Members`, `Context`, `Purpose`, `Access`, `History`
- `Files`, `Qs & As`, `Choices`, `Web`, `Terms`
- `Apps`, `AI`, `Instructions`

### 3) Continuous Conversations

Conversations are long-lived objects that:

- Persist even when users leave
- Restore context on rejoin
- Maintain history and relationships
- Support lifecycle transitions (active, archived, deleted)

## Target Architecture

At implementation time, Clark Chat is intended to follow this shape:

```
Clark Chat Client(s)
    |
    |  XMPP over WebSocket
    v
Prosody XMPP Server
    |
    +-- Human participants
    +-- AI participants
    +-- Typed conversation rooms
    +-- Microservices attached by conversation type
```

This keeps messaging transport simple while enabling rich domain semantics above the transport layer.

## What To Build Next

Recommended implementation order:

1. Define schemas for the 13 conversation types and 13 conversation elements.
2. Implement conversation lifecycle + access control rules.
3. Implement relationship and traceability model (especially Inputs -> Work -> Outputs).
4. Implement search/indexing + notification events.
5. Add microservice interface for read/write/subscribe on conversation elements.
6. Build UI views around typed conversations rather than generic channels.

## Contributor Workflow

Until full app scaffolding is committed, use this workflow:

1. Read `chat-centric-requirements.md` end to end.
2. Pick a requirement set (taxonomy, lifecycle, access, search, etc.).
3. Propose a data model + API contract in a short design note.
4. Implement incrementally with tests per requirement group.
5. Keep docs in sync with behavior changes.

## Design Principles

- **Conversation-first**: every capability attaches to a conversation.
- **Type-aware behavior**: conversation type drives policy and UX.
- **Composable elements**: features are reusable across types.
- **Persistent context**: history and state survive session boundaries.
- **Service interoperability**: humans and microservices share one model.

## Document Reference

- Product requirements: `chat-centric-requirements.md`
