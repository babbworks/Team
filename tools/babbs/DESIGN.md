# Technical Design: Record Generation

## Core Record Types

babbs recognizes four fundamental record types, each with a unique single-letter identifier:

- T (Time): Temporal events and durations
- A (Action): Tasks, plans, and executed activities
- J (Journal): Narrative entries and thoughts
- E (Exchange): Financial and value transactions

### Record Identity

Every record carries a type prefix in its identifier:
```
T250419.001  # A Time record
A250419.002  # An Action record
J250419.003  # A Journal record
E250419.004  # An Exchange record
```

### Record Sequencing

Records are sequenced with a three-digit number that:
- Resets to 001 at the start of each month
- Increments continuously within the month
- Maintains strict chronological order

Example sequence across days:
```
T250401.001  # First record of April
T250401.002  # Second record of April
T250402.003  # Third record of April (next day)
...
T250430.167  # Last record of April
T250501.001  # First record of May
```

### Time Handling

1. Time Notation
   ```shell
   t:1430:EST     # 24-hour format with timezone
   t:2:30pm:EST   # 12-hour format with am/pm
   ```

2. Date References
   ```shell
   d:250419              # Explicit date (YYMMDD)
   d:today               # Current date
   d:tomorrow            # Next day
   d:yesterday          # Previous day
   ```

3. Date and Time Together
   ```shell
   d:250419 t:1430:EST    # Explicit date and time
   d:today t:1430:EST     # Current date with time
   d:tomorrow t:0900:EST  # Next day with time
   d:250419 t:2:30pm:EST  # Same time, 12-hour format
   ```

4. Midnight Boundary Handling
   ```shell
   # Late night meeting spanning midnight
   babbs t Meeting discussion >> @due=250422 Prepare presentation
   
   # Multiple actions
   babbs t Client call
     >> @due=d:250419 t:1700:EST Follow up with email
     >> @due=d:250426 t:1200:EST Schedule follow-up meeting
     d:250420 t:0100:EST !! Maintenance completed
     >> @due=d:250420 t:0900:EST Verify system status
   ```

## Record Generation Principles

### Base Record Structure
```
<type><timestamp>.<sequence> [metadata] content
```

Example:
```
T250419.001 @category=meeting @duration=1h Meeting with team
A250419.002 @due=250420 @priority=high Prepare quarterly report
J250419.003 @project=babbs Initial design thoughts
E250419.004 @currency=USD @amount=150.00 Client payment received
```

### Time Record Syntax

#### Basic Command Structure
```shell
babbs t [recorder:] [content...]

# Default context (uses system user as recorder)
babbs t Meeting with engineering team

# Override recorder context
babbs t john: Meeting with marketing team
```

#### Special Prefix Syntax
1. Tags (@)
   ```shell
   babbs t Meeting discussion >> @due=d:250422 t:1700:EST Prepare presentation
   ```

2. Participant Attribution
   ```shell
   # s: indicates sender, r: indicates receiver
   babbs t s:alice r:bob Discussed project timeline
   
   # When only one party is specified, recorder is assumed as counterparty
   babbs t r:client Received project requirements
   ```

3. Exchange and Ledger Entries
   ```shell
   # Liability (l:) - for pending or debt obligations
   babbs t r:vendor l:82 Ordered supplies
   babbs t s:client l:500:usd Invoice sent
   babbs t r:farm l:goats:3 Arranged livestock purchase

   # Exchange (e:) - for completed transactions
   babbs t r:vendor e:82 Paid for supplies
   babbs t s:client e:500:usd Payment received
   babbs t r:farm e:goats:3 Livestock delivered

   # Date Attribution (d:) - for specific exchange dates
   babbs t s:client d:250420 t:1500:EST e:500:usd Payment for last month's work
   ```

4. Transaction Summarization (using brackets)
   ```shell
   # Simple transaction summary
   babbs t Met with Sarah (s:sarah d:250418 t:1400:EST e:250:usd consulting fee)

   # Multiple transactions in context
   babbs t Farm visit with Bob 
     (r:bob d:250420 t:1000:EST l:tractor:1 down payment due) 
     (s:bob d:250419 t:2pm:EST e:eggs:12 fresh delivery)
     >> @due=d:250426 t:1700:EST Follow up about tractor payment

   # Mixed date contexts
   babbs t Monthly review with client 
     (s:client d:250415 t:1400:EST e:2000:usd March invoice)
     (s:client d:250501 t:0900:EST l:2000:usd April invoice)
     !! Discussed upcoming project phases

   # Midnight spanning events
   babbs t Night shift handover
     (r:team d:250419 t:2330:EST e:shift:1 Current shift ending)
     (r:team d:250420 t:0030:EST e:shift:1 New shift starting)
     !! Discussed ongoing maintenance tasks
   ```

