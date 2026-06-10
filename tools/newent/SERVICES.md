# newent — Service Boundaries

> Microservice definitions, responsibilities, and interfaces.
> Read alongside ARCHITECTURE.md.
> Last revised: 2026-04-19

---

## Service Architecture Principles

1. **Single responsibility.** One clear job per service. When the job becomes hard to state in one sentence, split it.
2. **Owned data access.** A service accesses only its own tables. Other services call its methods, not its queries.
3. **Defined interfaces.** Services expose Python functions with typed signatures. Dataclasses for all inputs/outputs. No direct attribute access across service boundaries.
4. **No circular dependencies.** Services form a directed acyclic graph. CLI calls services; services may call Store; no service calls the CLI.
5. **Independently testable.** Every service can be tested in isolation with a test database and mock inputs.
6. **Future network boundary.** All method signatures must translate to HTTP endpoints. No method accepts or returns objects that cannot be serialized to JSON.

---

## Dependency Graph

```
CLI layer
    ├── New Service       → Store
    ├── Ent Service       → Store, Framework, Notes
    ├── Doctor Service    → Store
    ├── Coach Service     → Store, Framework, QA
    ├── Warmup Service    → Store, Notes, QA
    ├── QA Service        → Store
    ├── Develop Service   → Store, Coach
    ├── Document Service  → Store, Framework, Export
    ├── Notes Service     → Store
    ├── Finance Service   → Store, Framework (stub)
    ├── Export Service    → Store, Framework
    ├── Framework Service → Store (user selections only)
    ├── Spaces Service    → Store (global spaces DB)
    └── Cloud Service     → all services (stub)

All services → Store Service (data access)
```

---

## 1. Store Service

**Module:** `newent/services/store.py`

**Responsibility:** All reads and writes to `.newent/store.db`. The only code that touches SQLite.

**Selected interface:**
```python
# Business + Profile
def create_business(slug: str, name: str, profile_short: BusinessProfileShort) -> Business
def get_business(slug: str) -> Business | None
def list_businesses() -> list[Business]
def update_profile_short(business_id: UUID, profile: BusinessProfileShort) -> Business
def update_profile_long(business_id: UUID, profile: BusinessProfileLong) -> Business

# Plan
def create_plan(business_id: UUID, title: str, type: PlanType, framework_id: str) -> Plan
def fork_plan(source_plan_id: UUID, new_title: str) -> Plan
def get_plan(plan_id: UUID) -> Plan | None
def list_plans(business_id: UUID) -> list[Plan]

# Content hierarchy
def create_section(plan_id: UUID, slug: str, title: str) -> Section
def create_document(section_id: UUID, slug: str, title: str) -> Document
def create_heading(document_id: UUID, title: str, min_paragraphs: int | None) -> Heading
def create_content_item(heading_id: UUID, body: str) -> ContentItem
def create_context_item(content_item_id: UUID, body: str) -> ContextItem
def update_content_item(item_id: UUID, body: str) -> ContentItem
def reorder_content_items(heading_id: UUID, ordered_ids: list[UUID]) -> list[ContentItem]
def reorder_context_items(content_item_id: UUID, ordered_ids: list[UUID]) -> list[ContextItem]
def resolve_address(plan_id: UUID, address: str) -> AddressTarget

# Archive
def archive_entity(entity_type: EntityType, entity_id: UUID) -> ArchiveItem
def list_archive(plan_id: UUID) -> list[ArchiveItem]
def restore_archive_item(archive_item_id: UUID) -> AddressTarget
def clear_archive(plan_id: UUID) -> int  # returns count of permanently deleted items

# Notes
def create_note(business_id: UUID, body: str, source: NoteSource, title: str | None) -> Note
def update_note(note_id: UUID, body: str | None, title: str | None, display: bool | None) -> Note
def list_notes(business_id: UUID, status: NoteStatus | None) -> list[Note]
def assign_note(note_id: UUID, target: AssignmentTarget) -> Note
def reorder_notes(target: AssignmentTarget, ordered_ids: list[UUID]) -> list[Note]

# QA
def create_question(text: str, tags: list[str], follow_up: str | None) -> Question
def create_answer(question_id: UUID, business_id: UUID, body: str,
                  started_at: datetime, submitted_at: datetime) -> Answer
def list_questions(tags: list[str] | None, element_address: str | None) -> list[Question]

# Time tracking (derived from QA answers)
def get_time_by_heading(plan_id: UUID) -> dict[UUID, int]  # heading_id → seconds
def get_time_by_section(plan_id: UUID) -> dict[UUID, int]
def get_avg_time_per_content_unit(plan_id: UUID) -> float | None
```

Does NOT render output, apply business logic, or interpret frameworks.

