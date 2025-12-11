---
description: Perform a non-destructive cross-artifact consistency and quality analysis across spec.md, plan.md, and tasks.md for Databricks data pipelines.
scripts:
  sh: scripts/bash/check-prerequisites.sh --json --require-tasks --include-tasks
  ps: scripts/powershell/check-prerequisites.ps1 -Json -RequireTasks -IncludeTasks
---

## User Input

```text
$ARGUMENTS
```

You **MUST** consider the user input before proceeding (if not empty).

## Goal

Identify inconsistencies, duplications, ambiguities, and underspecified items across the three core artifacts (`spec.md`, `plan.md`, `tasks.md`) before implementation. This command MUST run only after `/speckit.tasks` has successfully produced a complete `tasks.md`. Analysis focuses on Databricks data pipeline concerns: medallion architecture consistency, data lineage validation, schema coherence, and PySpark/DLT alignment.

## Operating Constraints

**STRICTLY READ-ONLY**: Do **not** modify any files. Output a structured analysis report. Offer an optional remediation plan (user must explicitly approve before any follow-up editing commands would be invoked manually).

**Constitution Authority**: The project constitution (`/memory/constitution.md`) is **non-negotiable** within this analysis scope. Constitution conflicts are automatically CRITICAL and require adjustment of the spec, plan, or tasks—not dilution, reinterpretation, or silent ignoring of the principle. If a principle itself needs to change, that must occur in a separate, explicit constitution update outside `/speckit.analyze`.

## Execution Steps

### 1. Initialize Analysis Context

Run `{SCRIPT}` once from repo root and parse JSON for FEATURE_DIR and AVAILABLE_DOCS. Derive absolute paths:

- SPEC = FEATURE_DIR/spec.md
- PLAN = FEATURE_DIR/plan.md
- TASKS = FEATURE_DIR/tasks.md

Abort with an error message if any required file is missing (instruct the user to run missing prerequisite command).
For single quotes in args like "I'm Groot", use escape syntax: e.g 'I'\''m Groot' (or double-quote if possible: "I'm Groot").

### 2. Load Artifacts (Progressive Disclosure)

Load only the minimal necessary context from each artifact:

**From spec.md:**

- Pipeline Overview (description, objectives, schedule)
- Data Sources (systems, schemas, Unity Catalog volumes)
- Pipeline Stages (Bronze/Silver/Gold layers)
- Data Quality Requirements (thresholds, validation rules)
- Non-Functional Requirements (latency SLAs, cost constraints)
- Acceptance Scenarios

**From plan.md:**

- Architecture/stack choices (DLT vs notebooks, streaming vs batch)
- Medallion layer definitions
- Compute configuration (cluster types, Databricks Connect settings)
- Phases and technical constraints
- `uv` dependency management approach

**From tasks.md:**

- Task IDs
- Descriptions
- Phase grouping
- Parallel markers [P]
- Referenced file paths (notebooks, modules, configs)

**From constitution:**

- Load `/memory/constitution.md` for principle validation

### 3. Build Semantic Models

Create internal representations (do not include raw artifacts in output):

- **Data source inventory**: Each source system with catalog/schema/table paths and ingestion method
- **Medallion layer mapping**: Bronze → Silver → Gold transformations with table lineage
- **Schema registry**: Column definitions, data types, and consistency across layers
- **Task coverage mapping**: Map each task to one or more requirements or pipeline stages (inference by keyword / explicit reference patterns like IDs or key phrases)
- **Constitution rule set**: Extract principle names and MUST/SHOULD normative statements

### 4. Detection Passes (Token-Efficient Analysis)

Focus on high-signal findings. Limit to 50 findings total; aggregate remainder in overflow summary.

#### A. Data Lineage Validation

- Every source table in spec appears in Bronze layer
- Every Bronze table has a corresponding Silver transformation
- Every Silver table contributes to Gold aggregations
- No tables created but never consumed downstream
- Join/merge operations documented with keys specified

#### B. Schema Consistency

- Bronze columns match source data schema
- Primary/foreign keys consistent across related tables
- Data types consistent (e.g., `customer_id` is always STRING or always LONG)
- Column naming conventions followed across layers
- Nullable constraints explicitly defined

#### C. Ambiguity Detection

- Flag vague adjectives (fast, scalable, real-time) lacking measurable criteria
- Flag unresolved placeholders (TODO, TKTK, ???, `<placeholder>`, etc.)
- Flag undefined thresholds for data quality checks

#### D. Underspecification

- Requirements with verbs but missing object or measurable outcome
- Pipeline stages missing acceptance criteria alignment
- Tasks referencing notebooks or modules not defined in spec/plan
- Missing partition strategies for large tables
- Missing error handling for source failures

