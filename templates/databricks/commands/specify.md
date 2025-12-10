---
description: Create or update a data pipeline specification document with sources, stages, and quality requirements.
handoffs: 
  - label: Build Implementation Plan
    agent: speckit.plan
    prompt: Create an implementation plan for this data pipeline. I am building with...
    send: true
  - label: Clarify Requirements
    agent: speckit.clarify
    prompt: Help me clarify the data pipeline requirements
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

You are helping create a **data pipeline specification document** for a Databricks data engineering project. This is different from a typical web app feature specification—it focuses on data sources, transformation stages, and data quality requirements rather than user stories.

---

## Your Task

Based on the user's description, create a comprehensive data pipeline specification that covers:

### 1. **Pipeline Overview**
- Clear description of what the pipeline does
- Business problem it solves
- Key objectives and success criteria
- Schedule and SLA requirements

### 2. **Data Sources**
- Identify all source systems (databases, APIs, files, etc.)
- Document source schemas and data volumes
- Specify refresh rates and freshness requirements
- List any special access or authentication needs

### 3. **Pipeline Stages**
- Break down the pipeline into logical stages:
  - **Ingest**: Loading raw data into Bronze layer
  - **Transform**: Cleaning, deduplication, business rules into Silver layer
  - **Aggregate**: Creating business metrics in Gold layer
- For each stage, describe:
  - Input and output tables (Bronze/Silver/Gold)
  - Transformations and business logic
  - Data quality checks and validation rules
  - Expected data volumes and performance characteristics

### 4. **Data Quality Requirements**
- Define thresholds for:
  - Null rates, duplicates, schema compliance
  - Freshness SLAs and completeness metrics
- Document validation rules and acceptance criteria
- Specify what actions to take if quality thresholds are breached

### 5. **Deployment Context**
- Identify target environments: dev, staging, prod
- Note any environment-specific catalog or schema names
- Document compute and storage requirements

### 6. **Dependencies & Integrations**
- List upstream dependencies (other pipelines, systems)
- Identify downstream consumers (dashboards, ML models, other pipelines)
- Document any critical integration points

---

## Output Format

Structure the specification document following this template:

```markdown
# Data Pipeline Specification: [PIPELINE_NAME]

## Pipeline Overview
[Description, objectives, schedule]

## Data Sources
[Source systems with schemas and volumes]

## Pipeline Stages
[Ingest → Transform → Aggregate stages]

## Data Quality Requirements
[Validation rules and thresholds]

## Deployment Environments
[Dev/Staging/Prod configurations]

## Dependencies & Integration Points
[Upstream and downstream connections]
```

---

## Example

If the user says: "I need to build a customer data pipeline that combines customer records from our CRM and web events, deduplicates them, calculates daily metrics for each customer segment, and powers our analytics dashboards. Data comes in daily, must be fresh within 24 hours, and we need to validate that no customer records are missing."

You would create a spec with:

- **Overview**: Daily customer data pipeline, deduplication, segment-level metrics, 24-hour freshness SLA
- **Sources**: CRM API (customer records), web events database (behavioral data)
- **Stages**: 
  - Ingest: Raw CRM → bronze.raw_customers, Raw events → bronze.raw_events
  - Transform: Deduplicate & clean → silver.customers, silver.events
  - Aggregate: Calculate metrics → gold.customer_daily_metrics
- **Quality**: No duplicate customer IDs, null rate < 0.1%, all required columns populated
- **Dependencies**: Dashboards consume gold tables

---

## Tips for Success

- **Be specific about sources**: Include connection types, authentication methods, expected row counts
- **Define each transformation**: Explain deduplication keys, business rules, filtering logic
- **Set realistic thresholds**: Use actual business requirements, not generic defaults
- **Think in layers**: Bronze (raw) → Silver (clean) → Gold (business-ready)
- **Consider testing**: Mention how you'll validate the pipeline works correctly
- **Plan for failure**: Document what happens if quality checks fail

---

## Next Step

After completing the specification, use the "Build Implementation Plan" handoff to create a detailed technical plan for building this pipeline.
