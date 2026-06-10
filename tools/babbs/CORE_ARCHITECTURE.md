# Babbs Core Architecture

## Fundamental Principles

### 1. Everything is an Event

Like Unix's "everything is a file" principle, Babbs operates on the principle that "everything is an event." Every system operation, from record creation to metadata updates, generates one or more events that are logged and traceable.

```rust
// Example event structure
struct Event {
    id: EventId,           // Unique event identifier
    timestamp: DateTime,    // When the event occurred
    type: EventType,       // Type of event (Create, Update, Link, etc.)
    source: EventSource,   // Origin of the event (User, System, Plugin)
    payload: EventPayload, // Event-specific data
    parent: Option<EventId> // Parent event if part of a chain
}
```

### 2. Event Stream Processing

All operations in Babbs are processed as event streams. A single user command may generate multiple events that flow through the system:

```shell
# User command
babbs t Meeting with client (s:client e:200:usd paid) !! Good discussion

# Generates event stream:
1. CommandReceived(type=TimeEntry)
2. RecordCreated(type=Time)
3. RecordCreated(type=Exchange)
4. RecordCreated(type=Journal)
5. RelationshipCreated(Time->Exchange)
6. RelationshipCreated(Time->Journal)
```

### 3. Modular Architecture

Each component is designed to be independent and composable:

- Event Generators: Create system events
- Event Processors: Transform and validate events
- Event Loggers: Persist events
- Record Managers: Maintain record state
- Query Engine: Search and retrieve records

## System Architecture

### 1. Kernel Layer (Event Handling)

The core event processing system:

```rust
trait EventHandler {
    fn handle(&self, event: Event) -> Result<Vec<Event>, Error>;
}

struct BabbsKernel {
    event_bus: EventBus,
    processors: Vec<Box<dyn EventHandler>>,
    loggers: Vec<Box<dyn EventLogger>>
}
```

Primary responsibilities:
- Event routing and dispatch
- Basic event validation
- Event logging coordination
- Inter-process communication

### 2. System Services Layer

Services that operate on events to maintain system state:

1. Record Manager
   - Creates and updates records
   - Maintains relationships
   - Handles record lifecycle

2. Storage Manager
   - Persists records and events
   - Manages indexes
   - Handles queries

3. Event Logger
   - Maintains separate logs for each event type
   - Provides audit trails
   - Enables event replay

### 3. User Interface Layer

Command processing and user interaction:

```shell
# Command structure generates multiple events
babbs t "Team meeting" >> @due=tomorrow Follow up
│
├─ CommandEvent(TimeEntry)
│  └─ RecordEvent(Create, Time)
│     └─ MetadataEvent(Add, Category=meeting)
│
└─ CommandEvent(ActionEntry)
   └─ RecordEvent(Create, Action)
      ├─ MetadataEvent(Add, Due=tomorrow)
      └─ RelationshipEvent(Link, Time->Action)
```

## Event Logging System

### 1. Event Log Types

Each type of event has its own log:

```plaintext
/var/log/babbs/
├── command.log     # User commands
├── record/
│   ├── time.log    # Time record events
│   ├── action.log  # Action record events
│   ├── journal.log # Journal record events
│   └── exchange.log # Exchange record events
├── metadata.log    # Metadata modifications
└── relation.log    # Relationship changes
```

### 2. Event Log Format

Each log entry contains:
```json
{
    "event_id": "E250419001",
    "timestamp": "250419T1430EST",
    "type": "RecordCreated",
    "source": "UserCommand",
    "parent_event": "C250419001",
    "payload": {
        "record_type": "Time",
        "content": "Team meeting",
        "metadata": {
            "category": "meeting",
            "project": "babbs"
        }
    }
}
```

## Unix-Inspired Design Patterns

### 1. Event Streams and Pipes

Like Unix's pipe operator (|), Babbs events flow through processing chains:

```rust
trait EventProcessor {
    fn process(&self, event: Event) -> Result<Event, Error>;
}

// Example processing chain
struct EventPipeline {
    processors: Vec<Box<dyn EventProcessor>>,
}

// Usage: validate | transform | enrich | store
let pipeline = EventPipeline::new()
    .pipe(ValidateEvent::new())
    .pipe(TransformEvent::new())
    .pipe(EnrichEvent::new())
    .pipe(StoreEvent::new());
```

Example processing chains:
```shell
# Time entry pipeline
Parse -> Validate -> Enrich -> Store
  └─ Extract dates
      └─ Validate format
          └─ Add context
              └─ Persist

# Exchange entry pipeline
Parse -> Validate -> Calculate -> Store
  └─ Extract amount
      └─ Check balance
          └─ Apply rates
              └─ Persist
```

