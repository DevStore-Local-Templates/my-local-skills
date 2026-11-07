---
name: phase-5-html-specification-generation
description: Generate interactive HTML versions of microservice specification documents with styled code blocks, method-colored API cards, domain model visualizations, and responsive design. Use when producing stakeholder-ready HTML specs from markdown specifications.
---

# Phase 5: HTML Specification Generation

## Purpose

This steering file guides the generation of interactive HTML versions of the microservice specification documents produced in Phase 3. Each markdown specification (`*-specification.md`) is transformed into a visually rich HTML document with styled code blocks, method-colored API endpoint cards, domain model visualizations, integration point boxes, and responsive design suitable for stakeholder reviews and presentations.

## Reference Format

The generated HTML files MUST match the style and depth of the reference file `customer-management-service-spec.html` in the workspace root. Key characteristics of the expected output:

### Visual Style Requirements
1. **Domain Model as code blocks** — Entities and value objects rendered as structured code blocks inside styled `.domain-model` boxes (dark background, monospace font), NOT as plain tables
2. **API endpoints with method-colored cards** — Each endpoint in its own styled card with method-specific colors:
   - GET endpoints: blue border/header (`.method-get` class, color `#3498db`)
   - POST endpoints: green border/header (`.method-post` class, color `#28a745`)
   - PUT endpoints: yellow/orange border/header (`.method-put` class, color `#f39c12`)
   - DELETE endpoints: red border/header (`.method-delete` class, color `#e74c3c`)
3. **Integration points in styled boxes** — Each integration point in an orange-bordered `.integration-point` box with event schemas as code blocks
4. **Ubiquitous Language as definition lists** — A dedicated glossary section using `<dl>/<dt>/<dd>` inside a styled `.ubiquitous-language` box
5. **Info and warning boxes** — Key information in blue `.info` boxes, warnings in yellow `.warning` boxes
6. **Domain service operations as code** — Operation signatures shown as typed inline code (`operationName(param: Type)` → `ReturnType`)
7. **Use case flows as ordered lists** — Each step with requirement references (REQ-*) inline
8. **No Mermaid diagrams required** — Use styled code blocks and structured HTML instead of Mermaid (Mermaid is optional, not mandatory)

### Content Depth Requirements
Each HTML file MUST include these sections with FULL detail, mirroring the 8 sections of the markdown specification:

1. **Service Overview** — Purpose description + Ubiquitous Language glossary (definition list with 8+ terms) + service metadata (bounded context, domain type, team ownership, program count)
2. **Service Boundaries** — Owned entities/data stores (with full DSNs) + what the service does NOT own + dependencies on other services
3. **Data Ownership** — Aggregate root as code block + ALL entities as code blocks with field mapping tables + ALL value objects as code blocks with full specifications (attributes, validation rules with REQ-*, format, immutability) + ALL invariants with REQ-* references
4. **API Specification** — EVERY endpoint in its own method-colored card with:
   - Purpose description
   - Full request JSON schema (with field types)
   - Full response JSON schema (with field types)
   - Error responses table (all status codes)
   - Use case mapping
5. **Event Publishing** — ALL events with trigger conditions (including REQ-* references) + topic/channel names + full JSON schemas + subscriber lists with BC identifiers
6. **Service Communication** — Synchronous dependencies with SPECIFIC numeric resilience parameters (circuit breaker thresholds, retry counts, timeout values) + events consumed table + anti-corruption layer table
7. **Domain Services** — ALL domain services with full typed operation signatures (`operationName(param: type)` → `ReturnType`) + dependencies (internal and external) + REQ-* references + consuming use cases
8. **Application Services (Use Cases)** — ALL use cases with actor, API endpoint, input/output, flow steps (EACH with REQ-* reference), transaction boundary, error handling table, legacy program names

### CSS Classes Required
The HTML MUST include these CSS classes:
- `.container` — Main content wrapper (white background, rounded corners, shadow)
- `.toc` — Table of contents (gray background, rounded)
- `.domain-model` — Entity/value object code blocks (light gray background, bordered)
- `.api-endpoint` — Base API endpoint card
- `.method-get` — Blue-themed GET endpoint card
- `.method-post` — Green-themed POST endpoint card
- `.method-put` — Yellow-themed PUT endpoint card
- `.method-delete` — Red-themed DELETE endpoint card (add this)
- `.integration-point` — Orange-bordered integration box
- `.ubiquitous-language` — Blue-bordered glossary box
- `.info` — Blue info box
- `.warning` — Yellow warning box
- `pre code` — Dark background code blocks for schemas and domain models

## Prerequisites

Phase 5 requires that the following phases have been completed:

1. **Phase 3** produced individual microservice specification files (`*-specification.md`) in `outputs/microservices/`
2. **Phase 4** produced the `traceability-dashboard.html` confirming requirements coverage

The HTML generation also requires:
- Completed specification files with sections 1–8 (Service Overview through Application Services)
- Understanding of each service's bounded context, domain type, and integration patterns from the DDD analysis

## Input Sources

### 1. Primary Input (Markdown Specifications)

The 7 microservice specification markdown files in `outputs/microservices/`:

| File | Service | Domain Type |
|------|---------|-------------|
| `account-management-specification.md` | Account Management Service | Core Domain |
| `transaction-processing-specification.md` | Transaction Processing Service | Core Domain |
| `card-authorization-specification.md` | Card Authorization Service | Core Domain |
| `reporting-statements-specification.md` | Reporting and Statements Service | Supporting |
| `user-security-specification.md` | User Security Service | Supporting |
| `online-integration-specification.md` | Online Integration Service | Supporting |
| `platform-operations-specification.md` | Platform Operations Service | Generic |

### 2. Reference Input (DDD Analysis)

- `ddd-analysis.md` — For context map relationships, bounded context definitions, and integration patterns
- `outputs/microservices/microservice-inventory.md` — For service-to-service dependency information and integration dependency matrix

## Output Rules

**MANDATORY**: Each microservice specification MUST have its own dedicated HTML file.
- Do NOT combine multiple specifications into a single HTML document
- If a bounded context is decomposed into N services, produce N separate HTML files
- File naming convention: `<ServiceName>-Microservice-Spec.html` (PascalCase service name)
- Output location: `outputs/microservices/`
- Additionally produce a `MicroservicesArchitecture-Overview.html` system-level view
- Optionally produce a `test-mermaid.html` test file for diagram validation (only if Mermaid is used)

**REFERENCE FILE**: Use `customer-management-service-spec.html` in the workspace root as the gold standard for HTML structure, CSS styling, content depth, and visual presentation. Every generated HTML file must match this level of detail and styling.

## Execution Model: One HTML File at a Time

**CRITICAL**: Do NOT generate all HTML specification files in a single pass. Process **one service at a time** to ensure maximum quality and completeness for each HTML document.

**Execution sequence:**
1. Read the markdown specification file COMPLETELY before starting HTML generation
2. For EACH service, execute Steps 1–8 completely before moving to the next service
3. Verify Mermaid syntax correctness and content completeness before writing the file

**Why:** Generating all HTML files at once causes context pressure that leads to shallow conversions, missing content, dropped REQ-* anchors, and incomplete API endpoint coverage. Processing one service at a time ensures each HTML document faithfully represents the full specification.

---

## Mandatory Source Reading Per Service

**BEFORE generating each HTML file**, you MUST read the corresponding markdown specification file in its entirety. Extract and keep available while generating the HTML:

### What to Extract from the Markdown

1. **Service metadata** — Service name, bounded context, domain type, purpose, team ownership, program count
2. **Service boundaries** — Owned entities/data stores (with full DSNs), what it does NOT own, dependencies on other services
3. **Data ownership** — ALL aggregates, entities with field mappings (Domain Attribute, Source Field, PIC Type, Offset, Length), ALL value objects with their dedicated specification sections (attributes, validation rules with REQ-*, format, immutability), ALL invariants with REQ-* references
4. **API specification** — EVERY endpoint with method, path, description, use case, request schema, response schema, error responses
5. **Event publishing** — ALL events with trigger conditions (including REQ-* references), topic/channel names, schemas, subscriber lists
6. **Service communication** — Synchronous dependencies with SPECIFIC resilience parameters (circuit breaker thresholds, retry counts, timeout values), events consumed, anti-corruption layers
7. **Domain services** — ALL services with full typed operation signatures (`operationName(param: type)` → `ReturnType`), dependencies, REQ-* references, and consuming use cases
8. **Application services (use cases)** — ALL use cases with actor, API endpoint, input/output, flow steps (EACH with REQ-* reference), transaction boundary, error handling, legacy program names

**FAILURE MODE**: If you skip thorough reading, the HTML will lack critical content — missing value object specifications, incomplete API endpoints, dropped REQ-* anchors, or shallow use case flows. This is the #1 quality differentiator.

---

## HTML Generation Process

Follow these steps sequentially for each microservice specification.

### Step 1: Set Up HTML Structure

Create the HTML5 document matching the reference file's structure:
- Responsive viewport meta tag
- Full CSS stylesheet matching `customer-management-service-spec.html` styling
- Container structure with white background, rounded corners, shadow

