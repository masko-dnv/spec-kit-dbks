---
description: Execute the implementation planning workflow for a Databricks data pipeline using the plan template to generate design artifacts.
handoffs:
  - label: Create Tasks
    agent: speckit.tasks
    prompt: Break the plan into tasks
    send: true
  - label: Create Checklist
    agent: speckit.checklist
    prompt: Create a checklist for the following domain...
scripts:
  sh: scripts/bash/setup-plan.sh --json
  ps: scripts/powershell/setup-plan.ps1 -Json
agent_scripts:
  sh: scripts/bash/update-agent-context.sh __AGENT__
  ps: scripts/powershell/update-agent-context.ps1 -AgentType __AGENT__
---

## User Input

```text
$ARGUMENTS
```

You **MUST** consider the user input before proceeding (if not empty).

## Outline

1. **Setup**: Run `{SCRIPT}` from repo root and parse JSON for FEATURE_SPEC, IMPL_PLAN, SPECS_DIR, BRANCH. For single quotes in args like "I'm Groot", use escape syntax: e.g 'I'\''m Groot' (or double-quote if possible: "I'm Groot").

2. **Load context**: Read FEATURE_SPEC and `/memory/constitution.md`. Load IMPL_PLAN template (already copied).

3. **Execute plan workflow**: Follow the structure in IMPL_PLAN template to:
   - Fill Technical Context (mark unknowns as "NEEDS CLARIFICATION")
   - Fill Constitution Check section from constitution
   - Evaluate gates (ERROR if violations unjustified)
   - Phase 0: Generate research.md (resolve all NEEDS CLARIFICATION)
   - Phase 1: Generate data-model.md, contracts/, quickstart.md
   - Phase 1: Update agent context by running the agent script
   - Re-evaluate Constitution Check post-design

4. **Stop and report**: Command ends after Phase 1 planning. Report branch, IMPL_PLAN path, and generated artifacts.

## Phases

### Phase 0: Outline & Research

1. **Extract unknowns from Technical Context** above:
   - For each NEEDS CLARIFICATION → research task
   - For each Databricks dependency (DBR version, Unity Catalog, compute type) → best practices task
   - For each integration (source systems, Delta Lake, downstream consumers) → patterns task

2. **Generate and dispatch research agents**:

   ```text
   For each unknown in Technical Context:
     Task: "Research {unknown} for {pipeline context}"
   For each technology choice:
     Task: "Find best practices for {tech} in Databricks/PySpark"
   ```

3. **Consolidate findings** in `research.md` using format:
   - Decision: [what was chosen]
   - Rationale: [why chosen]
   - Alternatives considered: [what else evaluated]

**Output**: research.md with all NEEDS CLARIFICATION resolved

### Phase 1: Design & Contracts

**Prerequisites:** `research.md` complete

1. **Extract entities from feature spec** → `data-model.md`:
   - Entity name, fields, relationships
   - Bronze/Silver/Gold layer mappings
   - Delta Lake schema definitions
   - Validation rules from requirements
   - State transitions if applicable

2. **Generate pipeline contracts** from functional requirements:
   - For each data source → ingestion contract (schema, frequency, SLA)
   - For each transformation → input/output schema contract
   - For each output → delivery contract (format, destination, freshness)
   - Output schemas to `/contracts/`

3. **Define project structure**:
   ```
   [project-name]/
   ├── .github/                    # CI/CD workflows
   ├── .vscode/                    # VS Code settings
   ├── notebooks/                  # Databricks notebooks
   │   ├── _setup/                 # Schema setup DDL
   │   ├── ingest_*.py             # Bronze layer
   │   ├── transform_*.py          # Silver layer
   │   └── output_*.py             # Gold layer
   ├── src/                        # Reusable Python modules
   ├── tests/
   │   ├── unit/                   # No Databricks connection
   │   └── integration/            # With databricks-connect
   ├── fixtures/sample_data/       # Test data
   ├── databricks.yml              # Asset Bundle config
   └── pyproject.toml              # Dependencies (uv)
   ```

4. **Agent context update**:
   - Run `{AGENT_SCRIPT}`
   - These scripts detect which AI agent is in use
   - Update the appropriate agent-specific context file
   - Add only new technology from current plan (PySpark, Delta Lake, Unity Catalog)
   - Preserve manual additions between markers

**Output**: data-model.md, /contracts/*, quickstart.md, agent-specific file

## Key rules

- Use absolute paths
- ERROR on gate failures or unresolved clarifications
- Mark unknowns as "NEEDS CLARIFICATION" (DBR version, compute type, Unity Catalog schema)
- Databricks-specific: validate Unity Catalog access, cluster permissions, and Delta Lake compatibility
