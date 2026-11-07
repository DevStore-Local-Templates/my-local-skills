---
name: phase-2-ddd-bounded-context-analysis
description: Identify bounded contexts, aggregates, entities, value objects, domain events, and context maps from the consolidated business function analysis. Use when performing DDD analysis, defining microservice boundaries, or designing domain models.
---

# Phase 2: DDD Bounded Context Analysis

## Purpose

This skill guides the identification of bounded contexts and their associated domain objects from the consolidated business function analysis document produced in Phase 1.

## Input Sources

The **primary input** for this analysis is:

- **`ddd-analysis.md`** — The consolidated analysis document that synthesizes all business function requirements, program inventories, data store access patterns, and cross-function dependencies into a single reference. This document preserves full traceability to the original sources.

If additional detail is needed beyond what `ddd-analysis.md` provides, the original sources can be consulted:

1. **Business Requirements** — `inputs/spec/<FunctionName>/requirements.md`
2. **System Data Model** — `inputs/spec/data-model.md` (data stores, types, accessing capabilities)
3. **Domain Glossary** — `inputs/spec/.glossary.yaml` (data store and field descriptions)
4. **Shared Capabilities** — `inputs/spec/_shared/<ProgramName>/requirements.md`

## Analysis Process

Follow these steps sequentially. Each step builds on the previous one.

### Step 0: Discover Business Functions

Before starting the DDD analysis, discover all available business functions:

1. List all subdirectories under `inputs/spec/` (excluding `_shared/` and dot-files)
2. Each subdirectory name represents a business function
3. Verify each function has a `requirements.md`
4. Read `inputs/spec/data-model.md` to identify which data stores each function accesses
5. Inventory shared capabilities from `inputs/spec/_shared/` — read each shared program's requirements.md to understand their purpose
6. Build the inventory of functions to analyze

**Output format:**
```markdown
## Business Functions Inventory

| # | Function Name | Has Requirements | Data Stores Accessed |
|---|--------------|-----------------|---------------------|
| 1 | <directory name> | Yes/No | <from data-model.md Accessed By> |
```

### Step 1: Identify Bounded Contexts

For each discovered business function, analyze:
- The key capabilities described in its `requirements.md`
- The data stores it accesses (from `data-model.md`)
- The cross-boundary requirements (REQ-XBND-*, REQ-INTEG-*) that reveal integration points
- The shared capability dependencies that reveal cross-function program sharing
- Data stores accessed by multiple functions (from data-model.md) that reveal cross-function data dependencies

**Criteria for grouping into bounded contexts:**
- **Cohesion**: Functions that share data stores and change together belong in the same context
- **Autonomy**: A context should be independently deployable
- **Data Ownership**: Functions that are the primary accessor of specific data stores strongly indicate a context boundary. Use data-model.md to determine which functions access which stores.
- **Business Alignment**: Group by business capability, not by technical similarity
- **Language Consistency**: Terms used consistently within a function group define the ubiquitous language

**Classification:**
- **Core Domain**: Business functions that provide competitive advantage
- **Supporting Subdomain**: Functions that support the core but are not differentiating
- **Generic Subdomain**: Cross-cutting infrastructure concerns

**Output format for each bounded context:**
```markdown
### <Context Name>

- **Responsibility**: <one-sentence description>
- **Classification**: Core Domain | Supporting Subdomain | Generic Subdomain
- **Source Functions**: <list of business functions that map to this context>
- **Primary Data Stores**: <data stores primarily accessed by this context's functions, from data-model.md>
- **Shared Data Stores**: <data stores shared with other contexts>
- **Key Concepts** (ubiquitous language preview):
  - <Term 1>: <brief definition>
  - <Term 2>: <brief definition>
```

### Step 2: Define Ubiquitous Language

For each bounded context identified in Step 1:
- Extract domain terms from the requirements (field names, entity names, operation names)
- Review field and data store descriptions from .glossary.yaml
- Identify business concepts vs technical/legacy terms
- Define each term in business language
- Note context-specific meanings (same term may mean different things in different contexts)

**Output format:**
```markdown
| Term | Business Definition | Context-Specific Meaning | Related Terms |
|------|-------------------|-------------------------|---------------|
```

### Step 3: Identify Aggregates and Aggregate Roots

For each bounded context:
- Identify candidate entities from the data stores accessed by the context's functions (from data-model.md)
- Determine transactional boundaries from the requirements (look for REQ-LOCK-*, REQ-INTEG-* patterns)
- Identify consistency requirements from business rules described in requirements
- Group entities that must change together atomically
- Select the aggregate root (the entity with the primary identity that controls access)

**Aggregate root selection criteria:**
- Has a unique identifier (inferred from data store type and naming — KSDS stores are keyed)
- Manages lifecycle of contained entities
- Enforces business rules across the aggregate
- All external access goes through it
- Changes are atomic within the aggregate boundary

**Output format:**
```markdown
### <Aggregate Name>

- **Aggregate Root**: <Entity name>
  - Source: <data store DSN from data-model.md>
  - Identity: <primary key field, inferred from store type and glossary>
- **Contained Entities**: <list>
- **Invariants**:
  - <business rule enforced by this aggregate> (<FunctionName>::REQ-XXX-NNN)
```

### Step 4: Define Entities

For each aggregate, define the entities within it:
- Use data-model.md for data store identification and types
- Use .glossary.yaml for field descriptions
- Use requirements for field-level detail (requirements often describe specific fields, their formats, and constraints)
- Define entity operations from the requirements

**Output format:**
```markdown
### <Entity Name>

- **Source**: <data store DSN from data-model.md>
- **Type**: <VSAM KSDS, QSAM, etc.>
- **Identity**: <key field inferred from store type and glossary descriptions>
- **Key Fields** (from glossary and requirements):
  | Field | Description | Inferred Type |
  |-------|-------------|---------------|
- **Operations**: <list of methods derived from requirements>
- **Lifecycle**: <creation → states → termination>
```