---

## 2. Doctor Service

**Module:** `newent/services/doctor.py`

**Responsibility:** Diagnostic analysis. Read-only. Assembles health reports from multiple lenses. Designed for extension — new lenses can be added without redesigning the core.

**Lens architecture:**
```python
class DoctorLens(Protocol):
    def name(self) -> str: ...
    def run(self, plan_id: UUID, store: StoreService) -> LensReport: ...
```

Initial lens set:
- `ContentLens` — section/document/heading coverage, word counts, completeness
- `FrameworkFitnessLens` — required vs. present elements, fitness score
- `TimeInvestmentLens` — time spent per section/document, completion projection

Future lenses (register without touching existing code): financial health, narrative coherence, investor readiness, competitive positioning.

**Primary interface:**
```python
def diagnose_business(business_id: UUID) -> BusinessReport
def diagnose_plan(plan_id: UUID, lenses: list[str] | None = None) -> PlanReport
```

`lenses=None` runs all registered lenses. Specific lens names run only those.

**PlanReport:**
```python
@dataclass
class PlanReport:
    plan_title: str
    framework_id: str
    status: PlanStatus
    lens_reports: dict[str, LensReport]   # lens name → report
    errors: list[str]
    warnings: list[str]
    generated_at: datetime
```

**TimeInvestmentLens** produces:
- `time_by_section: dict[str, int]` (slug → seconds)
- `time_by_document: dict[str, int]`
- `avg_time_per_unit: float`
- `estimated_remaining_sec: int`
- `completion_projection: CompletionProjection` (optimistic/realistic/conservative)

Doctor is always safe to run. It makes no writes.

---

## 3. QA Service

**Module:** `newent/services/qa.py`

**Responsibility:** Unified question-and-answer engine. Manages a library of questions with unique IDs. Powers warmup sessions, coach prompts, framework population, and business profiling.

**Interface:**
```python
# Question library
def register_question(text: str, tags: list[str], follow_up: str | None,
                      element_address: str | None, framework_id: str | None) -> Question
def get_question(question_id: UUID) -> Question
def search_questions(tags: list[str] | None, topic: str | None,
                     element_address: str | None) -> list[Question]
def compose_set(question_ids: list[UUID], title: str) -> QuestionSet
def vibe_set(tags: list[str], count: int) -> QuestionSet  # ad-hoc cross-topic set

# Session management
def start_session(business_id: UUID, question_set: QuestionSet,
                  source_context: str) -> QASession
def get_next_question(session: QASession) -> Question | None
def record_answer(session: QASession, question_id: UUID,
                  body: str, started_at: datetime) -> Answer
def complete_session(session: QASession) -> QASession
def resume_session(session_id: UUID) -> QASession
```

**QASession:**
```python
@dataclass
class QASession:
    id: UUID
    business_id: UUID
    question_set: QuestionSet
    source_context: str  # "warmup", "coach", "framework-populate", "profiling"
    status: SessionStatus
    answered: list[Answer]
    current_question_idx: int
```

**Vibed sessions:** `vibe_set(tags=["market", "product"], count=5)` returns a cross-topic set of 5 questions. The user can run an ad-hoc session without committing to a specific warmup set or coach intent.

**Time tracking:** `record_answer` takes `started_at` (when the prompt was shown) and records `submitted_at = now()`. The delta is stored as `time_spent_sec` on the Answer.

---

## 4. New Service

**Module:** `newent/services/new_service.py`

**Responsibility:** Context-sensitive creation at any hierarchy level. Interprets the `new` keyword in any navigation context.

**Interface:**
```python
def resolve_new(context: NavigationContext) -> NewTarget
def create_at(target: NewTarget, content: str | None = None) -> CreatedEntity
def process_inline_sequence(
    context: NavigationContext,
    sequence: list[str | int]     # e.g. [1, 3, "new", 2, 4, "new"]
) -> list[CreatedOrReordered]
```

**NavigationContext** encodes the current address level (business, plan, section, document, heading, content_item). `resolve_new` returns the `NewTarget` — what would be created at this level (a new plan, section, document, etc.).

**Inline sequence processing:** `process_inline_sequence` handles the `1 3 new 2 4 new` syntax. For each `"new"` in the sequence, the service calls QA service to prompt for the new item's content, inserts the created entity at that position, and continues. The result is a reordered + augmented list.

---

## 5. Ent Service

**Module:** `newent/services/ent.py`

**Responsibility:** Meta-contextual lens. Generates a situated view of any element within its plan and company context.

