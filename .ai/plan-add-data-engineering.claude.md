# Plan: Add Databricks Data Engineering Flavor to Spec Kit

**Status**: Ready for Implementation
**Goal**: Add Databricks data engineering as a new project flavor alongside existing web app templates

## Summary

Add a `--databricks` flag to Spec Kit that bootstraps projects for Databricks data engineering workflows. This creates a parallel set of templates in `templates/databricks/` focused on:

- **Databricks Connect** for local Python/PySpark development in VS Code
- **Databricks Asset Bundles** for deployment and CI/CD
- **Unity Catalog** for data governance
- **Notebooks** for transformations, **Jobs** for orchestration
- **Delta tables** as output targets
- **pytest** with Databricks Connect for testing

**Not in scope**: Delta Live Tables (DLT) - standard notebooks + jobs only

---

## Architecture: Profile-Based Templates

Create a `templates/databricks/` directory that mirrors the main template structure. When `--databricks` flag is used, these templates override the defaults.

```text
templates/
├── spec-template.md          # Existing (web apps)
├── plan-template.md          # Existing (web apps)
├── tasks-template.md         # Existing (web apps)
├── commands/                 # Existing (web apps)
│   ├── specify.md
│   ├── plan.md
│   └── ...
└── databricks/               # NEW: Databricks flavor
    ├── spec-template.md      # Data pipeline specs
    ├── plan-template.md      # DABs project structure
    ├── tasks-template.md     # Pipeline phases
    ├── data-model-template.md # Schema documentation
    ├── databricks-yml-template.yml
    ├── pyproject-template.toml # uv dependency management
    ├── vscode-settings.json  # Databricks VS Code config
    └── commands/
        ├── specify.md
        ├── plan.md
        ├── tasks.md
        ├── implement.md
        ├── checklist.md
        ├── clarify.md
        └── analyze.md
```

---

## Files to Create

### 1. New Databricks Templates

| File | Purpose |
|------|---------|
| `templates/databricks/spec-template.md` | Data pipeline specification with stages, sources, DQ rules |
| `templates/databricks/plan-template.md` | Databricks Asset Bundle project structure |
| `templates/databricks/tasks-template.md` | Pipeline phases (Setup → Schema → Ingest → Transform → Output) |
| `templates/databricks/data-model-template.md` | Delta table schema documentation |
| `templates/databricks/databricks-yml-template.yml` | Asset Bundle configuration scaffold |
| `templates/databricks/pyproject-template.toml` | uv dependency management with required Databricks packages |
| `templates/databricks/vscode-settings.json` | VS Code settings for Databricks Connect |

### 2. New Databricks Commands

| File | Purpose |
|------|---------|
| `templates/databricks/commands/specify.md` | Data-centric specification prompts |
| `templates/databricks/commands/plan.md` | Schema & DABs architecture planning |
| `templates/databricks/commands/tasks.md` | Pipeline task generation |
| `templates/databricks/commands/implement.md` | Databricks bundle deploy/run patterns |
| `templates/databricks/commands/checklist.md` | Data quality checklist examples |
| `templates/databricks/commands/clarify.md` | Data engineering clarification taxonomy |
| `templates/databricks/commands/analyze.md` | Pipeline consistency analysis |

### 3. CLI Modifications

| File | Changes |
|------|---------|
| `src/specify_cli/__init__.py` | Add `--databricks` flag, update download logic |

### 4. Build/Release Updates

| File | Changes |
|------|---------|
| `.github/workflows/scripts/create-release-packages.ps1` | **Primary** - Generate `spec-kit-template-databricks-{agent}-{script}.zip` packages |
| `.github/workflows/scripts/create-release-packages.sh` | Secondary - Mirror PowerShell for cross-platform support |

---

## Detailed Template Content

### spec-template.md (Data Pipeline Specification)

**Key Sections**:

- **Pipeline Overview**: Name, description, schedule/trigger
- **Data Sources**: Source systems, connection types, schemas
- **Pipeline Stages**: Ingest → Transform → Output (replaces User Stories)
- **Data Quality Requirements**: Validation rules, thresholds
- **Acceptance Scenarios**: Data volume, freshness, quality metrics

**Stage Format** (replaces User Story format):

```markdown
### Stage 1 - Ingest [Source Name] (Priority: P1)

**Source**: [connection details]
**Target**: [Unity Catalog location]
**Load Pattern**: [Append/Overwrite/Merge]

**Data Quality Checks**:
- DQ-001: [column] MUST [validation rule]

**Acceptance Scenarios**:
1. Given [source state], When [notebook runs], Then [expected output]
```

