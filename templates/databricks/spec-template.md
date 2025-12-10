# Data Pipeline Specification: [PIPELINE_NAME]

**Author**: [Author Name]  
**Date**: [Date]  
**Status**: Draft

---

## Pipeline Overview

### Description

[Provide a clear, concise description of what this data pipeline does, what business problem it solves, and key objectives]

### Schedule & Trigger

- **Frequency**: [Daily / Weekly / On-Demand / Event-driven]
- **Schedule**: [e.g., "Every day at 02:00 UTC"]
- **SLA**: [e.g., "Data must be fresh within 24 hours"]

### Success Criteria

- [Criterion 1]
- [Criterion 2]
- [Criterion 3]

---

## Data Sources

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

## Pipeline Stages

### Stage 1: Ingest [Source Name] (Priority: P1)

**Objective**: Load raw data from source system into Bronze layer (Unity Catalog)

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
- DQ-001: [Column name] MUST NOT be NULL for [percentage]% of records
- DQ-002: [Column name] MUST match pattern [regex/format]
- DQ-003: [Column name] MUST be within range [min-max]

**Acceptance Scenarios**:
1. Given [source state], When [notebook runs], Then [expected output]
2. Given [source state], When [notebook runs], Then [expected output]

**Dependencies**:
- [Upstream stage or system dependency]

---

### Stage 2: Transform [Entity Name] (Priority: P1)

**Objective**: Apply business logic and transformations to create Silver layer (cleaned, deduplicated data)

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
- DQ-101: No duplicate records (deduplicated on [keys])
- DQ-102: All required columns populated
- DQ-103: Data type consistency with schema

**Acceptance Scenarios**:
1. Given [input state], When [transformation runs], Then [row counts match expectations]
2. Given [input state], When [transformation runs], Then [data quality checks pass]

**Dependencies**:
- Stage 1: Ingest [Source Name]

---

### Stage 3: Aggregate/Output [Entity Name] (Priority: P1)

**Objective**: Create Gold layer with business-ready aggregations and features

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
- DQ-201: Aggregate totals match source within [tolerance]%
- DQ-202: Time-based aggregations cover all expected periods

**Acceptance Scenarios**:
1. Given [prepared data], When [aggregation runs], Then [output matches expected metrics]

**Dependencies**:
- Stage 2: Transform [Entity Name]

---

## Data Quality Requirements

### Quality Thresholds

| Check | Requirement | Action if Failed |
|-------|-------------|-----------------|
| [Check Name] | [Threshold] | [Action: Alert/Halt/Log] |
| Null Rate | Max 0.1% in critical columns | Halt pipeline |
| Duplicate Rate | 0% in unique key columns | Alert and investigate |
| Schema Compliance | 100% | Halt pipeline |
| Freshness SLA | Data < [X] hours old | Alert monitoring |

### Data Validation Rules

**Custom Checks**:

```python
# Example validation logic that could be implemented
def validate_pipeline():
    # Check row counts
    assert expected_rows > 0, "Output table must have data"
    
    # Check null counts
    assert null_pct < 0.001, "Null rate exceeds threshold"
    
    # Check for duplicates
    assert duplicate_count == 0, "Duplicates detected"
```

---

## Deployment Environments

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

## Dependencies & Integration Points

### Upstream Dependencies

- [System/Pipeline Name]: [Specific tables or data products consumed]

### Downstream Consumers

- [System/Pipeline Name]: [How output is used]
- [Dashboard/BI Tool]: [Specific metrics or tables exposed]

### API & Service Integrations

- [API Name]: [Purpose and endpoint]

---

## Notes & Constraints

- [Important constraint or assumption]
- [Known limitation]
- [Future enhancement consideration]

---

## Sign-Off

| Role | Name | Date | Signature |
|------|------|------|-----------|
| Data Engineer | [Name] | [Date] | [ ] |
| Data Architect | [Name] | [Date] | [ ] |
| Business Owner | [Name] | [Date] | [ ] |
