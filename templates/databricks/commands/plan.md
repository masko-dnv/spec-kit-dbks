---
description: Create a comprehensive implementation plan for a Databricks data pipeline with project structure, phases, and technical details.
handoffs: 
  - label: Build Task List
    agent: speckit.tasks
    prompt: Create a detailed task list and phases for this implementation plan. The pipeline is...
    send: true
  - label: Clarify Architecture
    agent: speckit.clarify
    prompt: Help me clarify the data pipeline architecture and design decisions
    send: true
scripts:
  sh: scripts/bash/create-new-feature.sh --json "{ARGS}"
  ps: scripts/powershell/create-new-feature.ps1 -Json "{ARGS}"
---

## User Input

```text
$ARGUMENTS
```

You **MUST** consider the user input before proceeding (if not empty).

## Overview

You are creating a **detailed implementation plan** for a Databricks data pipeline. This plan translates the data specification into concrete technical architecture, project structure, technology choices, and phase-by-phase implementation strategy.

---

## Your Task

Create a comprehensive implementation plan that includes:

### 1. **Technical Context**
- Python version and Databricks Runtime (DBR) requirements
- Compute type (Job Clusters, All-Purpose, Serverless)
- Storage backend (Delta Lake on Unity Catalog)
- Testing framework (pytest with databricks-connect)
- Dependency management (uv package manager)
- Multi-environment targets (dev, staging, prod)

### 2. **Project Structure**
Organize the project following best practices:

```
[project-name]/
├── .github/                    # CI/CD workflows
├── .vscode/                    # VS Code settings
├── docs/                       # Documentation
├── fixtures/sample_data/       # Test data
├── notebooks/                  # Databricks notebooks
│   ├── _setup/                 # Schema setup
│   ├── ingest_*.py
│   ├── transform_*.py
│   └── output_*.py
├── resources/                  # Databricks Asset Bundle configs
├── scratch/                    # Exploratory notebooks
├── src/                        # Reusable Python modules
├── tests/
│   ├── unit/
│   └── integration/
├── databricks.yml
├── pyproject.toml
└── README.md
```

- Explain the purpose of each directory
- Describe how code is organized (src-layout for testability)
- Explain separation of concerns (notebooks for orchestration, src/ for logic)

### 3. **Key Components**

#### Databricks Asset Bundle Configuration (`databricks.yml`)
- Define job definitions with proper task dependencies
- Configure cluster specs for each phase (ingest, transform, output)
- Set up multi-target deployment (dev/staging/prod)
- Include scheduling and timeout settings
- Document parameterization for different environments

#### Python Dependencies (`pyproject.toml`)
- Production dependencies (minimal, deployed to clusters)
- Development dependencies (databricks-connect, pytest, black, ruff)
- Build system configuration (hatchling)
- Test and lint configurations
- Databricks builtins for Ruff configuration

#### Notebook Structure
- Provide example code for ingestion, transformation, and output notebooks
- Show how to use Databricks Connect for local development
- Demonstrate data quality validation patterns
- Include error handling and logging

#### Testing Strategy
- Unit tests (no Databricks connection required)
- Integration tests (with databricks-connect)
- Fixtures for test data
- Coverage expectations (> 80%)

### 4. **Implementation Phases**

Break down implementation into clear phases with deliverables:

- **Phase 1**: Bundle Setup & Environment (local dev environment, Databricks connection)
- **Phase 2**: Schema & Infrastructure (Bronze, Silver, Gold layers, DDL)
- **Phase 3**: Data Ingestion (ingest notebooks, unit tests, jobs)
- **Phase 4**: Transformations (transform notebooks, business logic, validators)
- **Phase 5**: Output & Aggregation (aggregations, metrics, final tables)
- **Phase 6**: Orchestration & CI/CD (job workflows, GitHub Actions)
- **Phase 7**: Documentation & Polish (code docs, runbooks, sign-off)

For each phase:
- List specific tasks and deliverables
- Estimate effort (days/weeks)
- Identify dependencies on previous phases
- Define success criteria

### 5. **Success Metrics**

Define how you'll measure the pipeline's success:
- Data freshness: Time from source update to Gold table availability
- Data quality: Validation pass rate, error detection
- Reliability: Job success rate, SLA compliance
- Performance: Query speed, processing time
- Testing: Code coverage, test pass rate
- Documentation: 100% code coverage

### 6. **Known Constraints & Assumptions**

List any limitations or assumptions:
- Databricks Runtime version and compatibility
- Unity Catalog prerequisites
- Cluster resource availability
- Source system stability
- Acceptable latency thresholds

---

## Output Format

Structure your plan using this template:

```markdown
# Implementation Plan: [PIPELINE_NAME]

## Technical Context
[DBR version, Python, compute, storage, testing, dependencies]

## Project Structure
[Directory layout with explanations]

## Detailed Component Design
### 1. Databricks Asset Bundle Configuration
[databricks.yml structure and configuration]

### 2. Python Dependencies
[pyproject.toml with prod and dev deps]

### 3. Notebook Structure
[Code examples for ingest, transform, output]

### 4. Testing Strategy
[Unit and integration test patterns]

## Implementation Phases
[Phase 1 through Phase 7 with tasks and deliverables]

## Success Metrics
[Measurable criteria for success]

## Next Steps
[Immediate action items]

## References
[Links to documentation]
```

---

## Key Considerations

### Environment Management
- Document how to set up `.databrickscfg` with workspace credentials
- Explain how databricks-connect enables local development
- Show how environments are parameterized in `databricks.yml`

### Code Organization
- Place business logic in `src/` for testability
- Use notebooks for orchestration and Spark session management
- Separate concerns: transformations, validators, utilities

### Testing Approach
- Unit tests don't require Databricks connection
- Integration tests use databricks-connect and real data
- Fixture data stored in `fixtures/sample_data/`
- Test fixtures use pytest markers for organization

### Deployment Strategy
- Document multi-target support (dev/staging/prod)
- Explain how to validate bundles before deployment
- Show deployment commands and verification steps
- Include rollback procedures

### Performance & Cost Optimization
- Suggest cluster sizing based on data volumes
- Recommend partitioning and caching strategies
- Discuss Z-ordering for columnar optimization
- Mention cost monitoring and optimization opportunities

---

## Example Section: Notebook Structure

Here's what a well-structured ingest notebook looks like:

```python
# Databricks notebook source
from pyspark.sql.functions import current_timestamp, lit

# Read from source
df = spark.read.format("jdbc").load(...)

# Add metadata
df = df.withColumn("ingested_at", current_timestamp()) \
       .withColumn("source_system", lit("[SOURCE_NAME]"))

# Validate quality
assert df.count() > 0, "No data from source"

# Write to Bronze
df.write.format("delta").mode("append").saveAsTable("[catalog].[schema].raw_[entity]")
```

---

## Tips for Success

- **Be specific about resources**: Include exact cluster node types and worker counts
- **Document environment differences**: Show how parameters change per target
- **Provide code examples**: Help teams understand the patterns to follow
- **Define clear phases**: Make implementation manageable with clear milestones
- **Plan for testing**: Include testing strategy from the beginning
- **Consider operations**: Plan for monitoring, alerting, and troubleshooting

---

## Next Step

After completing the implementation plan, use the "Build Task List" handoff to create a detailed checklist of all tasks organized by phase.
