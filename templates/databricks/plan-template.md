# Implementation Plan: [PIPELINE_NAME]

**Branch**: `[###-pipeline-name]` | **Date**: [DATE] | **Spec**: [link]
**Input**: Data specification from `/specs/[###-pipeline-name]/spec.md`

**Note**: This template is filled in by the `/speckit.plan` command. See `.specify/templates/commands/plan.md` for the execution workflow.

## Summary

[Extract from data spec: primary data pipeline requirement + technical approach from research]

## Technical Context

<!--
  ACTION REQUIRED: Replace the content in this section with the technical details
  for the pipeline. The structure here is presented in advisory capacity to guide
  the iteration process.
-->

**Databricks Runtime**: [e.g., 14.3 LTS, 15.4 LTS or NEEDS CLARIFICATION]
**Python Version**: [e.g., 3.11, 3.12 - must match DBR or NEEDS CLARIFICATION]
**Compute Type**: [Job Clusters, All-Purpose, Serverless or NEEDS CLARIFICATION]
**Storage Backend**: [Delta Lake on Unity Catalog or NEEDS CLARIFICATION]
**Unity Catalog**: [catalog.schema pattern, e.g., dev_catalog.bronze or NEEDS CLARIFICATION]
**Testing**: [pytest with databricks-connect or NEEDS CLARIFICATION]
**Dependency Manager**: [uv or NEEDS CLARIFICATION]
**Bundle Targets**: [dev, staging, prod or NEEDS CLARIFICATION]
**Data Freshness SLA**: [e.g., < 24 hours, near real-time or NEEDS CLARIFICATION]
**Expected Data Volume**: [e.g., 10GB/day, 1M rows/hour or NEEDS CLARIFICATION]
**Source Systems**: [e.g., JDBC, REST API, S3, ADLS or NEEDS CLARIFICATION]

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

[Gates determined based on constitution file]

## Project Structure

### Documentation (this pipeline)

```text
specs/[###-pipeline]/
├── plan.md              # This file (/speckit.plan command output)
├── research.md          # Phase 0 output (/speckit.plan command)
├── data-model.md        # Phase 1 output (/speckit.plan command)
├── quickstart.md        # Phase 1 output (/speckit.plan command)
├── contracts/           # Phase 1 output (/speckit.plan command)
└── tasks.md             # Phase 2 output (/speckit.tasks command - NOT created by /speckit.plan)
```

### Source Code (repository root)
<!--
  ACTION REQUIRED: Replace the placeholder tree below with the concrete layout
  for this pipeline. Delete unused options and expand the chosen structure with
  real paths. The delivered plan must not include Option labels.
-->

```text
# [REMOVE IF UNUSED] Option 1: Standard Databricks Pipeline (DEFAULT)
notebooks/
├── _setup/                 # Schema setup DDL
│   └── create_tables.py
├── ingest_[source].py      # Bronze layer ingestion
├── transform_[entity].py   # Silver layer transformations
└── output_[table].py       # Gold layer aggregations

src/
└── [package_name]/
    ├── transformations/    # PySpark transformation logic
    ├── validators/         # Data quality checks
    └── utils/              # Helper functions

tests/
├── unit/                   # No Databricks connection required
└── integration/            # With databricks-connect

fixtures/
└── sample_data/            # Test data (CSV, JSON, Delta)

# [REMOVE IF UNUSED] Option 2: Multi-Pipeline Project (when multiple independent pipelines)
notebooks/
├── pipeline_a/
│   ├── ingest_*.py
│   ├── transform_*.py
│   └── output_*.py
└── pipeline_b/
    ├── ingest_*.py
    ├── transform_*.py
    └── output_*.py

src/
├── shared/                 # Common utilities across pipelines
└── [pipeline_name]/        # Pipeline-specific logic

# [REMOVE IF UNUSED] Option 3: Streaming Pipeline (when real-time processing required)
notebooks/
├── _setup/
├── stream_ingest_[source].py   # Structured Streaming ingestion
├── stream_transform_[entity].py # Streaming transformations
└── batch_output_[table].py     # Periodic batch aggregations

src/
└── [package_name]/
    ├── streaming/          # Streaming-specific logic
    ├── transformations/
    └── validators/
```

**Structure Decision**: [Document the selected structure and reference the real
directories captured above]

## Complexity Tracking

> **Fill ONLY if Constitution Check has violations that must be justified**

| Violation | Why Needed | Simpler Alternative Rejected Because |
|-----------|------------|-------------------------------------|
| [e.g., 4th data layer] | [current need] | [why Bronze/Silver/Gold insufficient] |
| [e.g., Custom streaming] | [specific problem] | [why batch processing insufficient] |