#### E. Constitution Alignment

- Any requirement or plan element conflicting with a MUST principle
- Missing mandated sections or quality gates from constitution

#### F. Coverage Gaps

- Requirements with zero associated tasks
- Tasks with no mapped requirement/pipeline stage
- Non-functional requirements not reflected in tasks (e.g., latency SLAs, cost limits)
- Missing tests for transformation logic or data quality validators

#### G. Inconsistency

- Terminology drift (same concept named differently across files)
- Data entities referenced in plan but absent in spec (or vice versa)
- Task ordering contradictions (e.g., Gold layer tasks before Silver setup without dependency note)
- Conflicting requirements (e.g., one requires DLT while other specifies classic notebooks)
- Cluster/compute configuration mismatches between spec and plan

### 5. Severity Assignment

Use this heuristic to prioritize findings:

- **CRITICAL**: Violates constitution MUST, missing core spec artifact, broken data lineage (orphan tables), or requirement with zero coverage that blocks baseline functionality
- **HIGH**: Duplicate or conflicting requirement, schema inconsistency across layers, ambiguous SLA/performance attribute, untestable acceptance criterion
- **MEDIUM**: Terminology drift, missing non-functional task coverage, underspecified edge case, missing partition strategy
- **LOW**: Style/wording improvements, minor redundancy not affecting execution order, documentation gaps

### 6. Produce Compact Analysis Report

Output a Markdown report (no file writes) with the following structure:

## Pipeline Analysis Report

| ID | Category | Severity | Location(s) | Summary | Recommendation |
|----|----------|----------|-------------|---------|----------------|
| L1 | Lineage | CRITICAL | spec.md:L45, plan.md:L120 | Bronze table `raw_orders` has no Silver consumer | Add Silver transformation or remove from spec |
| S1 | Schema | HIGH | spec.md:L80, tasks.md:L55 | `customer_id` typed as STRING in spec, LONG in tasks | Align to single type |

(Add one row per finding; generate stable IDs prefixed by category initial: L=Lineage, S=Schema, A=Ambiguity, U=Underspec, C=Constitution, G=Gap, I=Inconsistency.)

**Medallion Coverage Table:**

| Layer | Table | Source/Upstream | Has Transformation? | Has Tests? | Notes |
|-------|-------|-----------------|---------------------|------------|-------|
| Bronze | raw_orders | source_system_a | Yes | No | Missing unit tests |

**Coverage Summary Table:**

| Requirement Key | Has Task? | Task IDs | Notes |
|-----------------|-----------|----------|-------|

**Constitution Alignment Issues:** (if any)

**Unmapped Tasks:** (if any)

**Metrics:**

- Total Data Sources
- Total Tables (Bronze/Silver/Gold)
- Total Requirements
- Total Tasks
- Coverage % (requirements with >=1 task)
- Lineage Completeness % (tables with upstream and downstream)
- Ambiguity Count
- Schema Issue Count
- Critical Issues Count

### 7. Provide Next Actions

At end of report, output a concise Next Actions block:

- If CRITICAL issues exist: Recommend resolving before `/speckit.implement`
- If only LOW/MEDIUM: User may proceed, but provide improvement suggestions
- Provide explicit command suggestions: e.g., "Run /speckit.specify to add missing Silver transformation", "Run /speckit.plan to define partition strategy", "Manually edit tasks.md to add coverage for 'data-quality-monitoring'"

### 8. Offer Remediation

Ask the user: "Would you like me to suggest concrete remediation edits for the top N issues?" (Do NOT apply them automatically.)

## Operating Principles

### Context Efficiency

- **Minimal high-signal tokens**: Focus on actionable findings, not exhaustive documentation
- **Progressive disclosure**: Load artifacts incrementally; don't dump all content into analysis
- **Token-efficient output**: Limit findings table to 50 rows; summarize overflow
- **Deterministic results**: Rerunning without changes should produce consistent IDs and counts

### Analysis Guidelines

- **NEVER modify files** (this is read-only analysis)
- **NEVER hallucinate missing sections** (if absent, report them accurately)
- **Prioritize constitution violations** (these are always CRITICAL)
- **Prioritize data lineage breaks** (orphan tables are CRITICAL for pipelines)
- **Use examples over exhaustive rules** (cite specific instances, not generic patterns)
- **Report zero issues gracefully** (emit success report with coverage statistics)

### Databricks-Specific Checks

- Validate Unity Catalog three-level namespace usage (catalog.schema.table)
- Verify DLT expectations align with spec data quality requirements
- Check compute configuration matches workload (streaming vs batch)
- Ensure `uv` dependencies in `pyproject.toml` support specified PySpark version
- Validate Databricks Connect configuration if local development specified

## Context

{ARGS}