### 2. Standard Event Channels

Following Unix's STDIN/STDOUT/STDERR model:

1. Primary Event Stream (EVOUT)
   ```rust
   pub trait EventStream {
       fn emit(&self, event: Event) -> Result<(), Error>;
       fn subscribe(&self) -> EventSubscriber;
   }
   ```

2. Error Event Stream (EVERR)
   ```rust
   pub trait ErrorStream {
       fn emit_error(&self, error: EventError);
       fn on_error(&self, handler: Box<dyn ErrorHandler>);
   }
   ```

3. Control Event Stream (EVIN)
   ```rust
   pub trait ControlStream {
       fn send_control(&self, cmd: ControlCommand);
       fn receive_control(&self) -> ControlReceiver;
   }
   ```

Example:
```rust
// Component with standard channels
struct BabbsComponent {
    event_out: EventStream,    // Normal events
    event_err: ErrorStream,    // Error events
    control_in: ControlStream, // Control commands
}
```

### 3. Process Isolation

Each major component runs in isolation:

1. Resource Boundaries
   ```rust
   struct ComponentBoundary {
       memory_limit: usize,
       file_descriptors: Vec<FileDescriptor>,
       permissions: Permissions,
   }
   ```

2. State Isolation
   ```rust
   // Each component maintains its own state
   struct ComponentState {
       local_cache: Cache,
       temp_storage: TempStore,
       metrics: Metrics,
   }
   ```

3. Security Domains
   ```rust
   // Components have specific capabilities
   struct ComponentCapabilities {
       can_read: Vec<ResourceType>,
       can_write: Vec<ResourceType>,
       can_execute: Vec<Operation>,
   }
   ```

### 4. Inter-Process Communication

Components communicate through event channels:

```rust
// Example: Time entry processing across components
struct TimeEntryProcessor {
    parser: Component,
    validator: Component,
    storage: Component,
}

impl TimeEntryProcessor {
    fn process(&self, input: String) {
        // Components communicate via events
        self.parser.process(input)
            .pipe_to(&self.validator)
            .pipe_to(&self.storage);
    }
}
```

Example of complete session lifecycle:
```shell
# Start new session
$ babbs new

Last Session Details (S250418012):
Operator: John Smith
Purpose: Weekly accounting
Company: Acme Corp
Status: Completed at 250418T1700EST

Continue with these details? [y/n]: n

New Session Setup:
Operator: Jane Doe
Purpose: Project planning
Company: TechCo

[S250419001] Session initialized
Ready for input...

# Create time entry
│       └─ Emits ParsedEvent
│
├─ Validator Component
│   └─ Validates syntax and rules
│       └─ Emits ValidatedEvent
│
├─ Record Creator Component
│   ├─ Creates Time record
│   └─ Creates Exchange record
│       └─ Emits CreatedEvent
│
└─ Storage Component
    └─ Persists records
        └─ Emits StoredEvent
```

## Session Management

### 1. Session Initialization

When `babbs new` is called, it first displays the last session details and prompts for session creation:

```rust
struct Session {
    id: SessionId,           // Format: S<YYMMDD><001-999>
    operator: String,        // Name of the session operator
    purpose: String,         // Purpose of the session
    company: String,         // Company context
    start_time: DateTime,    // Session start timestamp
    active: bool            // Session state
}
```

Example terminal interaction:
```shell
$ babbs new

Last Session Details (S250418012):
Operator: John Smith
Purpose: Weekly accounting
Company: Acme Corp
Status: Completed at 250418T1700EST

Continue with these details? [y/n]: n

New Session Setup:
Operator: Jane Doe
Purpose: Project planning
Company: TechCo

[S250419001] Session initialized
Ready for input...
```

The initialization process:
```shell
# User initiates new session
$ babbs new

# 1. Load and display last session
LastSessionEvent(S250418012)
   └─ Display last session metadata

# 2. Session initialization
SessionInitEvent(S250419001)
   ├─ Collect session metadata
   │  ├─ Operator: "Jane Doe"
   │  ├─ Purpose: "Project planning"
   │  └─ Company: "TechCo"
   │
   └─ Create session files
      ├─ .babbs/sessions/S250419001/
      └─ session.log

2. SessionStartEvent
   ├─ Load user preferences
   ├─ Set up event streams
   └─ Initialize component states

3. SessionReadyEvent
   └─ Session ready for commands
```

### 2. Session Event Structure

Each session event includes the core session metadata:

```json
{
    "session_id": "S250419001",
    "event_id": "E250419001",
    "type": "SessionCommand",
    "timestamp": "250419T1430EST",
    "session_context": {
        "operator": "Jane Doe",
        "purpose": "Project planning",
        "company": "TechCo"
    },
    "command": {
        "type": "TimeEntry",
        "raw_input": "babbs t Meeting discussion"
    }
}
```

Session metadata is automatically included in all records:
```shell
# Time entry example
T250419001 @operator="Jane Doe" @company="TechCo" Team sync meeting

# Exchange record example
E250419002 @operator="Jane Doe" @company="TechCo" e:200:usd Consulting fee
```

Command chaining within session:
```shell
# Session with multiple commands
$ babbs new
SessionInit(S250419001)

$ babbs t Team meeting
├─ SessionEvent(S250419001, command=TimeEntry)
└─ TimeRecord(T250419001)

$ babbs a Follow up
├─ SessionEvent(S250419001, command=ActionEntry)
└─ ActionRecord(A250419002)
   └─ LinkEvent(A250419002 -> T250419001)
```

### 3. Session Lifecycle

1. Session Start
   ```rust
   impl Session {
       fn new() -> Result<Session, Error> {
           // Generate session ID
           let id = generate_session_id()?;
           
           // Initialize session context
           let context = SessionContext::new()?;
           
           // Set up event streams
           let event_streams = setup_event_streams(id)?;
           
           // Create session log
           let logger = SessionLogger::new(id)?;
           
           Ok(Session { id, context, event_streams, logger })
       }
   }
   ```

2. Command Processing
   ```rust
   impl Session {
       fn process_command(&self, cmd: Command) -> Result<(), Error> {
           // Log command in session context
           self.logger.log_command(cmd);
           
           // Process through event pipeline
           let events = self.process_pipeline.handle(cmd)?;
           
           // Update session state
           self.update_context(&events)?;
           
           Ok(())
       }
   }
   ```

3. Session Recovery
   ```rust
   impl Session {
       fn recover(session_id: SessionId) -> Result<Session, Error> {
           // Load session log
           let log = SessionLog::load(session_id)?;
           
           // Replay events to reconstruct state
           let session = Session::new()?;
           session.replay_events(log.events())?;
           
           Ok(session)
       }
   }
   ```

Example of complete session lifecycle:
```shell
# Start new session
$ babbs new
[S250419001] Session initialized

# Create time entry
$ babbs t Meeting with team
[S250419001-E001] Command received
[S250419001-E002] TimeRecord created (T250419001)

# Add action from meeting
$ babbs a Follow up email
[S250419001-E003] Command received
[S250419001-E004] ActionRecord created (A250419002)
[S250419001-E005] Relationship created (A250419002 -> T250419001)

# Add exchange record
$ babbs t (s:client e:200:usd received)
[S250419001-E006] Command received
[S250419001-E007] TimeRecord created (T250419003)
[S250419001-E008] ExchangeRecord created (E250419004)
[S250419001-E009] Relationship created (E250419004 -> T250419003)

# Review session log
$ babbs log
[S250419001] Session events:
  ├─ E001: TimeEntry command
  ├─ E002: Time record creation
  ├─ E003: ActionEntry command
  ├─ E004: Action record creation
  ├─ E005: Record relationship
  ├─ E006: TimeEntry command
  ├─ E007: Time record creation
  ├─ E008: Exchange record creation
  └─ E009: Record relationship
```

## Implementation Principles

### 1. Error Handling

- Every event must be acknowledged
- Failed events are logged separately
- System maintains consistency through event replay

### 2. Extensibility

- Plugin system based on event handlers
- Custom event types can be registered
- Event processors can be chained

### 3. Performance

- Event batching for efficiency
- Asynchronous event processing
- Efficient storage and indexing

## Example: Complex Command Processing

```shell
# User command
babbs t Meeting with client (s:client e:200:usd paid) !! Good discussion

# Event Generation Sequence
1. CommandEvent
   └─ Parse command text
   
2. TimeRecordEvent
   └─ Create time record
      └─ Generate record ID
      
3. ExchangeRecordEvent
   └─ Create exchange record
      ├─ Parse amount and currency
      └─ Link to time record
      
4. JournalRecordEvent
   └─ Create journal entry
      └─ Link to time record

5. RelationshipEvents
   ├─ Time->Exchange
   └─ Time->Journal

# Each event is logged separately while maintaining relationship chains
```

This architecture ensures:
- Complete audit trail
- Data consistency
- System extensibility
- Complex operation support