**CRITICAL Requirements:**
- **MUST match the reference file's CSS** — Use the exact same class names and styling patterns
- **Mermaid is OPTIONAL** — Only include Mermaid if diagrams add value; the reference file does NOT use Mermaid
- **Method-colored API cards are MANDATORY** — Each API endpoint gets its own styled card
- **Domain model code blocks are MANDATORY** — Entities shown as structured code, not tables

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>[Service Name] Microservice Specification</title>
    <style>
        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            line-height: 1.6;
            color: #333;
            max-width: 1200px;
            margin: 0 auto;
            padding: 20px;
            background-color: #f8f9fa;
        }
        .container {
            background-color: white;
            padding: 30px;
            border-radius: 8px;
            box-shadow: 0 2px 10px rgba(0,0,0,0.1);
        }
        h1 {
            color: #2c3e50;
            border-bottom: 3px solid [SERVICE_COLOR];
            padding-bottom: 10px;
            margin-bottom: 30px;
        }
        h2 {
            color: #34495e;
            border-left: 4px solid [SERVICE_COLOR];
            padding-left: 15px;
            margin-top: 40px;
            margin-bottom: 20px;
        }
        h3 { color: #2c3e50; margin-top: 30px; margin-bottom: 15px; }
        h4 { color: #34495e; margin-top: 25px; margin-bottom: 10px; }
        .toc {
            background-color: #ecf0f1;
            padding: 20px;
            border-radius: 5px;
            margin-bottom: 30px;
        }
        .toc ul { list-style-type: none; padding-left: 0; }
        .toc li { margin: 8px 0; }
        .toc a {
            color: [SERVICE_COLOR];
            text-decoration: none;
            padding: 5px 10px;
            display: block;
            border-radius: 3px;
            transition: background-color 0.3s;
        }
        .toc a:hover { background-color: #bdc3c7; }
        pre {
            background-color: #2c3e50;
            color: #ecf0f1;
            padding: 15px;
            border-radius: 5px;
            overflow-x: auto;
            font-size: 14px;
        }
        code {
            background-color: #ecf0f1;
            padding: 2px 5px;
            border-radius: 3px;
            font-family: 'Courier New', monospace;
            color: #e74c3c;
        }
        pre code { background-color: transparent; color: #ecf0f1; padding: 0; }
        .api-endpoint {
            background-color: #e8f5e8;
            border: 1px solid #27ae60;
            border-radius: 5px;
            padding: 15px;
            margin: 15px 0;
        }
        .api-endpoint h4 { color: #27ae60; margin-top: 0; }
        .method-get { background-color: #e8f4fd; border-color: #3498db; }
        .method-get h4 { color: #3498db; }
        .method-put { background-color: #fff3cd; border-color: #f39c12; }
        .method-put h4 { color: #f39c12; }
        .method-post { background-color: #d4edda; border-color: #28a745; }
        .method-post h4 { color: #28a745; }
        .method-delete { background-color: #f8d7da; border-color: #dc3545; }
        .method-delete h4 { color: #dc3545; }
        .domain-model {
            background-color: #f8f9fa;
            border: 1px solid #dee2e6;
            border-radius: 5px;
            padding: 15px;
            margin: 15px 0;
        }
        .integration-point {
            background-color: #fdf2e9;
            border: 1px solid #e67e22;
            border-radius: 5px;
            padding: 15px;
            margin: 15px 0;
        }
        .warning {
            background-color: #fcf8e3;
            border: 1px solid #f39c12;
            border-radius: 5px;
            padding: 15px;
            margin: 15px 0;
        }
        .info {
            background-color: #d9edf7;
            border: 1px solid #3498db;
            border-radius: 5px;
            padding: 15px;
            margin: 15px 0;
        }
        table { width: 100%; border-collapse: collapse; margin: 15px 0; }
        th, td { border: 1px solid #ddd; padding: 12px; text-align: left; }
        th { background-color: [SERVICE_COLOR]; color: white; }
        tr:nth-child(even) { background-color: #f2f2f2; }
        .ubiquitous-language {
            background-color: #f0f8ff;
            border: 1px solid #87ceeb;
            border-radius: 5px;
            padding: 15px;
            margin: 15px 0;
        }
        .ubiquitous-language dt { font-weight: bold; color: #2c3e50; margin-top: 10px; }
        .ubiquitous-language dd { margin-left: 20px; margin-bottom: 10px; }
        footer {
            margin-top: 50px;
            padding-top: 20px;
            border-top: 1px solid #ddd;
            text-align: center;
            color: #666;
        }
    </style>
</head>
<body>
    <div class="container">
        <!-- Content per Steps 2–8 -->
        <footer>
            <p>[Service Name] Microservice Specification - Generated as part of DDD Microservices Analysis</p>
        </footer>
    </div>
</body>
</html>
```

**Service Color Coding** (applied to `h1` border-bottom, `h2` border-left, `th` background, and `.toc a` color):
- **Core Domain**: Green (`#27ae60`) — matches the reference file
- **Supporting Subdomain**: Blue (`#3498db`)
- **Generic Subdomain**: Gray (`#95a5a6`)

### Step 2: Add CSS Styling

The CSS is already included in the Step 1 template above. The styling matches the reference file `customer-management-service-spec.html` with:
- Segoe UI font stack
- White container with shadow
- Service-color-coded headings (h1 border-bottom, h2 border-left)
- Method-colored API endpoint cards (GET=blue, POST=green, PUT=yellow, DELETE=red)
- Domain model boxes with light gray background
- Integration point boxes with orange border
- Info boxes (blue) and warning boxes (yellow)
- Ubiquitous language glossary with light blue background
- Dark code blocks for JSON schemas and entity definitions
- Zebra-striped tables with colored headers
- Footer with generation attribution

### Step 3: Create Metadata Section

Extract from the specification's Section 1 (Service Overview):
- Service name
- Bounded context (with BC-N identifier)
- Domain type (with colored badge)
- Purpose (2–3 sentence description)
- Business capabilities list
- Team ownership
- Program count

### Step 4: Create Table of Contents

Generate navigation links matching the exact specification section structure:
1. Service Overview
2. Service Boundaries
3. Data Ownership (including Domain Model, Aggregates, Value Object Specifications, Invariants)
4. API Specification (all endpoints)
5. Event Publishing
6. Service Communication
7. Domain Services
8. Application Services (Use Cases)

Each link uses an anchor (`href="#section-id"`) pointing to the corresponding section.

### Step 5: Convert Content Sections

Transform all 8 markdown sections into semantic HTML, preserving the exact structure from Phase 3:

**Conversion rules:**
- `## N. Heading` → `<h2 id="section-id">N. Heading</h2>`
- `### Heading` → `<h3>Heading</h3>`
- `#### Heading` → `<h4>Heading</h4>`
- Markdown lists → `<ul><li>...</li></ul>`
- Code blocks → `<pre><code>...</code></pre>`
- JSON blocks → `<pre><code class="json">...</code></pre>`
- Markdown tables → `<table>` with `<thead>` and `<tbody>`
- Bold text → `<strong>...</strong>`
- Inline code (backticks) → `<code>...</code>`
- Block quotes → `<blockquote>...</blockquote>`

**Section-Specific Conversion Requirements:**

#### Section 3: Data Ownership (Critical)

Must preserve the full nested structure:
- **Aggregate definitions** with aggregate root, identity, source data store DSN, entities table
- **Entity field mapping tables** — ALL columns: Domain Attribute, Source Field, PIC Type, Offset, Length, Required, Description
- **Value Object summary table** — ALL value objects listed
- **Value Object Detailed Specifications** — EACH value object gets its own `<h4>` subsection with:
  - Source fields
  - Attributes table (Attribute, PIC Type, Length, Description)
  - Validation rules list (each referencing REQ-*)
  - Format specification
  - Immutability guarantee statement
- **Invariants** — ALL invariants with REQ-* references

**Do NOT collapse value objects into a summary table only. Each MUST have its full specification.**

#### Section 4: API Specification (Critical)

**MANDATORY**: Every API endpoint in the markdown MUST have a detailed specification in the HTML.

For EACH endpoint, include:
1. Property table (method, path, description, use case mapping)
2. Request schema — JSON example in `<pre><code>` with field types and descriptions
3. Response schema — Success response (200/201) in `<pre><code>`
4. Error responses table — ALL status codes with error code and triggering condition

**Verification**: Count API endpoints in markdown = count detailed API endpoint sections in HTML. Do NOT relegate any endpoints to summary tables without full specifications.

#### Section 5: Event Publishing (Critical)

For EACH event, include:
- Trigger condition with REQ-* references
- Topic/Channel name
- Full event schema in `<pre><code class="json">`
- Subscriber list with bounded context identifiers

**Minimum events per service**: Every service with write operations should have events for primary state changes, cross-context operations, and async/batch completions.

#### Section 6: Service Communication (Critical)

**MANDATORY: Specific numeric resilience parameters.** Do NOT use generic descriptions.

For each synchronous dependency, render a table with:
- Target service
- Endpoint path
- Purpose
- Resilience pattern with SPECIFIC values: Circuit Breaker (N failures / Ms window), Retry (X attempts, backoff strategy), Timeout (Ys)

Also include:
- Events consumed table (event name, source context, handler, action)
- Anti-corruption layer table (external concept, internal concept, translation logic)

#### Section 7: Domain Services (Critical)

For EACH domain service:
- Responsibility (single clear statement)
- **Full typed operation signatures**: `<code>operationName(param1: Type, param2: Type)</code>` → `<code>ReturnType</code>` — description with REQ-* references
- Dependencies (internal and external)
- Consuming use cases list

#### Section 8: Application Services / Use Cases (Critical)

For EACH use case:
- Actor, API endpoint mapping, input command, output result
- **Flow steps** — EACH step MUST show its REQ-* identifier reference
- Transaction boundary
- Error handling table (condition, error code, HTTP status + description with REQ-* reference)
- Legacy program names

**Every flow step MUST have a REQ-* reference — non-negotiable for traceability.**

### Step 6: Create Mermaid Diagrams

Generate interactive diagrams for key sections:

#### 6.1 Service Context Diagram (Section 2: Service Boundaries)
- Show the service at center with upstream/downstream dependencies
- Color by domain type: Core=red, Supporting=blue, Generic=gray
- Label edges with integration mechanism (REST, Events, File-based)

#### 6.2 Domain Model Class Diagram (Section 3: Data Ownership)
- Show aggregates, entities, and value objects
- Use `note for ClassName "Aggregate Root"` (NOT `<<stereotype>>`)
- Show relationships with multiplicity: `"1"`, `"*"`

#### 6.3 API Endpoints Flow Diagram (Section 4: API Specification)
- Show client → endpoint → response flow
- Color by HTTP method: GET=blue, POST=green, PUT=orange, DELETE=red
- Use quotes for labels with special chars: `["GET /api/v1/resource/{id}"]`

#### 6.4 Event Flow Sequence Diagram (Section 5: Event Publishing)
- Show event producers, event bus, and ALL subscribers
- Use `participant Alias as "Display Name"`
- Show event names on arrows

#### 6.5 Use Case Sequence Diagrams (Section 8: Application Services)
- Show primary use case flows (select 2–3 key use cases per service)
- Include Client → UseCase → DomainService → Repository → EventBus
- Label with operation names and event names

#### 6.6 Batch Processing Flowchart (if applicable)
- Show batch job flow: Start → Query → Process → Complete
- Include error/retry paths
- Batch operations are first-class — store initialization, data extraction, and file operations get their own flow diagrams

**CRITICAL Mermaid Syntax Rules (v11):**
- **DO** use `<br/>` for line breaks in node labels
- **DO** wrap special characters in quotes: `["Label with {special}"]`
- **DO** use `{id}` for path params (not `:id`)
- **DO** use `note for ClassName "stereotype"` for class stereotypes
- **DO** quote edge labels with special chars: `|"GET /api/{id}"|`
- **DO** apply styles AFTER defining all nodes and edges
- **DO NOT** use `<<stereotype>>` syntax in class diagrams
- **DO NOT** use colons `:` in unquoted node labels
- **DO NOT** use question marks `?` in unquoted node labels
- **DO NOT** use complex URLs as unquoted node labels

### Step 7: Create Tables

Convert all markdown tables to styled HTML tables:
- Use `<thead>` and `<tbody>` for proper structure
- Apply zebra striping via CSS (`:nth-child(even)`)
- Bold important columns (identifiers, names)
- Ensure responsive overflow handling (`overflow-x: auto` wrapper)
- Preserve ALL data — no rows dropped during conversion

**Common tables that MUST be faithfully converted:**
- Entity field mapping tables (Domain Attribute, Source Field, PIC Type, Offset, Length)
- API error response tables (Status, Code, Description)
- Synchronous dependencies table (with specific resilience parameters)
- Events consumed table
- Anti-corruption layer table
- Use case error handling tables
- Value object attributes tables

### Step 8: Validate and Finalize

**MANDATORY**: Before writing each HTML file, complete this verification checklist. Do NOT proceed to the next service until all items pass.

#### 8a. Content Completeness

1. Verify all 8 specification sections (1–8) are present in the HTML
2. Verify ALL aggregates and their entity field mapping tables are present
3. Verify ALL value objects have their dedicated specification sections (not just summary tables)
4. Verify ALL invariants are listed with REQ-* references

#### 8b. API Endpoint Coverage

1. Count API endpoints in the markdown specification
2. Count detailed API endpoint sections in the HTML (with property tables and request/response examples)
3. Verify counts match — no endpoints relegated to summary tables only
4. Verify each endpoint has request schema, response schema, and error responses

#### 8c. REQ-* Anchor Preservation

1. Verify ALL REQ-* identifiers from the markdown appear verbatim in the HTML
2. Verify ALL data store DSN names (e.g., `%%CICV.MU.MUVI01.LOVIM`) are preserved verbatim
3. Verify ALL legacy program names (e.g., `MUHUO02I`, `MUBSZSFT`) are preserved verbatim
4. Verify every use case flow step references at least one REQ-*

#### 8d. Event and Communication Completeness

1. Verify ALL domain events have trigger conditions, schemas, and subscriber lists
2. Verify ALL synchronous dependencies have specific numeric resilience parameters
3. Verify ALL consumed events are listed with source context and action

#### 8e. Domain Service Completeness

1. Verify ALL domain services have full typed operation signatures
2. Verify ALL consuming use cases are listed per domain service

#### 8f. Mermaid Syntax Validation

1. Verify no `<<stereotype>>` syntax in class diagrams
2. Verify no unquoted colons or special characters in node labels
3. Verify styles are applied AFTER all nodes and edges are defined
4. Verify all referenced nodes exist in their diagrams

**Only after ALL checklist items pass, write the HTML file and proceed to the next service.**

---

## System Architecture Overview Generation

After generating all individual HTML specification files, produce a **system-level architecture overview** HTML file (`MicroservicesArchitecture-Overview.html`) that shows all identified microservices and their interactions in a single interactive document.

### Purpose

This file provides stakeholders with a bird's-eye view of the entire microservice ecosystem — how services relate to each other, data flows between them, and integration patterns used. It complements the individual specification files by showing the system as a whole.

### Input Sources

- `ddd-analysis.md` Section 8 (Context Map) — upstream/downstream relationships, integration patterns
- `outputs/microservices/microservice-inventory.md` — service inventory table and integration dependency matrix
- `ddd-analysis.md` Section 6.5 (Cross-Context Data Flow Dependencies) — data flow details

### Content Structure

```html
<h1>Insurance Microservices — System Architecture Overview</h1>

<h2 id="inventory">1. Microservice Inventory</h2>
  Summary table of all services (name, bounded context, domain type, programs)

<h2 id="architecture-diagram">2. System Architecture Diagram</h2>
  [Full system Mermaid diagram showing all services and their interactions]

<h2 id="data-flows">3. Data Flow Diagram</h2>
  [Mermaid diagram showing primary data flows between services]

<h2 id="event-flows">4. Event-Driven Integration</h2>
  [Mermaid sequence diagram showing key event flows across services]

<h2 id="dependencies">5. Integration Dependencies</h2>
  Dependency matrix table (service → upstream dependencies / downstream dependents)

<h2 id="patterns">6. Integration Patterns</h2>
  Table of all inter-service relationships with pattern classification
  (Customer-Supplier, Open Host Service, Published Language, Anti-Corruption Layer)

<h2 id="domain-classification">7. Domain Classification</h2>
  Visual grouping of services by domain type (Core, Supporting, Generic)
```

### Diagram Requirements

#### 2. System Architecture Diagram

A `graph TB` Mermaid diagram showing:
- ALL microservices as nodes, color-coded by domain type
- Subgraphs grouping services by classification (Core Domain, Supporting Subdomain, Generic Subdomain)
- Edges showing integration relationships with labels indicating the mechanism (REST, Events, File-based)
- The central integration hotspot clearly visible

#### 3. Data Flow Diagram

A `graph LR` Mermaid diagram showing:
- Primary data stores and their owning services
- Read/write relationships between services and shared data

#### 4. Event-Driven Integration

A `sequenceDiagram` showing key cross-service event flows:
- Event producers, event bus, and ALL subscribers
- Event names on arrows

### Additional Content

#### Integration Dependencies Table

| Service | Upstream Dependencies (consumes from) | Downstream Dependents (provides to) |
|---------|---------------------------------------|--------------------------------------|

#### Integration Patterns Table

| Upstream | Downstream | Pattern | Mechanism |
|----------|------------|---------|-----------|

### Styling

- Use the same CSS template as individual specification files
- Color legend showing domain type classification
- Interactive Mermaid diagrams (clickable nodes linking to individual spec files if possible)
- Responsive layout for stakeholder presentations

### Validation

Before writing this file:
- ✅ All services are represented in diagrams
- ✅ All inter-service relationships from the context map are shown
- ✅ Color coding matches domain type classification
- ✅ Integration patterns are correctly labeled
- ✅ Mermaid syntax follows v11 rules (quoted labels, styles after definitions)
- ✅ File is self-contained and renders without a server

---

## Test File Generation

After generating all HTML specification files and the architecture overview, produce a single `test-mermaid.html` file that includes one example of each diagram type used across all specifications:

1. Service context graph (graph TB)
2. Class diagram with notes (classDiagram)
3. Sequence diagram (sequenceDiagram)
4. Flowchart with decisions (graph LR/TB)
5. API endpoint flow with HTTP method colors
6. System architecture graph with subgraphs (from the overview file)

This file serves as a regression test — if all diagrams in the test file render correctly, the syntax patterns used in the specifications are valid.

---

## Final HTML File Structure

Each HTML specification file must contain these 8 sections in order, **exactly mirroring the markdown specification structure** from Phase 3:

```
<h1>[Service Name] Microservice Specification</h1>

<div class="toc">Table of Contents (8 sections)</div>

<section id="service-overview">
  <h2>1. Service Overview</h2>
  <p>Purpose and responsibility description (2-3 paragraphs)</p>
  <div class="info">
    <ul>
      <li><strong>Bounded Context:</strong> BC-N — Context Name</li>
      <li><strong>Domain Type:</strong> Core Domain / Supporting / Generic</li>
      <li><strong>Team Ownership:</strong> ...</li>
      <li><strong>Program Count:</strong> N</li>
    </ul>
  </div>
  <div class="ubiquitous-language">
    <h3>Ubiquitous Language</h3>
    <dl>
      <dt>Term 1</dt><dd>Definition</dd>
      <dt>Term 2</dt><dd>Definition</dd>
      ... (8+ terms per service)
    </dl>
  </div>
</section>

<section id="service-boundaries">
  <h2>2. Service Boundaries</h2>
  <h3>Owns</h3>
  <ul>
    <li>Aggregate/entity with data store DSN</li>
  </ul>
  <h3>Does NOT Own</h3>
  <ul>
    <li>Entity → owned by Other Service/Context</li>
  </ul>
  <h3>Dependencies</h3>
  <ul>
    <li>Other Microservice: what is needed and why</li>
  </ul>
</section>

<section id="data-ownership">
  <h2>3. Data Ownership</h2>
  <h3>Domain Model</h3>
  <h4>Aggregate Name</h4>
  <div class="domain-model">
    <h4>Entity Name (Aggregate Root)</h4>
    <pre><code>EntityName {
      - field1: Type
      - field2: Type
      ...
    }</code></pre>
  </div>
  ... (one .domain-model box per entity)
  <h3>Entity Field Mappings</h3>
  <table>Entity field mapping tables (Domain Attribute, Source Field, PIC Type, Offset, Length)</table>
  <h3>Value Objects</h3>
  <div class="domain-model">
    <h4>ValueObjectName</h4>
    <pre><code>ValueObjectName {
      - field1: Type (validation rule)
    }</code></pre>
  </div>
  ... (one .domain-model box per value object with FULL specification)
  <h3>Invariants</h3>
  <ul>
    <li>Invariant description (REQ-*)</li>
  </ul>
</section>

<section id="api-specification">
  <h2>4. API Specification</h2>
  <div class="api-endpoint method-get">
    <h4>GET /api/v1/resource/{id}</h4>
    <p><strong>Description:</strong> ...</p>
    <p><strong>Use Case:</strong> ...</p>
    <p><strong>Request:</strong></p>
    <pre><code>{ JSON schema }</code></pre>
    <p><strong>Response (Success — 200):</strong></p>
    <pre><code>{ JSON schema }</code></pre>
    <h4>Error Responses</h4>
    <table>Status | Code | Description</table>
  </div>
  ... (one card per endpoint, colored by HTTP method)
</section>

<section id="event-publishing">
  <h2>5. Event Publishing</h2>
  <div class="integration-point">
    <h4>EventName</h4>
    <p><strong>Trigger:</strong> ... (REQ-* references)</p>
    <p><strong>Topic/Channel:</strong> <code>context.aggregate.event-verb</code></p>
    <pre><code>{ event JSON schema }</code></pre>
    <p><strong>Subscribers:</strong> list with BC identifiers</p>
  </div>
  ... (one box per event)
</section>

<section id="service-communication">
  <h2>6. Service Communication</h2>
  <h3>Synchronous Dependencies</h3>
  <table>Target Service | Endpoint | Purpose | Resilience Pattern (with specific numeric parameters)</table>
  <h3>Events Consumed</h3>
  <table>Event | Source Context | Handler | Action</table>
  <h3>Anti-Corruption Layer</h3>
  <table>External Concept | Internal Concept | Translation Logic</table>
</section>

<section id="domain-services">
  <h2>7. Domain Services</h2>
  <h3>ServiceName</h3>
  <p><strong>Responsibility:</strong> single clear statement</p>
  <h4>Operations</h4>
  <ul>
    <li><code>operationName(param1: Type, param2: Type)</code> → <code>ReturnType</code> — description with REQ-* references</li>
  </ul>
  <h4>Dependencies</h4>
  <ul>
    <li><strong>Internal:</strong> repositories, entities, value objects</li>
    <li><strong>External:</strong> services from other bounded contexts</li>
  </ul>
  <h4>Consuming Use Cases</h4>
  <ul>
    <li>Use case name</li>
  </ul>
  ... (one subsection per domain service)
</section>

<section id="application-services">
  <h2>8. Application Services (Use Cases)</h2>
  <h3>Use Case Name</h3>
  <div class="info">
    <ul>
      <li><strong>Actor:</strong> ...</li>
      <li><strong>API Endpoint:</strong> <code>METHOD /path</code></li>
      <li><strong>Legacy Programs:</strong> PROGNAME1, PROGNAME2</li>
    </ul>
  </div>
  <h4>Input Command</h4>
  <pre><code>{ input fields with types }</code></pre>
  <h4>Output Result</h4>
  <pre><code>{ output fields with types }</code></pre>
  <h4>Flow</h4>
  <ol>
    <li>Action description (<code>FunctionName::REQ-X-NNN</code>)</li>
    <li>Action description (<code>FunctionName::REQ-X-NNN</code>)</li>
  </ol>
  <h4>Transaction Boundary</h4>
  <p>What is atomic</p>
  <h4>Error Handling</h4>
  <table>Condition | Error Code | Response (HTTP status + REQ-* reference)</table>
  ... (one subsection per use case)
</section>

<footer>
  <p>[Service Name] Microservice Specification - Generated as part of DDD Microservices Analysis</p>
</footer>
```

---

## Output Files

### Required Output

| File | Location | Purpose |
|------|----------|---------|
| `AccountManagement-Microservice-Spec.html` | `outputs/microservices/` | BC-1 HTML spec |
| `TransactionProcessing-Microservice-Spec.html` | `outputs/microservices/` | BC-2 HTML spec |
| `CardAuthorization-Microservice-Spec.html` | `outputs/microservices/` | BC-3 HTML spec |
| `ReportingStatements-Microservice-Spec.html` | `outputs/microservices/` | BC-4 HTML spec |
| `UserSecurity-Microservice-Spec.html` | `outputs/microservices/` | BC-5 HTML spec |
| `OnlineIntegration-Microservice-Spec.html` | `outputs/microservices/` | BC-6 HTML spec |
| `PlatformOperations-Microservice-Spec.html` | `outputs/microservices/` | BC-7 HTML spec |
| `MicroservicesArchitecture-Overview.html` | `outputs/microservices/` | System-level architecture diagram with all services and interactions |

### Constraints

- **ONLY produce HTML files** — no JavaScript bundles, CSS files, or build artifacts
- **Self-contained** — each HTML file must work standalone (no external CSS/JS dependencies except the Mermaid CDN)
- **No server required** — files must render correctly when opened directly in a browser (`file://` protocol)

---

## Quality Checklist

Before completing each HTML file, verify:

### Completeness
- ✅ All 8 specification sections from markdown are included
- ✅ ALL API endpoints have detailed specifications (no summary-table-only endpoints)
- ✅ ALL value objects have dedicated specification sections (not just summary tables)
- ✅ ALL invariants are listed with REQ-* references
- ✅ ALL diagrams created for applicable sections
- ✅ ALL markdown tables converted to HTML tables with no rows dropped
- ✅ Table of contents is complete with working links
- ✅ Metadata section present with correct domain type badge

### API Specification Coverage (CRITICAL)
- ✅ Count of API endpoints in markdown = count of detailed API specs in HTML
- ✅ Each endpoint has method, path, description, use case mapping
- ✅ Each endpoint has request schema (JSON example)
- ✅ Each endpoint has response schema (success response)
- ✅ Each endpoint has error responses table (all status codes)
- ✅ No endpoints relegated to summary tables without full details

### REQ-* Preservation (CRITICAL)
- ✅ ALL REQ-* identifiers preserved verbatim in HTML content
- ✅ ALL data store DSNs preserved verbatim (e.g., `%%CICV.MU.MUVI01.LOVIM`)
- ✅ ALL program names preserved verbatim (e.g., `MUHUO02I`, `MUBSZSFT`)
- ✅ ALL shared capability identifiers preserved verbatim
- ✅ Every use case flow step references at least one REQ-*

### Domain Model Completeness
- ✅ ALL aggregates with aggregate roots and entity field mapping tables
- ✅ Entity field mappings include: Domain Attribute, Source Field, PIC Type, Offset, Length
- ✅ ALL value objects fully specified (attributes, validation rules, format, immutability)
- ✅ ALL invariants listed with REQ-* references

### Event and Communication Completeness
- ✅ ALL events have trigger conditions with REQ-* references
- ✅ ALL events have full JSON schemas
- ✅ ALL events list their subscribers with BC identifiers
- ✅ ALL synchronous dependencies have SPECIFIC numeric resilience parameters
- ✅ Anti-corruption layers documented where applicable

### Domain Service Completeness
- ✅ ALL domain services have full typed operation signatures
- ✅ Dependencies listed (internal and external)
- ✅ Consuming use cases identified

### Use Case Completeness
- ✅ ALL use cases include actor, API endpoint, input/output
- ✅ ALL flow steps reference REQ-* (non-negotiable)
- ✅ ALL use cases include transaction boundary
- ✅ ALL use cases include error handling table
- ✅ ALL use cases reference legacy program names

### Mermaid Correctness
- ✅ Mermaid v11+ with ES Module import
- ✅ No `<<stereotype>>` syntax (use `note for` instead)
- ✅ No unquoted colons in node labels
- ✅ No unquoted special characters in edge labels
- ✅ Styles applied AFTER node/edge definitions
- ✅ All referenced nodes exist in diagram

### Styling
- ✅ Professional appearance with correct service color coding
- ✅ Responsive design (works at mobile widths)
- ✅ Tables have proper headers and zebra striping
- ✅ Code blocks are formatted and readable (monospace, background, padding)

---

## Key Principles

- **Match the reference file** — Every generated HTML must match the style, depth, and structure of `customer-management-service-spec.html`
- **One HTML file per microservice** — never combine specifications
- **One file at a time** — generate each HTML in isolation with full source reading before moving to the next
- **Never merge bounded contexts** — if the DDD model defines separate contexts, they remain separate HTML files
- **Self-contained files** — each HTML works standalone without a web server
- **Domain model as code blocks** — entities and value objects rendered as structured code in `.domain-model` boxes, NOT as plain tables
- **Method-colored API cards** — every endpoint gets its own styled card colored by HTTP method (GET=blue, POST=green, PUT=yellow, DELETE=red)
- **Full JSON schemas** — every API endpoint includes complete request AND response JSON schemas in code blocks
- **Ubiquitous Language glossary** — every service includes a definition list with 8+ domain terms
- **Domain services fully specified** — every domain service includes typed operation signatures, dependencies, REQ-* references, and consuming use cases
- **Use cases fully specified** — every application service/use case includes actor, API endpoint, input/output, complete flow with REQ-* on every step, transaction boundary, error handling, and legacy program names
- **Integration points in styled boxes** — each integration pattern in its own orange-bordered box with event schemas
- **Error response format** — standard error JSON structure documented per API endpoint in error response tables
- **Resilience parameters are concrete** — specify failure threshold, window, retry count, backoff strategy, and timeout for each dependency
- **Events are comprehensive** — publish events for primary state changes, cross-context operations, AND async/batch completions with full JSON schemas
- **REQ-* preserved** — ALL REQ-* and program name anchors survive the markdown-to-HTML conversion without modification
- **Professional presentation** — suitable for stakeholder reviews with color-coded sections and interactive navigation
- **No implementation code** — HTML files are documentation artifacts, not application code
- **Footer attribution** — every file ends with a footer crediting the DDD Microservices Analysis

---

## Troubleshooting

### Diagrams Not Rendering
1. Verify Mermaid v11+ with ES Module import
2. Check `startOnLoad: true` in initialization
3. Open browser console for error messages
4. Validate syntax against the rules in Step 6
5. Test in `test-mermaid.html` first

### Syntax Error in Mermaid
| Error | Cause | Fix |
|-------|-------|-----|
| Colon in node label | Unquoted special char | Wrap in quotes: `["Label"]` |
| `<<stereotype>>` error | Invalid v11 syntax | Use `note for Class "stereotype"` |
| Edge label parse error | Unquoted `/`, `{`, `}` | Quote edge labels: `\|"GET /api/{id}"\|` |
| Diagram partially renders | Styles before nodes | Move `style` directives to end |
| Node not found | Referenced before definition | Define all nodes before edges |

### Large Specifications (e.g., batch-data-operations)
- For services with many use cases (695+ programs), select representative use cases for sequence diagrams rather than diagramming all
- Group related API endpoints into flow diagrams by sub-domain
- Keep individual Mermaid diagrams to ≤15–20 nodes for readability
- Use multiple smaller diagrams rather than one massive diagram
- For the largest services, prioritize primary aggregates and key use cases in diagrams while ensuring ALL content is present in the HTML text sections

### Content Too Large for Single Write
- For very large specifications, generate the HTML in chunks if necessary
- Ensure all chunks are properly concatenated into a single valid HTML file
- Verify opening and closing tags match across chunks
