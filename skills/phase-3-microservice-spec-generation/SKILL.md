---
name: phase-3-microservice-spec-generation
description: Generate detailed microservice specification documents from a completed DDD bounded context analysis. Use when creating service specifications, defining APIs, event contracts, data ownership, and use cases for each microservice.
---

# Phase 3: Microservice Specification Generation

## Purpose

This skill guides the generation of microservice specifications from a completed DDD bounded context analysis produced in Phase 2.

## Input Sources

This process requires two categories of inputs:

### 1. DDD Analysis Output (Primary Input)

The completed DDD bounded context analysis document produced by Phase 2. This provides:
- Bounded contexts with responsibilities and classifications
- Aggregates, entities, and value objects
- Domain events with triggers and subscribers
- Domain services and application services (use cases)
- Context map with integration patterns

### 2. Original Business Requirements (Reference Input)

Located under `inputs/spec/`:
- **Requirements** — `inputs/spec/<FunctionName>/requirements.md` for detailed use case flows and business rules
- **Shared Capabilities** — `inputs/spec/_shared/<ProgramName>/requirements.md` for requirements of shared subroutines
- **System Data Model** — `inputs/spec/data-model.md` for cross-capability data store access patterns
- **Domain Glossary** — `inputs/spec/.glossary.yaml` for data store and field descriptions

## Output Rules

**MANDATORY**: Each identified microservice MUST have its own dedicated specification file.
- Do NOT combine multiple microservice specifications into a single document
- If a bounded context is decomposed into N services, produce N separate files
- File naming convention: `<service-name>-specification.md`
- Output location: a dedicated output folder (e.g., `outputs/microservices/`)

## Execution Model: One Service at a Time

**CRITICAL**: Do NOT generate all microservice specifications in a single pass. Process **one service at a time** to ensure maximum depth and traceability.

**Execution sequence:**
1. Complete Step 1 (identify all microservices from bounded contexts) — this produces the full inventory
2. For EACH service in the inventory, execute Steps 2–10 completely before moving to the next service
3. For each service, read the relevant source artifacts BEFORE generating the specification (see Mandatory Source Reading below)

**Why:** Generating all services at once causes context pressure that leads to shallow specifications, merged services, and missing traceability. Processing one service at a time ensures each specification gets full attention and complete source cross-referencing.

---

## Mandatory Source Reading Per Service

**BEFORE generating each service specification**, you MUST read the following source artifacts for every business function mapped to that service's bounded context:

### Required Reads (Non-Negotiable)

For each `<FunctionName>` mapped to the service:

1. **`inputs/spec/<FunctionName>/requirements.md`** — Read completely. Extract ALL REQ-* identifiers relevant to this service's use cases.

### Required Reads (For Completeness)

2. **`inputs/spec/data-model.md`** — Read to identify data stores accessed by this service's functions and understand cross-function data sharing.
3. **`inputs/spec/.glossary.yaml`** — Read to get data store and field descriptions for the service's data stores.
4. **`inputs/spec/_shared/<ProgramName>/requirements.md`** — For each shared program referenced by this function's programs (identified by CALL references in requirements or cross-references in requirements), read its requirements to understand the shared subroutine's contract and REQ-* identifiers.

### What to Extract

From each source file, extract and keep available while generating the specification:
- Every `REQ-*` identifier and its description — always qualify with the function name as `<FunctionName>::REQ-XXX-NNN`
- Every legacy program name (mentioned in requirements)
- Data store names and types accessed by this service (from data-model.md)
- Data store descriptions (from .glossary.yaml)
- Field-level details inferred from requirements (requirements often describe specific fields, their formats, and constraints)
- Shared capability requirements from `inputs/spec/_shared/<ProgramName>/requirements.md`

