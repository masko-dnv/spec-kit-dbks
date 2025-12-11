---
description: "Task list template for data pipeline implementation"
---

# Tasks: [PIPELINE_NAME]

**Input**: Design documents from `/specs/[###-pipeline-name]/`
**Prerequisites**: plan.md (required), spec.md (required for data requirements), research.md, data-model.md, contracts/

**Tests**: The examples below include test tasks. Tests are OPTIONAL - only include them if explicitly requested in the data specification.

**Organization**: Tasks are grouped by data requirement to enable independent implementation and testing of each pipeline stage.

## Format: `[ID] [P?] [Req] Description`

- **[P]**: Can run in parallel (different files, no dependencies)
- **[Req]**: Which data requirement this task belongs to (e.g., DR1, DR2, DR3)
- Include exact file paths in descriptions

## Path Conventions

- **Standard pipeline**: `notebooks/`, `src/`, `tests/` at repository root
- **Multi-pipeline**: `notebooks/pipeline_a/`, `notebooks/pipeline_b/`
- **Streaming**: `notebooks/stream_*.py` for streaming, `notebooks/batch_*.py` for batch
- Paths shown below assume standard pipeline - adjust based on plan.md structure

<!--
  ============================================================================
  IMPORTANT: The tasks below are SAMPLE TASKS for illustration purposes only.

  The /speckit.tasks command MUST replace these with actual tasks based on:
  - Data requirements from spec.md (with their priorities P1, P2, P3...)
  - Pipeline configuration from plan.md
  - Entities from data-model.md (Bronze, Silver, Gold layer mappings)
  - Data contracts from contracts/

  Tasks MUST be organized by data requirement so each requirement can be:
  - Implemented independently
  - Tested independently
  - Delivered as an MVP increment

  DO NOT keep these sample tasks in the generated tasks.md file.
  ============================================================================
-->

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: Bundle initialization and environment configuration

- [ ] T001 Create project structure per implementation plan
- [ ] T002 Initialize Databricks Asset Bundle with databricks.yml
- [ ] T003 [P] Configure pyproject.toml with uv dependencies
- [ ] T004 [P] Set up .databrickscfg with workspace credentials
- [ ] T005 Verify databricks-connect session works locally

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Core infrastructure that MUST be complete before ANY data requirement can be implemented

**⚠️ CRITICAL**: No data requirement work can begin until this phase is complete

Examples of foundational tasks (adjust based on your pipeline):

- [ ] T006 Create Unity Catalog schema structure (Bronze, Silver, Gold)
- [ ] T007 [P] Create base DDL notebook in notebooks/_setup/create_tables.py
- [ ] T008 [P] Implement shared validators in src/validators/checks.py
- [ ] T009 [P] Create utility functions in src/utils/helpers.py
- [ ] T010 Configure databricks.yml with multi-target deployment (dev/staging/prod)
- [ ] T011 Set up test fixtures in fixtures/sample_data/

**Checkpoint**: Foundation ready - data requirement implementation can now begin in parallel

---

## Phase 3: Data Requirement 1 - [Title] (Priority: P1) 🎯 MVP

**Goal**: [Brief description of what this data requirement delivers]

**Independent Test**: [How to verify this requirement works on its own - e.g., query Gold table, check row counts]

### Tests for Data Requirement 1 (OPTIONAL - only if tests requested) ⚠️

> **NOTE: Write these tests FIRST, ensure they FAIL before implementation**

- [ ] T012 [P] [DR1] Unit test for ingestion logic in tests/unit/test_ingest_source.py
- [ ] T013 [P] [DR1] Integration test for pipeline in tests/integration/test_pipeline.py

### Implementation for Data Requirement 1

- [ ] T014 [P] [DR1] Create Bronze schema DDL for raw_[source] in notebooks/_setup/create_tables.py
- [ ] T015 [P] [DR1] Create Silver schema DDL for transformed_[entity] in notebooks/_setup/create_tables.py
- [ ] T016 [DR1] Implement ingest notebook in notebooks/ingest_[source].py
- [ ] T017 [DR1] Implement transform notebook in notebooks/transform_[entity].py
- [ ] T018 [DR1] Create transformation module in src/transformations/[entity].py
- [ ] T019 [DR1] Add data quality validators for DR1 entities
- [ ] T020 [DR1] Configure job definition in databricks.yml for DR1 pipeline

**Checkpoint**: At this point, Data Requirement 1 should be fully functional and testable independently

---

## Phase 4: Data Requirement 2 - [Title] (Priority: P2)

**Goal**: [Brief description of what this data requirement delivers]

**Independent Test**: [How to verify this requirement works on its own]

### Tests for Data Requirement 2 (OPTIONAL - only if tests requested) ⚠️

- [ ] T021 [P] [DR2] Unit test for transformation logic in tests/unit/test_transform_[entity].py
- [ ] T022 [P] [DR2] Integration test for DR2 pipeline in tests/integration/test_[entity].py

### Implementation for Data Requirement 2

- [ ] T023 [P] [DR2] Create Bronze schema DDL for raw_[source2] in notebooks/_setup/create_tables.py
- [ ] T024 [DR2] Implement ingest notebook in notebooks/ingest_[source2].py
- [ ] T025 [DR2] Implement transform notebook in notebooks/transform_[entity2].py
- [ ] T026 [DR2] Create transformation module in src/transformations/[entity2].py
- [ ] T027 [DR2] Integrate with Data Requirement 1 components (if needed)

