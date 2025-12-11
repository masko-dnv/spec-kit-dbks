# Data Pipeline Specification: [PIPELINE_NAME]

**Feature Branch**: `[###-pipeline-name]`  
**Created**: [DATE]  
**Status**: Draft  
**Input**: User description: "$ARGUMENTS"

---

## Pipeline Overview *(mandatory)*

<!--
  ACTION REQUIRED: Describe what this pipeline does and why it matters.
  Focus on business value, not implementation details.
-->

### Description

[Provide a clear, concise description of what this data pipeline does, what business problem it solves, and key objectives]

### Schedule & SLA Requirements

- **Frequency**: [Daily / Weekly / On-Demand / Event-driven]
- **Schedule**: [e.g., "Every day at 02:00 UTC"]
- **SLA**: [e.g., "Data must be fresh within 24 hours"]
- **Business Impact**: [What happens if SLA is missed]

---

## Data Sources *(mandatory)*

<!--
  ACTION REQUIRED: Document all data sources with schemas and volumes.
  Include connection details and freshness requirements.
-->

### Source 1: [Source Name]

| Property | Value |
|----------|-------|
| **System** | [Source system name, e.g., "PostgreSQL", "S3", "API"] |
| **Connection** | [Connection details or reference] |
| **Refresh Rate** | [e.g., "Real-time", "Daily", "Weekly"] |
| **Expected Volume** | [e.g., "10 million rows/day"] |
| **Data Freshness Requirement** | [e.g., "< 1 hour"] |

**Schema**:

```
- [column_name]: [data_type] [nullable: Y/N] - [description]
- [column_name]: [data_type] [nullable: Y/N] - [description]
```

### Source 2: [Source Name]

[Repeat pattern above for additional sources]

---

## Pipeline Stages *(mandatory)*

<!--
  IMPORTANT: Pipeline stages should be PRIORITIZED as data flows ordered by importance.
  Each stage must be INDEPENDENTLY TESTABLE - meaning if you implement just ONE stage,
  you should still have a viable MVP that delivers value.
  
  Assign priorities (P1, P2, P3, etc.) to each stage, where P1 is the most critical.
  Think of each stage as a standalone slice of functionality that can be:
  - Developed independently
  - Tested independently
  - Deployed independently
  - Demonstrated to stakeholders independently
-->

### Stage 1: Ingest [Source Name] (Priority: P1)

**Objective**: Load raw data from source system into Bronze layer (Unity Catalog)

**Why this priority**: [Explain the value and why it has this priority level - e.g., "Critical for all downstream processing" or "Enables initial business value"]

**Independent Test**: [Describe how this stage can be tested independently - e.g., "Can be fully tested by loading sample data and verifying Bronze table contents"]

**Source Details**:
- **System**: [Source system]
- **Connection**: [Connection reference]
- **Extract Method**: [Full load / Incremental / CDC]

**Target**:
- **Catalog**: [catalog_name]
- **Schema**: bronze
- **Table**: [table_name]
- **Load Pattern**: [Append / Overwrite / Merge]

**Data Quality Checks**:
- **DQ-001**: [Column name] MUST NOT be NULL for [percentage]% of records
- **DQ-002**: [Column name] MUST match pattern [regex/format]
- **DQ-003**: [Column name] MUST be within range [min-max]

**Acceptance Scenarios**:

1. **Given** [source state], **When** [ingestion runs], **Then** [expected output]
2. **Given** [source state], **When** [ingestion runs], **Then** [expected outcome]

**Dependencies**:
- [Upstream stage or system dependency]

---

### Stage 2: Transform [Entity Name] (Priority: P2)

**Objective**: Apply business logic and transformations to create Silver layer (cleaned, deduplicated data)

**Why this priority**: [Explain the value and why it has this priority level]

**Independent Test**: [Describe how this stage can be tested independently]

**Input Tables**:
- [bronze.table_name]

**Transformations**:
1. [Transformation description]
2. [Transformation description]
3. [Transformation description]

**Output**:
- **Catalog**: [catalog_name]
- **Schema**: silver
- **Table**: [table_name]

**Data Quality Checks**:
- **DQ-101**: No duplicate records (deduplicated on [keys])
- **DQ-102**: All required columns populated
- **DQ-103**: Data type consistency with schema

**Acceptance Scenarios**:

1. **Given** [input state], **When** [transformation runs], **Then** [row counts match expectations]
2. **Given** [input state], **When** [transformation runs], **Then** [data quality checks pass]

**Dependencies**:
- Stage 1: Ingest [Source Name]

---

### Stage 3: Aggregate/Output [Entity Name] (Priority: P3)

**Objective**: Create Gold layer with business-ready aggregations and features