**CRITICAL — NO RANGE SHORTHAND**: When listing REQ-* identifiers anywhere in the specification (Source lines, flow steps, validation rules, event triggers, etc.), you MUST enumerate EVERY individual REQ-* ID explicitly. NEVER use range shorthand such as "REQ-F-013 through REQ-F-038" or "REQ-F-001 to REQ-F-010". Instead, list each one: `REQ-F-013, REQ-F-014, REQ-F-015, ..., REQ-F-038`. This is mandatory because Phase 4 traceability verification matches individual REQ IDs — any requirement expressed only as part of a range will be missed and flagged as untraced.

**FAILURE MODE**: If you skip these reads, the specification will lack traceability (no REQ-* references, no program names) and will be too shallow for implementation. This is the #1 quality differentiator.

---

## Generation Process

Follow these steps sequentially for each microservice identified.

### Step 1: Identify Microservices from Bounded Contexts

Analyze each bounded context from the DDD analysis and determine the service decomposition:

**Mapping patterns:**

| Pattern | When to Use |
|---------|-------------|
| 1 BC → 1 Service | Small, cohesive domain with a single team owning it |
| 1 BC → N Services | Large bounded context that can be decomposed further based on distinct sub-capabilities |

**Decomposition criteria:**
- **Business Capability**: Each service owns a complete business capability
- **Autonomy**: Can be developed, deployed, and scaled independently
- **Data Ownership**: Clear ownership of data entities within bounded context only
- **Team Size**: Can be owned by a single team (2-pizza rule)
- **Cohesion**: Operations within the service change together

**Classification:**
- **Core Domain**: High business value, competitive advantage
- **Supporting Subdomain**: Necessary but not differentiating
- **Generic Subdomain**: Common functionality, consider off-the-shelf

**Output format:**
```markdown
## Microservice Inventory

| # | Service Name | Bounded Context | Mapping Pattern | Domain Type | Key Capabilities |
|---|-------------|----------------|-----------------|-------------|-----------------|
| 1 | <name> | <BC name> | 1:1 or 1:N | Core/Supporting/Generic | <capabilities> |
```

### Step 2: Generate Service Overview

For each microservice, define:

```markdown
## Service Overview

- **Service Name**: <clear, business-oriented name>
- **Bounded Context**: <from DDD analysis>
- **Domain Type**: Core Domain | Supporting Subdomain | Generic Subdomain
- **Purpose**: <2-3 sentence description of what this service does>
- **Business Capabilities**:
  - <capability 1>
  - <capability 2>
- **Team Ownership**: <suggested team>
```

### Step 3: Define Service Boundaries

**MANDATORY rules:**
- List ONLY what the service owns within its specific bounded context
- Explicitly list what the service does NOT own
- Identify dependencies on other microservices

```markdown
## Service Boundaries

### Owns
- <aggregate/entity from this bounded context>

### Does NOT Own (belongs to other contexts)
- <entity> → owned by <other service/context>

### Dependencies
- <other microservice>: <what is needed and why>
```

### Step 4: Define Data Ownership

Map aggregates from the DDD analysis to this service's data model:

**CRITICAL RULES:**
- Only include aggregates, entities, and value objects from this specific bounded context
- No cross-context data — do not include entities belonging to other bounded contexts
- Single database per service — no shared databases
- External data access must be through service calls, not direct database access
- **Value objects MUST be fully specified** — not just listed in a summary table. Each value object requires its own dedicated section with attributes, validation rules (with <FunctionName>::REQ-* references), formatting requirements, and immutability guarantees.

```markdown
## Data Ownership

### Domain Model

#### <Aggregate Name>
- **Aggregate Root**: <entity>
- **Identity**: <field> (<type>)
- **Source Data Store**: <data store DSN from data-model.md>
- **Data Store Type**: <VSAM KSDS, QSAM, etc.>
- **Entities**:
  | Entity | Key Field | Source Data Store | Description |
  |--------|-----------|-------------------|-------------|

**<Entity Name> — Key Fields** (inferred from glossary and requirements):

| Domain Attribute | Description | Inferred Type | Source |
|-----------------|-------------|---------------|--------|

**Value Objects**:

| Value Object | Attributes | Validation |
|-------------|-----------|------------|
| <VO Name> (VO-N) | <attribute list> | <validation summary with <FunctionName>::REQ-*> |
```