#### Embedded Record Generation

1. Action Generation (using >>)
   ```shell
   # Generate action from time entry
   babbs t Meeting discussion >> @due=250422 Prepare presentation
   
   # Multiple actions
   babbs t Client call
     >> @due=today Follow up with email
     >> @due=next-week Schedule follow-up meeting
   ```

2. Journal Integration (using !!)
   ```shell
   # Add journal entry with time record
   babbs t Team sync !! Initial thoughts and concerns
   
   # Multiple journal entries
   babbs t Project review
     !! Initial feedback positive
     !! Need to address performance issues
   ```

3. Exchange Record Generation
   ```shell
   # Basic exchange with journal note
   babbs t s:client l:200:usd !! Payment for consulting
   
   # Exchange with multiple items and actions
   babbs t r:supplier 
     l:parts:50
     l:labor:2h
     l:180:usd
     !! Maintenance service completed
     >> @due=d:250420 t:1700:EST Send receipt
   ```

#### Complex Examples

1. Meeting with Multiple Outcomes
```shell
babbs t @meeting s:team r:client Product review
  >> @due=d:250420 t:0900:EST Send status report
  >> @due=d:250426 t:0900:EST Begin Phase 2
  !! Client expressed concerns about timeline
  (d:250419 t:1400:EST l:widgets:5 Order confirmed)
  (d:250419 t:1500:EST l:service:2h Installation scheduled)
  (d:250425 t:1700:EST l:750:usd Payment due)
  >> @due=today Send invoice
  >> @due=1week Follow up if unpaid
  !! New customer, seemed satisfied
```

4. Project Milestone
```shell
babbs t @milestone @project=alpha r:stakeholders
  !! Phase 1 complete, all tests passing
  >> @due=tomorrow Send status report
  >> @due=next-week Begin Phase 2
  l:5000:usd !! Milestone payment triggered
```

### Record Relationships

Records can reference each other using their identifiers:
```
A250419.002 @from=T250419.001 Action items from meeting
J250419.003 @refers=A250419.002 Thoughts about the report
```

## Metadata Framework

### Core Metadata Fields
1. Temporal metadata
   - timestamp: Creation time
   - duration: Time span (if applicable)
   - due: Future target (if applicable)

2. Relationship metadata
   - from: Source record
   - refers: Referenced record
   - spawns: Child record

3. Contextual metadata
   - category: General classification
   - tags: Free-form labels
   - project: Project association

### Metadata Inheritance
- Child records inherit parent's project context
- Actions spawned from Time entries inherit category
- Exchange records can inherit from related Actions

## Record Creation Pathways

### Direct Creation
```shell
babbs t "Team meeting discussion"  # Creates Time record
babbs a "Follow up with client"    # Creates Action record
babbs j "Project ideas"            # Creates Journal record
babbs e "Invoice payment"          # Creates Exchange record
```

### Record Spawning
- Time record can spawn multiple Actions
- Action can generate related Exchange records
- Journal can reference any other record type

## Extension Points

### Core Extension Mechanisms
1. Custom metadata fields
   ```
   @custom.field=value
   ```

2. Record type relationships
   ```
   @links=T250419.001,A250419.002
   ```

3. Content processors
   ```
   @format=markdown
   @encrypt=true
   ```

### Plugin Architecture
- Plugins register for specific record types
- Metadata processors are chainable
- Content handlers are pluggable
- Storage backends are exchangeable

## Record Lifecycle

1. Creation
   - Generate unique identifier
   - Apply default metadata
   - Process content

2. Validation
   - Verify references
   - Check metadata format
   - Validate content structure

3. Storage
   - Record is immutable once stored
   - Metadata is indexed
   - References are tracked

4. Retrieval
   - By identifier
   - By metadata query
   - By content search

## Implementation Guidelines

1. Command Interface
   - Single letter commands for core types
   - Consistent flag pattern
   - Composable operations

2. Data Format
   - Plain text by default
   - UTF-8 encoding
   - Line-oriented format

3. Storage
   - Append-only records
   - Index-driven queries
   - Atomic operations

4. Extension API
   - Clear interfaces
   - Versioned schemas
   - Documented behaviors