### plan-template.md (Databricks Implementation Plan)

**Technical Context** (Databricks-specific):

```markdown
**Python Version**: 3.12+ (Databricks Runtime compatible)
**Databricks Runtime**: [e.g., 17.3 LTS]
**Compute**: [Job Clusters / All-Purpose / Serverless]
**Storage**: Delta Lake on Unity Catalog
**Catalog**: [catalog_name]
**Schemas**: [bronze/silver/gold schema names]
**Testing**: pytest with databricks-connect
**Bundle Targets**: dev / staging / prod
```

**Project Structure** (Databricks Asset Bundle):

```text
[project-name]/
├── .github/                    # GitHub configs & CI workflows
├── .vscode/                    # VS Code workspace settings
├── docs/                       # Documentation
├── fixtures/
│   └── sample_data/            # Test data files
├── notebooks/                  # Databricks notebooks for orchestration
│   └── _setup/                 # Table setup & teardown notebooks
├── resources/                  # Databricks Job & Pipeline YAML configs
├── scratch/                    # Exploratory notebooks (not deployed)
├── src/                        # Python source code (no notebooks)
├── tests/
│   ├── unit/                   # Unit tests
│   └── integration/            # Integration tests (uses fixtures/sample_data)
├── databricks.yml              # Databricks Asset Bundle config
├── pyproject.toml              # Python project & uv dependency config
└── README.md
```

### tasks-template.md (Pipeline Task Phases)

**Phase Structure**:

- **Phase 1: Bundle Setup** - `databricks bundle init`, environment config, VS Code setup
- **Phase 2: Schema & Infrastructure** - DDL for Delta tables, utility functions
- **Phase 3+: Pipeline Stages** - One phase per pipeline (Ingest → Transform → Output → Job Definition)
- **Final Phase: Observability & Polish** - Monitoring, documentation, validation

**Task Format**:

```markdown
- [ ] T001 [P1] Create setup notebook in notebooks/_setup/create_tables.py
- [ ] T002 [P1] Create processing notebook in notebooks/process_[entity].py
- [ ] T003 [P1] Create transformation module in src/transformations/[entity].py
- [ ] T004 [P1] Define workflow job in resources/[pipeline].job.yml
- [ ] T005 [P1] Add unit tests in tests/unit/test_[entity].py
- [ ] T006 [P1] Add integration tests in tests/integration/test_[pipeline].py
```

### pyproject.toml (uv Dependency Management)

All Databricks projects use **uv** for dependency management. A pre-configured `pyproject.toml` template is included with required dependencies:

```toml
[project]
name = "[PROJECT_NAME]"
version = "0.1.0"
description = "Databricks data engineering pipeline"
requires-python = ">=3.12"
dependencies = [
    "databricks-connect>=[VERSION]",
    "databricks-sdk>=[VERSION]",
    "pyspark>=[VERSION]",
    "delta-spark>=[VERSION]",
    "pytest>=[VERSION]",
    "pytest-cov>=[VERSION]",
]

[project.optional-dependencies]
dev = [
    "black>=[VERSION]",
    "ruff>=[VERSION]",
    "mypy>=[VERSION]",
]

[tool.pytest.ini_options]
testpaths = ["tests"]
python_files = ["test_*.py"]
python_functions = ["test_*"]

[tool.black]
line-length = 88
target-version = ["py312"]

[tool.ruff]
line-length = 88
target-version = "py312"
```

### New Template File

| File | Purpose |
|------|---------|
| `templates/databricks/pyproject-template.toml` | uv dependency management with required Databricks packages |

### vscode-settings.json (Databricks VS Code Config)

```json
{
  "python.defaultInterpreterPath": "${workspaceFolder}/.venv/bin/python",
  "databricks.overrideDatabricksConfigFile": "${workspaceFolder}/.databrickscfg",
  "python.analysis.extraPaths": ["${workspaceFolder}/src/python"],
  "files.associations": {
    "*.yml": "yaml"
  },
  "yaml.schemas": {
    "https://raw.githubusercontent.com/databricks/bundle-examples/main/schema/bundle_config_schema.json": "databricks.yml"
  },
  "[python]": {
    "editor.defaultFormatter": "ms-python.black-formatter",
    "editor.formatOnSave": true
  }
}
```

### databricks-yml-template.yml (Asset Bundle Scaffold)