**Checkpoint**: At this point, Data Requirements 1 AND 2 should both work independently

---

## Phase 5: Data Requirement 3 - [Title] (Priority: P3)

**Goal**: [Brief description of what this data requirement delivers]

**Independent Test**: [How to verify this requirement works on its own]

### Tests for Data Requirement 3 (OPTIONAL - only if tests requested) ⚠️

- [ ] T028 [P] [DR3] Unit test for aggregation logic in tests/unit/test_output_[metric].py
- [ ] T029 [P] [DR3] Integration test for Gold layer in tests/integration/test_gold.py

### Implementation for Data Requirement 3

- [ ] T030 [P] [DR3] Create Gold schema DDL for agg_[metric] in notebooks/_setup/create_tables.py
- [ ] T031 [DR3] Implement output notebook in notebooks/output_[metric].py
- [ ] T032 [DR3] Create aggregation module in src/output/aggregations.py
- [ ] T033 [DR3] Add final data quality validators

**Checkpoint**: All data requirements should now be independently functional

---

[Add more data requirement phases as needed, following the same pattern]

---

## Phase N: Polish & Cross-Cutting Concerns

**Purpose**: Improvements that affect multiple data requirements

- [ ] TXXX [P] Documentation updates in docs/
- [ ] TXXX Code cleanup and refactoring
- [ ] TXXX Performance optimization across all pipeline stages
- [ ] TXXX [P] Additional unit tests (if requested) in tests/unit/
- [ ] TXXX Configure CI/CD in .github/workflows/
- [ ] TXXX Run quickstart.md validation

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: No dependencies - can start immediately
- **Foundational (Phase 2)**: Depends on Setup completion - BLOCKS all data requirements
- **Data Requirements (Phase 3+)**: All depend on Foundational phase completion
  - Data requirements can then proceed in parallel (if staffed)
  - Or sequentially in priority order (P1 → P2 → P3)
- **Polish (Final Phase)**: Depends on all desired data requirements being complete

### Data Requirement Dependencies

- **Data Requirement 1 (P1)**: Can start after Foundational (Phase 2) - No dependencies on other requirements
- **Data Requirement 2 (P2)**: Can start after Foundational (Phase 2) - May integrate with DR1 but should be independently testable
- **Data Requirement 3 (P3)**: Can start after Foundational (Phase 2) - May integrate with DR1/DR2 but should be independently testable

### Within Each Data Requirement

- Tests (if included) MUST be written and FAIL before implementation
- Schema DDL before notebooks
- Ingest before transform
- Transform before output
- Core implementation before integration
- Requirement complete before moving to next priority

### Parallel Opportunities

- All Setup tasks marked [P] can run in parallel
- All Foundational tasks marked [P] can run in parallel (within Phase 2)
- Once Foundational phase completes, all data requirements can start in parallel (if team capacity allows)
- All tests for a requirement marked [P] can run in parallel
- Schema DDL tasks within a requirement marked [P] can run in parallel
- Different data requirements can be worked on in parallel by different team members

---

## Parallel Example: Data Requirement 1

```bash
# Launch all tests for Data Requirement 1 together (if tests requested):
Task: "Unit test for ingestion logic in tests/unit/test_ingest_source.py"
Task: "Integration test for pipeline in tests/integration/test_pipeline.py"

# Launch all schema DDL for Data Requirement 1 together:
Task: "Create Bronze schema DDL for raw_[source] in notebooks/_setup/create_tables.py"
Task: "Create Silver schema DDL for transformed_[entity] in notebooks/_setup/create_tables.py"
```

---

## Implementation Strategy

### MVP First (Data Requirement 1 Only)

1. Complete Phase 1: Setup
2. Complete Phase 2: Foundational (CRITICAL - blocks all requirements)
3. Complete Phase 3: Data Requirement 1
4. **STOP and VALIDATE**: Test Data Requirement 1 independently
5. Deploy/demo if ready

### Incremental Delivery

1. Complete Setup + Foundational → Foundation ready
2. Add Data Requirement 1 → Test independently → Deploy/Demo (MVP!)
3. Add Data Requirement 2 → Test independently → Deploy/Demo
4. Add Data Requirement 3 → Test independently → Deploy/Demo
5. Each requirement adds value without breaking previous requirements

### Parallel Team Strategy

With multiple developers:

1. Team completes Setup + Foundational together
2. Once Foundational is done:
   - Developer A: Data Requirement 1 (Bronze → Silver for source 1)
   - Developer B: Data Requirement 2 (Bronze → Silver for source 2)
   - Developer C: Data Requirement 3 (Silver → Gold aggregations)
3. Requirements complete and integrate independently

---

## Notes

- [P] tasks = different files, no dependencies
- [Req] label maps task to specific data requirement for traceability
- Each data requirement should be independently completable and testable
- Verify tests fail before implementing
- Commit after each task or logical group
- Stop at any checkpoint to validate requirement independently
- Avoid: vague tasks, same file conflicts, cross-requirement dependencies that break independence