#### Value Object Detailed Specifications

**MANDATORY**: Each value object identified in the aggregate MUST have a fully expanded specification below the summary table. Do NOT just list value objects — specify them completely.

For each value object, provide:

```markdown
#### <Value Object Name> (VO-N)

- **Source**: <derived from which data store / requirements>
- **Attributes**:
  | Attribute | Description | Inferred Type |
  |-----------|-------------|---------------|
- **Validation Rules**:
  - <Rule description> (<FunctionName>::REQ-* identifier)
  - <Rule description> (<FunctionName>::REQ-* identifier)
- **Format**: <expected format, e.g., YYYY-MM-DD, (AAA) PPP-LLLL>
- **Immutability**: Guaranteed — new instance created for any change
```

Use the DDD analysis field information and reference glossary descriptions and requirements for field-level detail.

### Step 5: Define API Specification

Design REST API endpoints for each use case in this bounded context:

**MANDATORY requirements:**
- ALL use cases MUST have corresponding API endpoints
- ALL API endpoints MUST have complete detailed specifications
- When listing source REQ-* identifiers for an endpoint, enumerate EACH ID individually — NEVER use "through" or range notation

```markdown
## API Specification

### <Endpoint Name>

- **Method**: GET | POST | PUT | PATCH | DELETE
- **Path**: `/api/v1/<resource>/{id}`
- **Description**: <what this endpoint does>
- **Use Case**: <mapped use case from DDD analysis>

#### Request
```json
{
  "<field>": "<type> — <description>"
}
```

#### Response (Success — 200/201)
```json
{
  "<field>": "<type> — <description>"
}
```

#### Error Responses
| Status | Code | Description |
|--------|------|-------------|
| 400 | VALIDATION_ERROR | <when> |
| 404 | NOT_FOUND | <when> |
| 409 | CONFLICT | <when> |
```

### Step 6: Define Event Publishing

Map domain events from the DDD analysis that originate from this bounded context:

**MANDATORY REQUIREMENTS for event-driven design:**
- Identify ALL state changes that other bounded contexts need to know about
- For each aggregate write operation, ask: "Does any other service need to react to this change?" If yes, publish an event.
- Events from MQ-based interactions MUST also be modeled as domain events
- Each event MUST have explicit trigger conditions with <FunctionName>::REQ-* references
- Each event MUST list ALL known subscribers with their bounded context identifier

**Minimum events per service:** Every service that performs write operations should publish at least:
1. An event for its primary aggregate state change
2. An event for any cross-context operation it receives
3. An event for any async/batch operation completion

```markdown
## Event Publishing

### <EventName>

- **Trigger**: <what causes this event — be specific about the use case and <FunctionName>::REQ-* references>
- **Topic/Channel**: `<context-name>.<aggregate>.<event-verb>`
- **Schema**:
  ```json
  {
    "eventId": "string (UUID)",
    "eventType": "<EventName>",
    "occurredAt": "ISO-8601 timestamp",
    "aggregateId": "<field name> (<type>)",
    "payload": {
      "<relevant state change fields with types>"
    }
  }
  ```
- **Subscribers**: <list ALL consuming services with their BC identifier>
```

### Step 7: Define Service Communication

Document how this service interacts with other bounded contexts:

**Communication patterns to specify:**
- **Synchronous (REST)**: For immediate consistency requirements
- **Asynchronous (Events)**: For eventual consistency and decoupling
- **Resilience patterns**: Circuit breaker, retry, timeout, fallback — **with specific numeric parameters**

**MANDATORY: Resilience Pattern Specificity**

Do NOT use generic descriptions like "Circuit Breaker / Retry / Timeout". Each synchronous dependency MUST specify concrete resilience parameters:

- **Circuit Breaker**: failure threshold, window duration, half-open retry interval
- **Retry**: maximum attempts, backoff strategy, initial delay
- **Timeout**: connection timeout and read timeout in seconds
- **Fallback**: what happens when the circuit is open

