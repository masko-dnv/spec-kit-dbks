---
description: Execute the implementation plan by processing and executing all tasks defined in tasks.md for Databricks data pipelines.
scripts:
  sh: scripts/bash/check-prerequisites.sh --json --require-tasks --include-tasks
  ps: scripts/powershell/check-prerequisites.ps1 -Json -RequireTasks -IncludeTasks
---

## User Input

```text
$ARGUMENTS
```

You **MUST** consider the user input before proceeding (if not empty).

## Outline

1. Run `{SCRIPT}` from repo root and parse FEATURE_DIR and AVAILABLE_DOCS list. All paths must be absolute. For single quotes in args like "I'm Groot", use escape syntax: e.g 'I'\''m Groot' (or double-quote if possible: "I'm Groot").

2. **Check checklists status** (if FEATURE_DIR/checklists/ exists):
   - Scan all checklist files in the checklists/ directory
   - For each checklist, count:
     - Total items: All lines matching `- [ ]` or `- [X]` or `- [x]`
     - Completed items: Lines matching `- [X]` or `- [x]`
     - Incomplete items: Lines matching `- [ ]`
   - Create a status table:

     ```text
     | Checklist | Total | Completed | Incomplete | Status |
     |-----------|-------|-----------|------------|--------|
     | data-quality.md | 12 | 12     | 0          | ✓ PASS |
     | testing.md      | 8  | 5      | 3          | ✗ FAIL |
     | security.md     | 6  | 6      | 0          | ✓ PASS |
     ```

   - Calculate overall status:
     - **PASS**: All checklists have 0 incomplete items
     - **FAIL**: One or more checklists have incomplete items

   - **If any checklist is incomplete**:
     - Display the table with incomplete item counts
     - **STOP** and ask: "Some checklists are incomplete. Do you want to proceed with implementation anyway? (yes/no)"
     - Wait for user response before continuing
     - If user says "no" or "wait" or "stop", halt execution
     - If user says "yes" or "proceed" or "continue", proceed to step 3

   - **If all checklists are complete**:
     - Display the table showing all checklists passed
     - Automatically proceed to step 3

3. Load and analyze the implementation context:
   - **REQUIRED**: Read tasks.md for the complete task list and execution plan
   - **REQUIRED**: Read plan.md for tech stack, architecture, medallion layer design, and file structure
   - **IF EXISTS**: Read data-model.md for entities, schemas, and table relationships
   - **IF EXISTS**: Read contracts/ for API specifications and test requirements
   - **IF EXISTS**: Read research.md for technical decisions and constraints
   - **IF EXISTS**: Read quickstart.md for integration scenarios

4. **Project Setup Verification**:
   - **REQUIRED**: Create/verify ignore files based on actual project setup:

   **Detection & Creation Logic**:
   - Check if the following command succeeds to determine if the repository is a git repo (create/verify .gitignore if so):

     ```sh
     git rev-parse --git-dir 2>/dev/null
     ```

   - Check if Dockerfile* exists or Docker in plan.md → create/verify .dockerignore
   - Check if .eslintrc* exists → create/verify .eslintignore
   - Check if eslint.config.* exists → ensure the config's `ignores` entries cover required patterns
   - Check if .prettierrc* exists → create/verify .prettierignore
   - Check if pyproject.toml exists → verify Python patterns in .gitignore
   - Check if databricks.yml exists → verify Databricks patterns in .gitignore

   **If ignore file already exists**: Verify it contains essential patterns, append missing critical patterns only
   **If ignore file missing**: Create with full pattern set for detected technology

   **Databricks/Python Patterns** (from plan.md tech stack):
   - **Python/PySpark**: `__pycache__/`, `*.pyc`, `.venv/`, `venv/`, `.uv/`, `dist/`, `*.egg-info/`, `.python-version`
   - **Databricks**: `.databricks/`, `*.whl`, `.bundle/`, `target/`
   - **Spark**: `spark-warehouse/`, `metastore_db/`, `derby.log`
   - **Testing**: `.pytest_cache/`, `.coverage`, `htmlcov/`, `.hypothesis/`
   - **IDE/Editor**: `.vscode/`, `.idea/`, `*.swp`, `.DS_Store`
   - **Environment**: `.env`, `.env.*`, `!.env.example`, `*.local`
   - **Logs/Temp**: `*.log`, `logs/`, `tmp/`, `temp/`

   **Databricks-Specific Setup**:
   - Verify `pyproject.toml` exists with correct dependencies (pyspark, databricks-connect, pytest)
   - Verify `databricks.yml` exists for Asset Bundle configuration
   - Check `.databrickscfg` or environment variables for authentication
   - Verify Unity Catalog namespace configuration in plan.md

5. Parse tasks.md structure and extract:
   - **Task phases**: Setup, Bronze Layer, Silver Layer, Gold Layer, Testing, Integration, Deployment
   - **Task dependencies**: Sequential vs parallel execution rules
   - **Task details**: ID, description, file paths, parallel markers [P]
   - **Execution flow**: Order and dependency requirements (Bronze before Silver before Gold)

6. Execute implementation following the task plan:
   - **Phase-by-phase execution**: Complete each phase before moving to the next
   - **Respect medallion dependencies**: Bronze layer tasks complete before Silver, Silver before Gold
   - **Respect dependencies**: Run sequential tasks in order, parallel tasks [P] can run together
   - **Follow TDD approach**: Execute test tasks before their corresponding implementation tasks
   - **File-based coordination**: Tasks affecting the same notebooks/modules must run sequentially
   - **Validation checkpoints**: Verify each phase completion before proceeding

7. Implementation execution rules:
   - **Setup first**: Initialize project structure, `uv` dependencies, Databricks configuration
   - **Tests before code**: Write tests for transformations, data quality validators, and integration scenarios
   - **Bronze layer**: Implement ingestion notebooks, raw data landing, source schema preservation
   - **Silver layer**: Implement cleansing, deduplication, standardization, and business keys
   - **Gold layer**: Implement aggregations, business metrics, and consumption-ready tables
   - **Integration work**: DLT pipeline definitions, job workflows, cross-notebook dependencies
   - **Polish and validation**: Unit tests, integration tests, performance optimization, documentation

8. Progress tracking and error handling:
   - Report progress after each completed task
   - Halt execution if any non-parallel task fails
   - For parallel tasks [P], continue with successful tasks, report failed ones
   - Provide clear error messages with context for debugging
   - Suggest next steps if implementation cannot proceed
   - **IMPORTANT** For completed tasks, make sure to mark the task off as [X] in the tasks file.

9. Completion validation:
   - Verify all required tasks are completed
   - Check that implemented pipeline matches the original specification
   - Validate that tests pass: `uv run pytest`
   - Verify data lineage is complete (Bronze → Silver → Gold)
   - Confirm DLT expectations and data quality checks are in place
   - Validate Asset Bundle configuration is deployable: `databricks bundle validate`
   - Report final status with summary of completed work

Note: This command assumes a complete task breakdown exists in tasks.md. If tasks are incomplete or missing, suggest running `/speckit.tasks` first to regenerate the task list.
