---
name: phase-1-business-function-analysis
description: Comprehensive analysis of all business function inputs (requirements documents) to produce a consolidated ddd-analysis.md document. Use when starting legacy system analysis, reading business requirements, or preparing input for DDD bounded context identification.
---

# Phase 1: Business Function Comprehensive Analysis

## Purpose

This skill guides the comprehensive analysis of all business function inputs — requirements documents — to produce a single consolidated markdown document (`ddd-analysis.md`). That document serves as the sole input for Phase 2 (DDD bounded context analysis).

## Goal

Read, cross-reference, and synthesize every artifact in the `inputs/spec/` folder into a structured, self-contained analysis document that preserves full traceability to the original sources. The output must be rich enough that the DDD bounded context analysis can proceed without re-reading the raw inputs.

## Execution Model: Chunked Output

**CRITICAL**: Do NOT attempt to produce the entire `ddd-analysis.md` document in a single pass. The volume of data across all business functions (typically 20–40 source files, 1000+ requirements) will exceed context limits and produce incomplete or shallow output.

**Instead, produce the document in sequential chunks:**

### Chunk 1: Sections 1 and 2 (Inventory + Detailed Function Analysis)

1. List all subdirectories under `inputs/spec/` (excluding `_shared/` and dot-files) to build the function inventory
2. Read all `requirements.md` files to extract REQ-* identifiers and workflow summaries
3. Read shared capabilities from `_shared/` to understand cross-function dependencies (requirements.md files)
4. **Write** the `ddd-analysis.md` file with Section 1 (Business Functions Inventory) and Section 2 (Business Function Detailed Analysis for all functions)

### Chunk 2: Section 3 (Data Store Analysis)

1. Read `data-model.md` for the system-wide data store access map (data store names, types, and which capabilities access them)
2. Read `.glossary.yaml` for data store and field descriptions
3. Cross-reference requirements for program-to-data-store relationships (inferred from requirement descriptions and program names mentioned in requirements)
4. **Append** Section 3 to `ddd-analysis.md` (Data Store Inventory, Data Ownership Matrix, Entity Relationships, Shared Data Hotspots)

### Chunk 3: Section 4 (Programs and Business Rules)

1. Extract program names mentioned in requirements documents
2. Build a program inventory grouped by function
3. **Append** Section 4 to `ddd-analysis.md` (Program Inventory, Program-to-Function Map, Business Rule Distribution by Program)

### Chunk 4: Section 5 (Cross-Function Synthesis + Appendix)

1. Synthesize cross-function interactions from data already gathered
2. **Append** Section 5 to `ddd-analysis.md` (Function Interaction Map, Business Rule Distribution, Integration Points Summary)
3. **Append** Appendix A (Source File References)

### Why Chunked Execution

- **Prevents context overflow**: Each chunk reads a focused subset of files and produces a bounded output section
- **Enables incremental verification**: Each chunk can be reviewed before proceeding
- **Preserves depth**: Smaller focused passes produce richer cross-referencing than one shallow pass over everything
- **Handles large systems**: Systems with 10+ business functions and hundreds of rules cannot fit in a single context window

### File Operations

- **Chunk 1**: Use file-write to create `ddd-analysis.md` with Sections 1–2
- **Chunks 2–4**: Use file-append to add Sections 3, 4, and 5 to the existing file
- Each chunk should read its required source files, produce the output, then move to the next chunk

## Input Sources

### 1. System-Level Files

| Source | Path | Content |
|--------|------|---------|
| Data Model | `inputs/spec/data-model.md` | System-wide data model listing all data stores (type, accessing capabilities) |
| Glossary | `inputs/spec/.glossary.yaml` | Domain glossary with data store descriptions and field definitions |

### 2. Business Function Specifications

Located under `inputs/spec/`. Each subdirectory (excluding `_shared/` and dot-files) is a business function:

| Source | Path Pattern | Content |
|--------|-------------|---------|
| Requirements | `inputs/spec/<FunctionName>/requirements.md` | User workflows, functional requirements (REQ-*), preconditions, cross-boundary constraints |

### 3. Shared Capabilities

Located under `inputs/spec/_shared/`. Each subdirectory is a shared subroutine/program used across multiple business functions:

| Source | Path Pattern | Content |
|--------|-------------|---------|
| Requirements | `inputs/spec/_shared/<ProgramName>/requirements.md` | Functional requirements for the shared program (REQ-* identifiers) |

## Analysis Process

Execute the following steps sequentially. Each step builds on the previous one.

### Step 1: Discover and Inventory All Business Functions

1. List all subdirectories under `inputs/spec/` (excluding `_shared/` and dot-files) — each is a business function
2. For each subdirectory, verify the presence of `requirements.md`
3. Read `inputs/spec/data-model.md` to determine which data stores each function accesses
4. Inventory shared capabilities from `inputs/spec/_shared/` — list each shared program directory and read its requirements.md
5. Build a complete inventory

**Output section: "Business Functions Inventory"**

For each function, record:
- Function name (directory name)
- Whether requirements.md exists
- Programs involved (extracted from requirement descriptions and program names mentioned in requirements)
- Data stores accessed (from data-model.md `Accessed By` column)
- Dependencies on shared capabilities (inferred from shared programs that reference similar data stores or are called by this function's programs)

### Step 2: Analyze Each Business Function

For each discovered business function, read its `requirements.md` in full. Produce a structured summary that captures:

#### 2a. Requirements Summary

For each function's `requirements.md`:
- **Function purpose**: One-paragraph description of what this function does (derived from the requirements introduction and navigation context)
- **Actors**: Who uses this function (user roles)
- **Preconditions**: Global preconditions listed at the top
- **User workflows**: List each numbered section with its title and a brief description
- **Key requirements inventory**: Table of all REQ-* identifiers grouped by workflow, with a one-line summary of each
- **Cross-boundary requirements**: Any REQ-XBND-*, REQ-INTEG-*, REQ-LOCK-* or similar cross-cutting requirements that indicate integration points with other functions
- **Validation rules**: Any REQ-VALID-* requirements that define input validation
- **Error handling patterns**: Key error conditions and their handling
- **Open questions**: Any OQ-* items listed at the end of the requirements document

**IMPORTANT**: Preserve all REQ-* identifiers exactly as they appear. These are traceability anchors.

#### 2b. Shared Dependencies Summary

For each shared capability in `_shared/` that is referenced by this function's programs (identified by matching program names in CALL patterns from requirement descriptions, or by explicit cross-references in the requirements):
- List the shared capability with its program name
- Summarize its requirements (from `_shared/<ProgramName>/requirements.md`)

### Step 3: Analyze Data Stores (Data Model)

Read `inputs/spec/data-model.md` and `.glossary.yaml` to produce:

#### 3a. Data Store Inventory

For each data store listed in data-model.md:
- Store name (full DSN, e.g., `AWS.M2.CARDDEMO.ACCTDATA.VSAM.KSDS`)
- Type (VSAM KSDS, VSAM PATH, QSAM, GDG Base, Non VSAM, DB2)
- Accessing capabilities (from the `Accessed By` column)
- Description (from .glossary.yaml `tables` section)

#### 3b. Data Ownership Matrix

Build a matrix showing which functions access each data store. Use `data-model.md` as the primary source:

| Data Store | Type | Accessing Functions |
|-----------|------|---------------------|

#### 3c. Entity Relationships

Identify relationships between data stores by analyzing:
- Shared naming patterns (e.g., ACCTDATA, CARDDATA, CARDXREF suggest account-card relationships)
- Cross-reference stores (stores with XREF in name link other entities)
- Alternate index paths (AIX.PATH stores provide secondary access to primary stores)
- Field descriptions from .glossary.yaml that mention related entities

#### 3d. Shared Data Hotspots

Identify data stores accessed by 3+ functions — these are integration hotspots that will drive bounded context boundaries:
- List each hotspot store
- List all functions that access it
- Note the likely contention pattern (read-heavy, write-heavy, mixed — inferred from function purposes)

### Step 4: Analyze Programs and Business Rules

Extract program information from requirements documents and produce:

#### 4a. Program Inventory

From each function's `requirements.md`, extract all program names mentioned:
- Group by function
- For each program: name, inferred role based on requirements context
- Classify by type based on naming conventions and requirement context (batch programs like CBACTxxC, online programs like COACTxx, shared subroutines)

#### 4b. Program-to-Function Map

Build a map showing which programs belong to which function:

| Program | Function | Role |
|---------|----------|------|

#### 4c. Business Rule Distribution by Program

For key programs, summarize what business logic they implement based on requirements:
- Group requirements by program
- Identify the primary responsibility of each program based on requirement descriptions
- Note which data stores are referenced in requirements (e.g., "read from the account file", "write to the transaction file")

### Step 5: Cross-Reference and Synthesize

Produce cross-cutting analysis sections:

#### 5a. Function Interaction Map

Using the data ownership matrix, cross-function data store access, and cross-boundary requirements, build a function interaction map:
- Which functions share access to the same data stores (from data-model.md)
- What the nature of each dependency is (read-only lookup, shared write, inferred from function purposes)
- Which requirements drive each interaction (cite REQ-* identifiers from cross-boundary requirements)

#### 5b. Business Rule Distribution

Map business rules to functional areas:
- Which requirements validate or transform data
- Which requirements span multiple data stores (inferred from requirement descriptions mentioning multiple files/stores)
- Which requirements are function-local vs cross-function (based on data store ownership from data-model.md)

#### 5c. Integration Points Summary

Consolidate all evidence of cross-function integration:
- Cross-boundary requirements (REQ-XBND-*, REQ-INTEG-*)
- Data stores accessed by multiple functions (from data-model.md)
- Message queue interactions (inferred from requirement descriptions mentioning MQ operations, queue names)
- Shared subroutine dependencies (from _shared/ capabilities)

## Output Document Structure

The output file `ddd-analysis.md` must follow this exact structure:

```markdown
# CardDemo System — Comprehensive Business Function Analysis

## Document Purpose
<Brief statement that this document consolidates all business function analysis
 and serves as the input for DDD bounded context identification>

## 1. Business Functions Inventory
<Step 1 output>

## 2. Business Function Detailed Analysis

### 2.1 <FunctionName>
#### 2.1.1 Requirements Summary
#### 2.1.2 Shared Dependencies Summary

### 2.2 <FunctionName>
...
<Repeat for all functions>

## 3. Data Store Analysis

### 3.1 Data Store Inventory
### 3.2 Data Ownership Matrix
### 3.3 Entity Relationships
### 3.4 Shared Data Hotspots

## 4. Program and Business Rule Analysis

### 4.1 Program Inventory
### 4.2 Program-to-Function Map
### 4.3 Business Rule Distribution by Program

## 5. Cross-Function Synthesis

### 5.1 Function Interaction Map
### 5.2 Business Rule Distribution
### 5.3 Integration Points Summary

## Appendix A: Source File References

<Complete listing of all input files read during this analysis,
 with their relative paths, for full traceability>
```

## Traceability Rules

Throughout the output document, maintain these traceability conventions:

1. **Requirement references**: Always cite as `REQ-XXX-NNN` exactly as they appear in the source requirements.md
2. **Data store references**: Always cite using the full DSN name as it appears in data-model.md (e.g., `AWS.M2.CARDDEMO.ACCTDATA.VSAM.KSDS`)
3. **Program references**: Always cite the program name as it appears in requirements (e.g., `CBACT01C`, `COACCT01`)
4. **Shared capability references**: Always cite the shared program name as it appears in the `_shared/` directory name (e.g., `COMEN01C`)
5. **Source file references**: When citing a specific fact, note the source file path in parentheses

## Key Principles

- **Completeness over brevity**: Include all requirements and all data stores. The DDD analysis cannot go back to re-read inputs.
- **Preserve identifiers**: Every REQ-*, program name, and data store identifier must be preserved verbatim for traceability.
- **Cross-reference aggressively**: The value of this document is in connecting requirements to data stores to programs. Make these connections explicit.
- **Include shared capabilities**: The `_shared/` programs represent cross-cutting logic used by multiple business functions — their requirements must be attributed to the consuming functions.
- **Flag ambiguities**: If ownership is unclear or if data shows unexpected patterns, call them out explicitly as items for the DDD analysis to resolve.
- **Separate facts from inference**: Clearly mark any inferred relationships or assumptions with [INFERRED] tags.
- **Use data-model.md as ground truth for data access**: The `Accessed By` column definitively shows which capabilities access which data stores.
- **Use .glossary.yaml for descriptions**: The glossary provides human-readable descriptions of data stores and fields that aid understanding.