```markdown
## Service Communication

### Synchronous Dependencies (Outbound REST Calls)

| Target Service | Endpoint | Purpose | Resilience Pattern |
|---------------|----------|---------|-------------------|
| <service> | <path> | <why> | Circuit Breaker (<N> failures / <M>s window), Retry (<X> attempts, exponential backoff), Timeout (<Y>s) |

### Events Consumed (Inbound)

| Event | Source Context | Handler | Action |
|-------|---------------|---------|--------|
| <event name> | <context> | <handler class name> | <what happens> |

### Anti-Corruption Layer

| External Concept | Internal Concept | Translation Logic |
|-----------------|-----------------|-------------------|
| <external model term> | <internal domain term> | <specific translation> |
```

### Step 8: Define Domain Services

Map domain services from the DDD analysis that belong to this bounded context:

**CRITICAL RULES:**
- Only include domain services operating within this specific bounded context
- Cross-context coordination belongs in application services, not domain services
- **Identify ALL domain services** — not just one per bounded context

**MANDATORY: Full operation signatures with parameter types and return types**

```markdown
## Domain Services

### <Service Name>

- **Responsibility**: <single, clear statement>
- **Operations**:
  - `<operationName>(<param1>: <type>, <param2>: <type>)` → `<ReturnType>` — <description with <FunctionName>::REQ-* references>
- **Dependencies**:
  - Internal: <repositories, entities, value objects within this context>
  - External: <services from other bounded contexts>
- **Used By**: <which use cases invoke this service>
```

### Step 9: Define Application Services (Use Cases)

Map use cases from the DDD analysis to this service:

**CRITICAL RULES:**
- Only include use cases operating within this specific bounded context
- Single transaction boundary within this bounded context
- **Every flow step MUST have a REQ-* reference** — non-negotiable for traceability. REQ-* identifiers must be qualified with the function name: `<FunctionName>::REQ-XXX-NNN`

```markdown
## Application Services (Use Cases)

### <Use Case Name>

- **Actor**: <who triggers this>
- **API Endpoint**: <mapped endpoint from Step 5>
- **Input Command**:
  - <field>: <type> — <description>
- **Output Result**:
  - <field>: <type> — <description>
- **Flow**:
  1. <action description> (<FunctionName>::REQ-* identifier)
  2. <action description> (<FunctionName>::REQ-* identifier)
  ...
- **Transaction Boundary**: <what is atomic>
- **Error Handling**:
  | Condition | Error Code | Response |
  |-----------|-----------|----------|
  | <specific condition> | <CODE> | <HTTP status + description> (<FunctionName>::REQ-* identifier) |
- **Legacy Programs**: <program names from requirements>
```

### Step 10: Verification Checklist

**MANDATORY**: Before finalizing each service specification, complete this checklist. Do NOT move to the next service until all items pass.

#### 10a. Requirements Coverage

For each `<FunctionName>` mapped to this service:
1. Open `inputs/spec/<FunctionName>/requirements.md`
2. List ALL REQ-* identifiers that belong to use cases in this service
3. Verify that EACH REQ-* identifier appears in the specification, qualified with the function name (`<FunctionName>::REQ-XXX-NNN`)
4. If any REQ-* identifier is missing, add it to the appropriate section

#### 10b. Legacy Program Traceability

For each use case in the specification:
1. Verify it references the legacy program name(s) it derives from (from program names mentioned in requirements)
2. If missing, add a `Legacy Programs` field to the use case

#### 10c. Data Store Traceability

For each aggregate/entity in the Data Ownership section:
1. Verify the source data store DSN is referenced (from data-model.md)
2. Verify key fields are documented (inferred from glossary and requirements)
3. If missing, cross-reference data-model.md and .glossary.yaml to complete

#### 10d. Batch Operations Coverage

1. Review the function's requirements.md for batch-related workflows (sequential file processing, batch loops, end-of-file handling patterns)
2. Verify that ALL batch operations have corresponding use cases and API endpoints
3. If any batch operation is missing, add it as a use case with its own API endpoint

**Only after ALL checklist items pass, write the specification file and proceed to the next service.**