**Interface:**
```python
def situate_business(business_id: UUID) -> EntView
def situate_plan(plan_id: UUID) -> EntView
def situate_section(section_id: UUID) -> EntView
def situate_document(document_id: UUID) -> EntView
def situate_heading(heading_id: UUID) -> EntView
def situate_content_item(content_item_id: UUID) -> EntView
def situate_address(plan_id: UUID, address: str) -> EntView
```

**EntView:**
```python
@dataclass
class EntView:
    element_type: str
    element_summary: str
    parent_context: list[ContextBreadcrumb]     # hierarchy above
    sibling_context: list[SiblingRef]           # same-level siblings
    assigned_notes: list[Note]
    related_warmup: list[WarmupSummaryRef]
    framework_position: str                     # where this falls in the framework
    ai_context: str | None                      # future: AI-generated situating text
```

The service is designed to grow. `ai_context` is `None` in v1 (Cloud/AI not yet connected). When Cloud is available, Ent passes the structured view to a babb AI endpoint and returns the enriched result.

---

## 6. Coach Service

**Module:** `newent/services/coach.py`

**Responsibility:** Guided workflow through develop and document channels. Manages sessions, loads hints, delegates Q&A to QA service.

**Interface:**
```python
def start_session(business_id: UUID, plan_id: UUID,
                  intent: CoachIntent, voice: CoachVoice) -> CoachSession
def resume_session(session_id: UUID) -> CoachSession
def get_prompt(session: CoachSession, heading_id: UUID) -> CoachPrompt
def advance(session: CoachSession) -> CoachSession | None
```

**CoachPrompt** is assembled by loading the hint YAML for the current heading via Framework service, then calling QA service to register (or retrieve) the question for that heading and return the appropriately voice-filtered output.

```python
@dataclass
class CoachPrompt:
    question: Question          # from QA service
    quiet_label: str
    partial_hint: str | None
    full_context: str | None
    examples: list[str]
    references: list[Reference]
    voice: CoachVoice
```

When the user responds to the prompt, Coach calls `qa_service.record_answer()` — the answer is stored with timestamps for time tracking. The response body is then written to the plan via Store as a ContentItem.

---

## 7. Warmup Service

**Module:** `newent/services/warmup.py`

**Responsibility:** Informal elicitation sessions. Hangout experience. Delegates Q&A to QA service.

**Interface:**
```python
def get_home(business_id: UUID | None) -> WarmupHome
def list_question_sets(tags: list[str] | None) -> list[QuestionSetMeta]
def start_session(business_id: UUID, question_set_id: str) -> WarmupSession
def resume_session(session_id: UUID) -> WarmupSession
def complete_session(session: WarmupSession) -> Note
def list_sessions(business_id: UUID) -> list[WarmupSessionSummary]
```

**WarmupHome** is the `warmups` landing view:
```python
@dataclass
class WarmupHome:
    available_sets: list[QuestionSetMeta]
    in_progress: list[WarmupSessionSummary]
    completed: list[WarmupSessionSummary]
    suggested_next: QuestionSetMeta | None
```

Warmup does not manage questions itself. It calls `qa_service.compose_set()` to build the question set for a given warmup YAML definition, then calls `qa_service.start_session()`. All question delivery and answer recording goes through QA service.

`complete_session` generates a structured summary Note from the QA session's answers using the question set's summary template.

---

## 8. Develop Service

**Module:** `newent/services/develop.py`

**Responsibility:** Tracks and routes develop-channel progress. Coordinator only — no content generation.

**Interface:**
```python
def get_develop_status(plan_id: UUID) -> DevelopStatus
def suggest_next_intent(plan_id: UUID) -> CoachIntent | None
def get_intent_sections(intent: CoachIntent, framework_id: str) -> list[SectionRef]
```

---

## 9. Document Service

**Module:** `newent/services/document.py`

**Responsibility:** Manages narrate, prove, capital channels. Prepares documents for export.

**Interface:**
```python
def get_document_status(plan_id: UUID, channel: DocumentChannel) -> DocumentStatus
def assemble_document(plan_id: UUID, document_id: UUID) -> AssembledDocument
def list_exportable(plan_id: UUID) -> list[DocumentRef]
```

---

## 10. Notes Service

**Module:** `newent/services/notes.py`

**Responsibility:** CRUD for Notes. Assignment, reordering, display toggle management.

**Interface:**
```python
def add_note(business_id: UUID, body: str, source: NoteSource,
             title: str | None = None) -> Note
def edit_note(note_id: UUID, body: str | None, title: str | None) -> Note
def get_note(note_id: UUID) -> Note
def list_notes(business_id: UUID, status: NoteStatus | None,
               tags: list[str] | None) -> list[Note]
def list_notes_for_target(target: AssignmentTarget) -> list[Note]
def assign_note(note_id: UUID, target: AssignmentTarget) -> Note
def reorder_notes(target: AssignmentTarget, ordered_ids: list[UUID]) -> list[Note]
def set_display(note_id: UUID, display: bool) -> Note
def set_display_global(business_id: UUID, display: bool) -> int  # count affected
def tag_note(note_id: UUID, tags: list[str]) -> Note
def archive_note(note_id: UUID) -> Note
def mark_used(note_id: UUID) -> Note
```

