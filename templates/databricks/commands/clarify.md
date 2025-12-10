---
description: Clarify ambiguous requirements and resolve questions about your Databricks data pipeline.
handoffs: []
scripts:
  sh: scripts/bash/create-new-feature.sh --json "{ARGS}"
  ps: scripts/powershell/create-new-feature.ps1 -Json "{ARGS}"
---

## User Input

```text
$ARGUMENTS
```

## Overview

You are helping clarify and resolve ambiguities in a **Databricks data pipeline specification**. Use a structured questioning approach to:

- Clarify what data sources are actually available
- Understand business requirements and constraints
- Identify data quality expectations
- Determine performance and freshness SLAs
- Understand dependencies and integrations
- Resolve technical questions about architecture

---

## Clarification Framework

Ask targeted questions in these areas:

### 1. **Data Sources & Integration**

Help clarify source systems:

```markdown
## Data Source Questions

- What systems will feed data into this pipeline?
  - Database type, API, file format, event stream?
  - Connection authentication (basic, OAuth, service principal)?
  - Data refresh frequency (real-time, daily, weekly)?
  - Expected data volumes (rows/day, GB/day)?

- Which specific tables or API endpoints are needed?
  - All columns or a subset?
  - Any filtering on the source side?
  - Historical backfill needed (how many years)?

- What are the data quality expectations from the source?
  - Known data issues to handle?
  - Missing or null value rates?
  - Duplicates or data inconsistencies?
```

### 2. **Transformation & Business Logic**

Clarify what transformations are needed:

```markdown
## Transformation Questions

- What business logic must be applied?
  - Specific calculation formulas?
  - Reference lookups or enrichments?
  - Segmentation or categorization rules?

- How should you handle data quality issues?
  - Silently filter bad records?
  - Create a quarantine table for investigation?
  - Alert on data quality failures?

- What deduplication logic is needed?
  - Primary key for uniqueness?
  - When duplicates exist, keep first/last/highest value?
  - Document duplicate rates for monitoring?

- Are there dependent transformations?
  - Must Entity A be completed before Entity B?
  - Are there circular dependencies?
  - Can transformations run in parallel?
```

### 3. **Output & Consumption**

Understand who uses the data and how:

```markdown
## Output & Consumption Questions

- Who are the downstream consumers?
  - Analytics teams, ML models, BI dashboards, other pipelines?
  - What's the query pattern (real-time lookups, batch reporting)?
  - How frequently do they access the data?

- What business metrics are most critical?
  - Revenue, customer count, inventory levels?
  - Segment-level aggregations or global metrics?
  - Time granularity (hourly, daily, monthly)?

- What data freshness is required?
  - Must data be fresh within X hours?
  - Are there different SLAs for different metrics?
  - What's acceptable latency?

- How many rows/columns in output tables?
  - Growth projections (will table size increase over time)?
  - Impact on query performance?
```

### 4. **Performance & Infrastructure**

Clarify non-functional requirements:

```markdown
## Performance & Infrastructure Questions

- What's the acceptable pipeline execution time?
  - Should ingest complete within 1 hour?
  - Total pipeline (ingest + transform + output) < 2 hours?

- Data volume projections?
  - Current vs. expected in 6-12 months?
  - Peak loads vs. average?
  - Storage retention policy (90 days, 1 year)?

- Cluster sizing preferences?
  - Cost optimization important?
  - Performance requirements override cost?
  - Serverless vs. on-demand vs. spot instances?

- Monitoring & alerting expectations?
  - What constitutes a failure requiring alert?
  - Who should be notified (ops, data team, business)?
  - Escalation procedures?
```

### 5. **Testing & Quality Assurance**

Define quality standards:

```markdown
## Testing & QA Questions

- Data quality thresholds?
  - Acceptable null rate (e.g., < 0.1%)?
  - Duplicate detection and handling?
  - Validation rules for each critical column?

- Testing scope?
  - Unit tests for business logic?
  - Integration tests with real data?
  - Performance testing under production load?
  - Disaster recovery testing?

- Code coverage expectations?
  - Target coverage % (typically 70-80%)?
  - Critical paths that must be tested?

- Regression testing?
  - How to detect when pipeline breaks?
  - Automated alerts on data anomalies?
  - Historical data validation?
```

### 6. **Deployment & Operations**

Clarify deployment and operational concerns:

```markdown
## Deployment & Operations Questions

- Multi-environment strategy?
  - Dev, staging, prod environments?
  - How similar should staging be to prod?
  - Promotion workflow (dev → staging → prod)?

- Disaster recovery?
  - How quickly must pipeline recover from failure?
  - Backup/restore strategy for data?
  - Rollback procedures?

- Documentation needs?
  - Runbook for operations team?
  - Troubleshooting guide for common issues?
  - Glossary of business terms and metrics?

- Change management?
  - How to handle schema changes?
  - Breaking vs. non-breaking changes?
  - Communication protocol for production changes?
```

### 7. **Budget & Constraints**

Understand limitations:

```markdown
## Budget & Constraint Questions

- Cost constraints?
  - Monthly spend limit?
  - Cost optimization important?
  - Can we use serverless compute?

- Timeline constraints?
  - MVP timeline (when is v1 needed)?
  - Available resources/team size?
  - Are there dependencies that block start?

- Technical constraints?
  - Data residency requirements (data location)?
  - Compliance or regulatory requirements?
  - Integration constraints with existing systems?

- Staffing & expertise?
  - Team experience with Databricks?
  - Training needs?
  - Support model (dedicated team vs. shared)?
```

---

## Question Taxonomy for Data Pipelines

### Source Data
- **Availability**: Where is it? How to access?
- **Freshness**: How often is it updated?
- **Quality**: What are known issues?
- **Volume**: How much data, and growth rate?
- **History**: How far back do you need?

### Transformation Logic
- **Rules**: What business logic applies?
- **Deduplication**: How to identify unique records?
- **Enrichment**: External data to join?
- **Validation**: What makes data valid?
- **Dependencies**: Order of transformations?

### Output Requirements
- **Consumers**: Who uses the data?
- **Metrics**: What key metrics matter?
- **Freshness**: How current must data be?
- **Format**: Tables, APIs, files?
- **Access**: Who can see what data?

### Quality Standards
- **Accuracy**: Match source data?
- **Completeness**: All expected records present?
- **Consistency**: Data uniform across tables?
- **Timeliness**: Available when needed?
- **Validity**: Data within acceptable ranges?

---

## How to Proceed

Based on clarifications, help the user:

1. **Update the specification** with new clarity
2. **Identify gaps** in existing requirements
3. **Surface risks** or constraints that could impact timeline/cost
4. **Propose solutions** for ambiguous or conflicting requirements
5. **Recommend next steps** (talk to stakeholders, gather more info)

---

## Example Clarification

**User says**: "I need to build a customer analytics pipeline"

**You ask**:
- What specific customer data sources? (CRM, web events, transactions?)
- What metrics matter most? (CAC, LTV, retention, churn?)
- Who needs access? (Analytics team, finance, marketing?)
- How fresh must data be? (Real-time dashboards or daily reports?)
- What volume? (1M, 10M, 100M customers?)
- Current pain points? (Manual exports, slow queries, data inconsistencies?)

**Outcome**: Clear specification with specific requirements instead of vague goal

---

## Tips for Effective Clarification

- Ask **why** not just **what**
- Listen for **assumptions** and test them
- Look for **conflicting requirements** (e.g., low-cost but high-performance)
- Identify **stakeholders** who have different needs
- Be **specific** (not "improve data" but "reduce query time from 5min to 30sec")
- Document **trade-offs** (cost vs. performance, speed vs. accuracy)