---

## Final Specification File Structure

Each microservice specification file (`<service-name>-specification.md`) must contain these sections in order:

```
# <Service Name> — Microservice Specification

> **Source**: <BC-N Context Name> | **Mapping**: <1:1 or 1:N> | **Source Function**: <FunctionName>
> **Validates**: <comma-separated list of ALL individual REQ-* identifiers covered — NO ranges, NO "through" shorthand>

---

## 1. Service Overview
## 2. Service Boundaries
## 3. Data Ownership
   ### Domain Model
   #### <Aggregate Name>
     - Entity key fields
     - Value Object summary table
   #### Value Object Detailed Specifications (one subsection per VO)
     - Attributes with types
     - Validation rules with <FunctionName>::REQ-*
     - Format specifications
   #### Invariants (with <FunctionName>::REQ-* references)
## 4. API Specification (all endpoints with full request/response/error schemas)
## 5. Event Publishing (all events with trigger conditions, schemas, and subscriber lists)
## 6. Service Communication (with specific resilience parameters per dependency)
## 7. Domain Services (multiple services with full typed operation signatures)
## 8. Application Services (Use Cases) (with <FunctionName>::REQ-* on every flow step)
```

## Key Principles

- **Qualify REQ-* identifiers with function name** — Because different business functions can have the same REQ-* identifiers, always prefix them with the source function name using the format `<FunctionName>::REQ-XXX-NNN` (e.g., `AccountManagement::REQ-F-001`). This ensures unambiguous traceability when multiple functions map to the same bounded context or service.
- **One service at a time** — generate each specification in isolation with full source reads before moving to the next
- **One file per microservice** — never combine specifications
- **Never merge bounded contexts** — if the DDD model defines separate contexts, they MUST remain separate services
- **Bounded context scope** — each service only owns data and logic from its bounded context
- **No shared databases** — services communicate through APIs and events
- **Complete API coverage** — every use case (online AND batch) has a corresponding API endpoint
- **Explicit boundaries** — always state what the service does NOT own
- **Resilience by default** — all cross-service communication includes resilience patterns WITH SPECIFIC NUMERIC PARAMETERS
- **Event-driven integration** — prefer asynchronous events for cross-context communication
- **Anti-corruption layers** — protect internal models from external context changes
- **Mandatory source reading** — ALWAYS read requirements.md for each function before generating its service spec
- **Full legacy traceability** — every use case must reference its legacy program name (from requirements); every validation must reference its <FunctionName>::REQ-* ID
- **Batch operations are first-class** — batch file processing, sequential record operations, and file-based data flows are full use cases with API endpoints
- **Verify before finalizing** — complete the Step 10 verification checklist for each service before writing the file
- **Value objects are fully specified** — each VO gets its own section with attributes, validation rules, format, and immutability guarantee
- **Multiple domain services per context** — identify ALL distinct service responsibilities
- **REQ-* on every flow step** — every step must reference the specific requirement it implements, qualified with the function name (<FunctionName>::REQ-XXX-NNN)
- **Events are comprehensive** — publish events for primary state changes, cross-context operations, AND async/batch completions
- **Resilience parameters are concrete** — specify failure threshold, window, retry count, backoff strategy, and timeout for each dependency
- **Data store info from data-model.md** — use data-model.md for store names, types, and access patterns since no separate data-stores.yaml field dictionaries are available
- **Field details from multiple sources** — infer field-level information from field descriptions in .glossary.yaml and field references in requirements.md
- **No range shorthand for REQ-* IDs** — NEVER write "REQ-F-001 through REQ-F-010" or "REQ-F-001 to REQ-F-010". Always enumerate every individual REQ-* identifier explicitly (e.g., `REQ-F-001, REQ-F-002, REQ-F-003, REQ-F-004, REQ-F-005, REQ-F-006, REQ-F-007, REQ-F-008, REQ-F-009, REQ-F-010`). Phase 4 traceability verification performs exact-match lookups on individual IDs — ranges are invisible to that process and cause requirements to appear untraced.