**Prologue behavior:** Notes assigned to plan, section, or document level have `display=True` by default. When displayed, they render as prologues above the element in views and exports. `set_display` toggles per note; `set_display_global` toggles for all notes associated with a business.

Notes with `source=warmup` are auto-titled from the warmup session name. Notes with `source=manual` or `source=imported` are freely editable.

---

## 11. Finance Service

**Module:** `newent/services/finance.py`

**Status:** Reserved stub. Raises `NotImplementedError` with a message. Do not implement until the Finance design session is completed.

---

## 12. Framework Service

**Module:** `newent/services/framework.py`

**Responsibility:** Load, validate, query, and manage framework YAML files. Handle framework installation and updates.

**Interface:**
```python
def list_industry_frameworks() -> list[FrameworkMeta]
def list_audience_frameworks() -> list[FrameworkMeta]
def get_industry_framework(framework_id: str) -> IndustryFramework
def get_audience_framework(framework_id: str) -> AudienceFramework
def get_hints(framework_id: str, section_slug: str, document_slug: str) -> HintSet
def validate_framework_file(path: Path) -> ValidationResult
def scaffold_plan_from_framework(framework_id: str) -> PlanScaffold
def install_framework(source: Path | str) -> FrameworkMeta  # path or URL
def check_updates() -> list[FrameworkUpdate]  # requires Cloud
def apply_update(framework_id: str, update: FrameworkUpdate) -> FrameworkMeta
def populate_via_qa(framework_id: str, field_path: str) -> QASession
```

**Framework YAML schema:** All fields are fully declared. Optional fields are marked inline with `optional: true` or grouped in an `optional_fields:` section. No implicit fields. This makes files self-documenting and enables QA-driven population via `populate_via_qa`.

**`scaffold_plan_from_framework`** returns a `PlanScaffold` (structured list of sections, documents, headings with their addresses and `min_paragraphs` values) that the CLI layer passes to Store to create the initial plan structure.

---

## 13. Export Service

**Module:** `newent/services/export.py`

**Responsibility:** Generate output files from plan content. Apply audience framework overlays. Non-destructive.

**Interface:**
```python
def export_plan(plan_id: UUID, output_dir: Path,
                audience_framework_id: str | None,
                format: ExportFormat) -> ExportResult
def preview_export(plan_id: UUID, audience_framework_id: str | None) -> ExportPreview
```

**ExportFormat:** `markdown` | `json` | `pdf` (pdf requires pandoc)

Audience framework application: filter and reorder sections, apply vocabulary substitutions, write in framework's preferred structure. Source plan is never modified.

---

## 14. Spaces Service

**Module:** `newent/services/spaces.py`

**Responsibility:** Optional global organizational groupings for businesses and plans. Spaces are independent of any workspace directory.

**Interface:**
```python
def list_spaces() -> list[Space]
def create_space(name: str, description: str | None) -> Space
def get_space(name_or_id: str) -> Space | None
def add_business(space_id: UUID, business_id: UUID) -> SpaceMembership
def remove_business(space_id: UUID, business_id: UUID) -> None
def add_plan(space_id: UUID, plan_id: UUID) -> SpaceMembership
def list_members(space_id: UUID) -> list[SpaceMember]
def get_space_summary(space_id: UUID) -> SpaceSummary
```

Spaces are stored in a global location (`~/.newent/spaces.db`) separate from workspace-level `store.db`. When Cloud is connected, spaces sync to the babb platform.

---

## 15. Cloud Service

**Module:** `newent/services/cloud.py`

**Status:** Reserved stub. Not implemented in local v1. Raises `NotImplementedError` directing to the babb roadmap.

When implemented: authentication, workspace sync, collaboration, AI feature enablement (Ent service AI context, QA vibed session AI curation), remote framework registry, framework updates, spaces sync. All other services are unchanged when Cloud activates — they route through Cloud's network layer without modifying their own logic.

---

## 16. Service Communication Pattern

In local v1, services are Python modules called directly. Services are stateless — all state is in Store.

```python
# CLI layer:
from newent.services import doctor, store

business = store.get_business(slug)
report = doctor.diagnose_business(business.id)
```

**Toward babb:** Each import becomes an HTTP client call. Method signatures become request/response schemas. CLI layer does not change structurally. Enforced by the rule: all inputs/outputs must be JSON-serializable dataclasses.