**Why this priority**: [Explain the value and why it has this priority level]

**Independent Test**: [Describe how this stage can be tested independently]

**Input Tables**:
- [silver.table_name]

**Business Logic**:
1. [Aggregation/calculation]
2. [Aggregation/calculation]

**Output**:
- **Catalog**: [catalog_name]
- **Schema**: gold
- **Table**: [table_name]

**Data Quality Checks**:
- **DQ-201**: Aggregate totals match source within [tolerance]%
- **DQ-202**: Time-based aggregations cover all expected periods

**Acceptance Scenarios**:

1. **Given** [prepared data], **When** [aggregation runs], **Then** [output matches expected metrics]

**Dependencies**:
- Stage 2: Transform [Entity Name]

---

[Add more pipeline stages as needed, each with an assigned priority]

### Edge Cases

<!--
  ACTION REQUIRED: The content in this section represents placeholders.
  Fill them out with the right edge cases.
-->

- What happens when [source data is missing or delayed]?
- How does pipeline handle [schema changes in source]?
- What occurs when [data quality thresholds are breached]?
- How does system handle [duplicate processing attempts]?

---

## Data Quality Requirements *(mandatory)*

<!--
  ACTION REQUIRED: Define measurable data quality thresholds.
  These must be specific, measurable, and actionable.
-->

### Quality Thresholds

| Check | Requirement | Action if Failed |
|-------|-------------|-----------------|
| [Check Name] | [Threshold] | [Action: Alert/Halt/Log] |
| Null Rate | Max 0.1% in critical columns | Halt pipeline |
| Duplicate Rate | 0% in unique key columns | Alert and investigate |
| Schema Compliance | 100% | Halt pipeline |
| Freshness SLA | Data < [X] hours old | Alert monitoring |

### Data Validation Rules

<!--
  IMPORTANT: These are examples only. Do NOT include implementation code in the spec.
  Describe validation rules in business terms, not code.
-->

**Example Quality Rules** (describe in business terms):

- Row count validation: Output table MUST contain data
- Null rate validation: Null percentage MUST be below threshold
- Duplicate validation: Unique key columns MUST have no duplicates
- Freshness validation: Data timestamp MUST be within SLA window

---

## Success Criteria *(mandatory)*

<!--
  ACTION REQUIRED: Define measurable success criteria.
  These must be technology-agnostic and measurable from a business perspective.
-->

### Measurable Outcomes

- **SC-001**: [Pipeline freshness metric, e.g., "Data is refreshed within 4 hours of source update"]
- **SC-002**: [Volume metric, e.g., "Pipeline processes 10M records per day without degradation"]
- **SC-003**: [Quality metric, e.g., "Duplicate rate is less than 0.1% across all tables"]
- **SC-004**: [Business metric, e.g., "Downstream dashboards show data within 24 hours"]
- **SC-005**: [Reliability metric, e.g., "Pipeline achieves 99.5% successful run rate"]

---

## Deployment Environments *(optional)*

<!--
  Include only if environment-specific configurations are important.
  Remove this section if all environments use the same configuration.
-->

### Development (`dev`)

- **Catalog**: `dev_[project_name]`
- **Workspace**: [Dev workspace URL]
- **Cluster**: [Dev cluster config]
- **Compute Resources**: [e.g., "2-worker cluster, 4 cores each"]

### Staging (`staging`)

- **Catalog**: `staging_[project_name]`
- **Workspace**: [Staging workspace URL]
- **Cluster**: [Staging cluster config]

### Production (`prod`)

- **Catalog**: `prod_[project_name]`
- **Workspace**: [Prod workspace URL]
- **Cluster**: [Prod cluster config]
- **Retention Policy**: [e.g., "Keep 90 days of history"]

---

## Dependencies & Integration Points *(optional)*

<!--
  Include only if the pipeline has significant upstream or downstream dependencies.
  Remove this section if the pipeline is standalone.
-->

### Upstream Dependencies

- [System/Pipeline Name]: [Specific tables or data products consumed]

### Downstream Consumers

- [System/Pipeline Name]: [How output is used]
- [Dashboard/BI Tool]: [Specific metrics or tables exposed]

### Key Data Entities *(include if pipeline involves complex data models)*

- **[Entity 1]**: [What it represents, key attributes without implementation]
- **[Entity 2]**: [What it represents, relationships to other entities]

---

## Assumptions & Constraints *(optional)*

<!--
  Document important assumptions made during specification.
  Include only if there are significant constraints or assumptions.
  Remove this section if not applicable.
-->

- [Important assumption about data sources or availability]
- [Known constraint or limitation]
- [Business rule or policy assumption]
- [Future enhancement consideration]