### Step 5: Define Value Objects

Identify attributes that should be modeled as value objects:
- Composite values (e.g., Address = line1 + line2 + line3 + state + country + zip — inferred from field groupings in requirements)
- Values with validation rules (from REQ-VALID-* requirements)
- Values with formatting requirements (e.g., dates, phone numbers, monetary amounts — from glossary descriptions)
- Domain concepts without identity (e.g., Money, DateRange, CreditScore)

**Output format:**
```markdown
### <Value Object Name>

- **Source**: <derived from which data store / requirements>
- **Attributes**:
  | Attribute | Description | Inferred Type |
  |-----------|-------------|---------------|
- **Validation Rules**: <from requirements>
- **Operations**: <formatting, comparison, etc.>
- **Immutability**: Guaranteed — new instance created for any change
```

### Step 6: Identify Domain Events

Look for state changes and integration points:
- Aggregate creation/update/deletion patterns in requirements
- Cross-boundary requirements (REQ-XBND-*, REQ-INTEG-*) that indicate events needed between contexts
- Data stores accessed by multiple functions (from data-model.md) that indicate cross-function state changes
- Audit requirements
- Notification patterns (message queue interactions identified from requirements mentioning MQ operations)

**Output format:**
```markdown
### <EventName> (past tense)

- **Trigger**: <what causes this event> (<FunctionName>::REQ-XXX-NNN)
- **Source Context**: <bounded context that publishes>
- **Payload**:
  - aggregateId: <type>
  - <relevant state change fields>
  - occurredAt: timestamp
- **Subscribers**: <list of consuming contexts>
```

### Step 7: Define Domain Services

Identify operations that span multiple entities or aggregates:
- Operations involving multiple data stores (inferred from requirements describing reads/writes to multiple files)
- Complex validations across entities (from requirements)
- Calculations using multiple data sources
- Coordination logic from requirements
- **Shared capability business logic** (from `_shared/<ProgramName>/requirements.md`) — these represent reusable domain logic invoked by multiple functions and should be modeled as domain services or embedded in the consuming aggregate's invariants

**Output format:**
```markdown
### <Service Name>

- **Responsibility**: <what it does>
- **Operations**:
  - <operation name>(<inputs>) → <output>
- **Dependencies**: <aggregates/entities it works with>
```

### Step 8: Define Application Services (Use Cases)

Map each major workflow from requirements.md to a use case:
- Each numbered section in requirements.md typically maps to one use case
- Include the step-by-step flow with embedded business rules
- Use program names mentioned in requirements to identify which legacy programs implement each use case

**Output format:**
```markdown
### <Use Case Name>

- **Actor**: <who triggers this>
- **Input**: <command/request structure>
- **Output**: <result/response structure>
- **Flow**:
  1. <step with business rule reference> (<FunctionName>::REQ-XXX-NNN)
  2. <step> (<FunctionName>::REQ-XXX-NNN)
- **Error Handling**:
  - <condition> → <error response> (<FunctionName>::REQ-XXX-NNN)
- **Legacy Programs**: <program names mentioned in requirements that implement this use case>
```

### Step 9: Define Context Mapping

Identify relationships between bounded contexts:
- Use data-model.md to identify data stores shared across functions (shared access patterns)
- Use cross-boundary requirements to identify integration patterns
- Determine upstream/downstream relationships based on function purposes and data flow
- Use program call patterns from requirements to identify runtime dependencies

**Relationship patterns to look for:**
- Data stores accessed by multiple functions → potential Shared Kernel or Customer-Supplier
- Read-only access to another context's data → Anti-Corruption Layer
- Message queue interactions (MQ calls identified from requirements) → Published Language / Open Host Service
- No shared data stores → independent contexts

**Output format:**
```markdown
### Context Map

| Upstream Context | Downstream Context | Pattern | Shared Concept | Integration Mechanism |
|-----------------|-------------------|---------|----------------|----------------------|
```

## Output Structure

The final DDD analysis document should follow this structure:

1. **Executive Summary** — Artifact counts, context overview, key decisions
2. **Business Functions Inventory** — Discovered functions and their data store associations
3. **Bounded Contexts** — Each context with responsibility, classification, language, and aggregates
4. **Aggregate Definitions** — Roots, entities, value objects
5. **Domain Events** — Grouped by context with triggers and subscribers
6. **Domain Services** — Stateless operations spanning aggregates
7. **Application Services** — Use cases with flows and business rules
8. **Context Map** — Relationships and integration patterns

## Key Principles

- **Qualify REQ-* identifiers with function name** — Because different business functions can have the same REQ-* identifiers, always prefix them with the source function name using the format `<FunctionName>::REQ-XXX-NNN` (e.g., `AccountManagement::REQ-F-001`). This ensures unambiguous traceability across bounded contexts.
- Start with discovery — never assume which business functions exist; always scan `inputs/spec/` first
- Keep aggregates small — prefer multiple small aggregates over one large one
- Reference other aggregates by ID only
- Use eventual consistency between aggregates
- One transaction = one aggregate modification
- Data stores accessed by multiple functions (from data-model.md) indicate cross-context dependencies
- Shared capabilities (from `_shared/`) represent cross-cutting logic — consider whether they form a Generic Subdomain or should be distributed across contexts.
- Program names come from requirements documents — these are the legacy COBOL program names
- Data store access patterns come from data-model.md — this is the authoritative source for which functions access which stores
- Field-level detail must be inferred from glossary descriptions and requirement descriptions since no separate data-stores.yaml field dictionaries are available
