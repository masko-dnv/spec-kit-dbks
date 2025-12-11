---
description: Generate an actionable, dependency-ordered tasks.md for the data pipeline based on available design artifacts.
handoffs:
  - label: Analyze For Consistency
    agent: speckit.analyze
    prompt: Run a project analysis for consistency
    send: true
  - label: Implement Project
    agent: speckit.implement
    prompt: Start the implementation in phases
    send: true
scripts:
  sh: scripts/bash/check-prerequisites.sh --json
  ps: scripts/powershell/check-prerequisites.ps1 -Json
---

## User Input

```text
$ARGUMENTS
```

You **MUST** consider the user input before proceeding (if not empty).

## Outline

1. **Setup**: Run `{SCRIPT}` from repo root and parse FEATURE_DIR and AVAILABLE_DOCS list. All paths must be absolute. For single quotes in args like "I'm Groot", use escape syntax: e.g 'I'\''m Groot' (or double-quote if possible: "I'm Groot").

2. **Load design documents**: Read from FEATURE_DIR:
   - **Required**: plan.md (tech stack, Databricks config, project structure), spec.md (data requirements with priorities)
   - **Optional**: data-model.md (entities, Bronze/Silver/Gold schemas), contracts/ (data contracts), research.md (decisions), quickstart.md (test scenarios)
   - Note: Not all projects have all documents. Generate tasks based on what's available.

3. **Execute task generation workflow**:
   - Load plan.md and extract tech stack, DBR version, Unity Catalog config, project structure
   - Load spec.md and extract data requirements with their priorities (P1, P2, P3, etc.)
   - If data-model.md exists: Extract entities and map to data layers (Bronze, Silver, Gold)
   - If contracts/ exists: Map data contracts to pipeline stages
   - If research.md exists: Extract decisions for setup tasks
   - Generate tasks organized by data requirement (see Task Generation Rules below)
   - Generate dependency graph showing pipeline stage completion order
   - Create parallel execution examples per data layer
   - Validate task completeness (each data requirement has all needed tasks, independently testable)

4. **Generate tasks.md**: Use `templates/tasks-template.md` as structure, fill with:
   - Correct pipeline name from plan.md
   - Phase 1: Setup tasks (bundle initialization, environment configuration)
   - Phase 2: Foundational tasks (blocking prerequisites - schemas, infrastructure)
   - Phase 3+: One phase per data requirement (in priority order from spec.md)
   - Each phase includes: requirement goal, independent test criteria, tests (if requested), implementation tasks
   - Final Phase: Polish & cross-cutting concerns
   - All tasks must follow the strict checklist format (see Task Generation Rules below)
   - Clear file paths for each task
   - Dependencies section showing pipeline stage completion order
   - Parallel execution examples per layer
   - Implementation strategy section (MVP first, incremental delivery)

5. **Report**: Output path to generated tasks.md and summary:
   - Total task count
   - Task count per data requirement
   - Parallel opportunities identified
   - Independent test criteria for each requirement
   - Suggested MVP scope (typically just Data Requirement 1)
   - Format validation: Confirm ALL tasks follow the checklist format (checkbox, ID, labels, file paths)

Context for task generation: {ARGS}

The tasks.md should be immediately executable - each task must be specific enough that an LLM can complete it without additional context.

## Task Generation Rules

**CRITICAL**: Tasks MUST be organized by data requirement to enable independent implementation and testing.

**Tests are OPTIONAL**: Only generate test tasks if explicitly requested in the data specification or if user requests TDD approach.

### Checklist Format (REQUIRED)

Every task MUST strictly follow this format:

```text
- [ ] [TaskID] [P?] [Req?] Description with file path
```

**Format Components**:

1. **Checkbox**: ALWAYS start with `- [ ]` (markdown checkbox)
2. **Task ID**: Sequential number (T001, T002, T003...) in execution order
3. **[P] marker**: Include ONLY if task is parallelizable (different files, no dependencies on incomplete tasks)
4. **[Req] label**: REQUIRED for data requirement phase tasks only
   - Format: [DR1], [DR2], [DR3], etc. (maps to data requirements from spec.md)
   - Setup phase: NO requirement label
   - Foundational phase: NO requirement label
   - Data Requirement phases: MUST have requirement label
   - Polish phase: NO requirement label
5. **Description**: Clear action with exact file path

**Examples**:

- ✅ CORRECT: `- [ ] T001 Create project structure per implementation plan`
- ✅ CORRECT: `- [ ] T005 [P] Configure databricks.yml with Unity Catalog settings`
- ✅ CORRECT: `- [ ] T012 [P] [DR1] Create Bronze schema DDL in notebooks/_setup/create_tables.py`
- ✅ CORRECT: `- [ ] T014 [DR1] Implement ingest notebook in notebooks/ingest_source.py`
- ❌ WRONG: `- [ ] Create ingest notebook` (missing ID and Requirement label)
- ❌ WRONG: `T001 [DR1] Create notebook` (missing checkbox)
- ❌ WRONG: `- [ ] [DR1] Create ingest notebook` (missing Task ID)
- ❌ WRONG: `- [ ] T001 [DR1] Create notebook` (missing file path)

### Task Organization

1. **From Data Requirements (spec.md)** - PRIMARY ORGANIZATION:
   - Each data requirement (P1, P2, P3...) gets its own phase
   - Map all related components to their requirement:
     - Schema definitions needed for that requirement
     - Notebooks needed for that requirement (ingest, transform, output)
     - Validators needed for that requirement
     - If tests requested: Tests specific to that requirement
   - Mark requirement dependencies (most requirements should be independent)

2. **From Data Contracts**:
   - Map each contract → to the data requirement it serves
   - If tests requested: Each contract → contract test task [P] before implementation in that requirement's phase

3. **From Data Model**:
   - Map each entity to the data requirement(s) that need it
   - Bronze/Silver/Gold layer mappings → appropriate requirement phase
   - If entity serves multiple requirements: Put in earliest requirement or Foundational phase

4. **From Setup/Infrastructure**:
   - Shared infrastructure → Setup phase (Phase 1)
   - Foundational/blocking tasks (Unity Catalog, base schemas) → Foundational phase (Phase 2)
   - Requirement-specific setup → within that requirement's phase

### Phase Structure

- **Phase 1**: Setup (bundle initialization, environment configuration, databricks-connect)
- **Phase 2**: Foundational (blocking prerequisites - MUST complete before data requirements)
- **Phase 3+**: Data Requirements in priority order (P1, P2, P3...)
  - Within each requirement: Tests (if requested) → Schemas → Notebooks → Validators → Integration
  - Each phase should be a complete, independently testable increment
- **Final Phase**: Polish & Cross-Cutting Concerns