```yaml
bundle:
  name: [PROJECT_NAME]

variables:
  catalog:
    description: Unity Catalog name
    default: dev_catalog

workspace:
  host: ${var.workspace_url}

resources:
  jobs:
    [pipeline_name]_job:
      name: "[PIPELINE_NAME] Pipeline"
      tasks:
        - task_key: ingest
          notebook_task:
            notebook_path: ./src/notebooks/ingest_[source].py
          job_cluster_key: pipeline_cluster
        - task_key: transform
          depends_on:
            - task_key: ingest
          notebook_task:
            notebook_path: ./src/notebooks/transform_[entity].py
          job_cluster_key: pipeline_cluster
      job_clusters:
        - job_cluster_key: pipeline_cluster
          new_cluster:
            spark_version: 14.3.x-scala2.12
            node_type_id: Standard_DS3_v2
            num_workers: 1

targets:
  dev:
    default: true
    variables:
      workspace_url: https://[dev-workspace].azuredatabricks.net
  prod:
    variables:
      workspace_url: https://[prod-workspace].azuredatabricks.net
```

---

## CLI Implementation

### Add `--mode` Flag

In `src/specify_cli/__init__.py`:

```python
from enum import Enum

class ProjectMode(str, Enum):
    DEFAULT = "default"
    DATABRICKS = "databricks"

def init(
    project_name: str,
    # ... existing args ...
    mode: ProjectMode = typer.Option(
        ProjectMode.DEFAULT,
        "--mode",
        "-m",
        help="Project type: 'default' (web apps) or 'databricks' (data engineering)"
    ),
):
```

### Usage Examples

```bash
# Default mode (web apps) - all equivalent:
specify init my-web-app --ai claude
specify init my-web-app --ai claude --mode default
specify init my-web-app --ai claude -m default

# Databricks mode (data engineering):
specify init my-pipeline --ai claude --mode databricks
specify init my-pipeline --ai claude -m databricks
```

### Update Download Logic

Modify asset name construction based on mode:

- **Default mode**: `spec-kit-template-{agent}-{script}.zip`
- **Databricks mode**: `spec-kit-template-databricks-{agent}-{script}.zip`

---

## Build & Release Process

### Update Release Scripts

**Priority: PowerShell scripts** - Create PowerShell versions first, then bash equivalents.

#### `create-release-packages.ps1` (Primary)

1. Keep existing logic for standard templates
2. Add loop for Databricks templates:
   - Copy base `.specify/` structure
   - Overlay with `templates/databricks/*`
   - Generate agent-specific commands
   - Create `spec-kit-template-databricks-{agent}-{script}.zip`

#### `create-release-packages.sh` (Secondary)

Mirror the PowerShell implementation for cross-platform CI/CD support.

---

## Databricks Patterns to Embed

### Databricks Connect Session

```python
from databricks.connect import DatabricksSession
spark = DatabricksSession.builder.getOrCreate()
```

### pytest Fixture

```python
@pytest.fixture(scope="session")
def spark():
    return DatabricksSession.builder.getOrCreate()
```

### Bundle Commands

```bash
databricks bundle validate        # Check configuration
databricks bundle deploy -t dev   # Deploy to workspace
databricks bundle run -t dev job  # Execute job
databricks bundle destroy         # Clean up
```

### Typical Pipeline Pattern

A notebook that:

1. Reads data from source(s) using Spark
2. Applies series of transformations
3. Writes output to Delta table in Unity Catalog

Orchestrated by a Job that chains notebooks with dependencies.

---

## Implementation Sequence

1. **Phase 1**: Create `templates/databricks/` directory structure
2. **Phase 2**: Create core templates (spec, plan, tasks)
3. **Phase 3**: Create command templates (specify, plan, tasks, implement, etc.)
4. **Phase 4**: Create supporting files (vscode-settings, databricks-yml-template, pyproject-template)
5. **Phase 5**: Update CLI (`src/specify_cli/__init__.py`)
6. **Phase 6**: Update build scripts (**PowerShell first**, then bash)
   - Create `create-release-packages.ps1` (primary)
   - Update `create-release-packages.sh` (secondary, mirror PowerShell)

---

## Verification Plan

1. **Build Test**: Run `create-release-packages.sh` locally, verify both default and databricks ZIP files created
2. **CLI Test**:
   - `specify init my-web-app --ai claude` → Default (web app) structure
   - `specify init my-web-app --ai claude --mode default` → Default (web app) structure
   - `specify init my-pipeline --ai claude --mode databricks` → Databricks structure
   - `specify init my-pipeline --ai claude -m databricks` → Databricks structure
