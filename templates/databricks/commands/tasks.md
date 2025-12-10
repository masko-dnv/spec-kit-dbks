---
description: Break down the implementation plan into detailed, actionable tasks organized by phase with clear priorities.
handoffs: 
  - label: Start Implementation
    agent: speckit.clarify
    prompt: Help me clarify specific aspects of the task list I can start working on
    send: false
scripts:
  sh: scripts/bash/create-new-feature.sh --json "{ARGS}"
  ps: scripts/powershell/create-new-feature.ps1 -Json "{ARGS}"
---

## User Input

```text
$ARGUMENTS
```

## Overview

You are creating a **detailed task list** for implementing a Databricks data pipeline. Break down the implementation plan into concrete, actionable tasks organized by phase with clear priorities (P1, P2) and dependencies.

---

## Your Task

Create a comprehensive task list that includes:

### 1. **Phase Breakdown**
Organize tasks into 7 implementation phases:

- **Phase 1**: Bundle Setup & Environment Configuration
- **Phase 2**: Schema & Infrastructure (Bronze, Silver, Gold layers)
- **Phase 3**: Data Ingestion (source connectors, validation)
- **Phase 4**: Data Transformations (business logic, deduplication)
- **Phase 5**: Output & Aggregation (metrics, features)
- **Phase 6**: Orchestration & Deployment (jobs, CI/CD)
- **Phase 7**: Documentation & Polish (docs, testing, sign-off)

### 2. **Task Structure**

For each task, use this format:

```markdown
- [ ] T### [P#] [Description of concrete deliverable]
```

Where:
- **T###**: Task ID (T001, T002, etc.)
- **[P#]**: Priority (P1=Must Have, P2=Should Have)
- **Description**: Specific, measurable, actionable outcome

### 3. **Task Categories**

Organize tasks within each phase:

#### Phase 1: Bundle Setup
- Initialization tasks (bundle init, credentials, configuration)
- Local development setup (Python env, databricks-connect)
- Repository setup (.gitignore, docs structure, README)
- Verification (validate configuration, test connections)

#### Phase 2: Schema & Infrastructure
- Bronze layer (raw data table definitions)
- Silver layer (cleaned data schemas)
- Gold layer (business-ready table definitions)
- Test data and fixtures
- Documentation (data lineage, schema docs)

#### Phase 3: Data Ingestion
- Ingest notebook creation
- Connector and module implementations
- Unit tests for ingestion logic
- Job definition and scheduling
- Manual verification of data loading

#### Phase 4: Data Transformations
- Transform notebook creation
- Transformation module extraction
- Data quality validators
- Unit and integration tests
- Performance optimization

#### Phase 5: Output & Aggregation
- Output notebook creation
- Aggregation modules
- Final data quality checks
- Integration tests
- Job definition for output layer

#### Phase 6: Orchestration & CI/CD
- Complete job configuration with dependencies
- GitHub Actions CI/CD pipeline
- Testing and validation automation
- Multi-environment deployment
- Production deployment and verification

#### Phase 7: Documentation & Polish
- Code documentation and docstrings
- User documentation (README, quickstart, troubleshooting)
- Operational documentation (runbooks, recovery)
- Code quality checks (linting, formatting, types)
- Final sign-off and reviews

### 4. **Task Dependencies**

Show which tasks depend on others:
- Indicate when tasks must be completed sequentially
- Group independent tasks that can run in parallel
- Mark critical path tasks

### 5. **Priority Levels**

- **P1 (Must Have)**: Core functionality, required for MVP
- **P2 (Should Have)**: Nice-to-have features, post-launch optimization

### 6. **Progress Tracking**

Include a summary table:
- Overall completion percentage
- Completion by phase
- Task count summary (total P1 vs P2)

---

## Example Task List Structure

```markdown
# Tasks: [PIPELINE_NAME]

## Phase 1: Bundle Setup & Environment Configuration

### Initialization
- [ ] T001 [P1] Initialize Databricks Asset Bundle with databricks bundle init
- [ ] T002 [P1] Create .databrickscfg with workspace credentials
- [ ] T003 [P1] Configure databricks.yml with catalog, schema, and target settings

### Local Development Environment
- [ ] T010 [P1] Create Python virtual environment (uv sync)
- [ ] T011 [P1] Install databricks-connect matching DBR version
- [ ] T012 [P1] Configure VS Code settings for Databricks Connect

### Verification
- [ ] T020 [P1] Run databricks bundle validate successfully
- [ ] T021 [P1] Confirm Databricks Connect session works locally

## Phase 2: Schema & Infrastructure

### Bronze Layer
- [ ] T101 [P1] Create setup notebook (notebooks/_setup/create_tables.py)
- [ ] T102 [P1] Define Bronze schema DDL for each raw data table
  - [ ] T102a: raw_source_1 table definition
  - [ ] T102b: raw_source_2 table definition

### Silver Layer
- [ ] T105 [P1] Define Silver schema DDL for transformed entities
  - [ ] T105a: transformed_entity_1 table definition

[Continue for remaining phases...]

## Progress Tracking

| Phase | Status | Tasks | Completion % |
|-------|--------|-------|--------------|
| 1: Setup | [ ] | 15 | 0% |
| 2: Schema | [ ] | 20 | 0% |
[Rest of phases...]
```

---

## Tips for Success

- **Be specific**: Each task should be completable in 1-4 hours
- **Use concrete deliverables**: "Create X notebook" not "Work on notebooks"
- **Link to artifacts**: Reference specific files, notebooks, or configuration
- **Set realistic priorities**: P1 for must-have, P2 for polish
- **Show dependencies**: Indicate what must be done first
- **Include verification**: How to confirm each task is complete
- **Group related tasks**: Keep logical units together
- **Balance phases**: Spread work across team members

---

## Output Considerations

The task list should be:
1. **Comprehensive**: Cover all aspects of building the pipeline
2. **Actionable**: Clear enough that someone can pick up a task and work independently
3. **Traceable**: Easy to mark progress and see what's done
4. **Realistic**: Consider actual effort required for Databricks development
5. **Flexible**: Allow for parallelization where possible

---

## Next Steps

Once you have the task list, team members can:
1. Pick P1 tasks to complete the MVP
2. Assign P2 tasks for post-launch
3. Parallelize independent tasks
4. Track progress using the checklist
5. Identify blockers early
