---
description: Create a data pipeline quality assurance checklist covering functionality, performance, and operational readiness.
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

You are creating a **quality assurance checklist** for a Databricks data pipeline. This checklist ensures the pipeline is production-ready, meets business requirements, performs well, and can be safely operated.

---

## Checklist Sections

Create a comprehensive checklist organized into these categories:

### 1. **Functional Requirements**
Verify that all data pipeline stages work correctly:

```markdown
## Functional Requirements

### Data Ingestion
- [ ] All source systems connect successfully
- [ ] Data volumes match expectations (within X% variance)
- [ ] Latency from source update to Bronze table is < [SLA]
- [ ] Handling of missing/late-arriving data is documented
- [ ] Failure scenarios (API down, connection timeout) are handled

### Data Transformation
- [ ] Deduplication logic removes expected duplicates (test with known duplicates)
- [ ] Business rule transformations produce correct output
- [ ] Null handling follows specifications
- [ ] Derived columns (e.g., full_name = first_name + last_name) are calculated correctly
- [ ] Filtering logic (e.g., is_deleted = false) works as specified
- [ ] Joins produce correct results (no missing or spurious rows)

### Output & Aggregation
- [ ] Aggregates sum to source table totals (within 0.1%)
- [ ] Time series data covers all expected periods
- [ ] Dimension values are correct (no unexpected values)
- [ ] Metric calculations match business requirements
- [ ] Output tables are readable by downstream consumers
```

### 2. **Data Quality**
Confirm quality metrics meet thresholds:

```markdown
## Data Quality

- [ ] Null rates in critical columns < 0.1%
- [ ] No unexpected duplicate records
- [ ] All required columns are populated
- [ ] Data types match schema
- [ ] String values are within expected lengths
- [ ] Numeric ranges are within boundaries
- [ ] Dates are valid and within expected ranges
- [ ] Foreign key relationships are valid
- [ ] Aggregate totals match source within tolerance
- [ ] No data loss between layers (Bronze → Silver → Gold)
```

### 3. **Testing Coverage**
Ensure adequate test coverage:

```markdown
## Testing

- [ ] Unit test coverage > 80% of transformation logic
- [ ] Unit tests for data quality validators
- [ ] Integration tests for end-to-end pipeline
- [ ] Integration tests with sample data fixtures
- [ ] Error handling is tested (missing source, bad data, etc.)
- [ ] Edge cases are tested (empty input, null values, duplicates)
- [ ] Tests run successfully with: pytest tests/unit && pytest tests/integration
- [ ] Coverage report shows gaps
```

### 4. **Performance**
Verify pipeline executes efficiently:

```markdown
## Performance

- [ ] Ingest stage completes in < [X] minutes
- [ ] Transform stage completes in < [Y] minutes
- [ ] Output stage completes in < [Z] minutes
- [ ] Cluster utilization is 60-80% (not idle, not overloaded)
- [ ] No data skew issues (even partition sizes)
- [ ] Queries on Gold tables return results in < 30 seconds
- [ ] No unnecessary full table scans
- [ ] Caching strategy is appropriate for data volumes
```

### 5. **Code Quality**
Ensure code follows standards:

```markdown
## Code Quality

- [ ] All Python files pass ruff linting
- [ ] All code is formatted with black
- [ ] Function signatures have type hints
- [ ] All public functions have docstrings
- [ ] Complex logic has inline comments
- [ ] No hardcoded values (use configuration)
- [ ] Error messages are clear and actionable
- [ ] No TODOs or FIXMEs in production code
- [ ] Notebooks follow naming conventions
- [ ] Module structure follows src-layout pattern
```

### 6. **Documentation**
Verify complete and accurate documentation:

```markdown
## Documentation

- [ ] README.md exists with project overview
- [ ] Setup instructions are clear and tested
- [ ] Data lineage is documented (sources → tables → consumers)
- [ ] Table schemas are documented with column descriptions
- [ ] Transformation logic is documented with examples
- [ ] API/connection details for sources are documented
- [ ] Known limitations are documented
- [ ] Troubleshooting guide exists for common issues
- [ ] Runbook exists for ops team (how to run, check status, rollback)
```

### 7. **Deployment & Operations**
Ensure production readiness:

```markdown
## Deployment & Operations

- [ ] Bundle validates: databricks bundle validate -t [target]
- [ ] Bundle deploys successfully: databricks bundle deploy -t [target]
- [ ] Jobs are defined in databricks.yml (not manual)
- [ ] Job scheduling is configured correctly
- [ ] Retry policies are set appropriately
- [ ] Timeout values are realistic
- [ ] Job names follow naming conventions
- [ ] Logs are accessible and readable
- [ ] Alerts are configured for failures
- [ ] Runbook for manual job execution exists
```

### 8. **Security & Access Control**
Verify security best practices:

```markdown
## Security & Access Control

- [ ] Credentials are in .databrickscfg (not in code)
- [ ] Service principals are used for automation
- [ ] Access control is configured via Unity Catalog
- [ ] Bronze layer access is restricted to data engineers
- [ ] Silver layer access is restricted to data engineers
- [ ] Gold layer access is granted to analysts/consumers
- [ ] Sensitive columns are masked if needed
- [ ] No hardcoded secrets in code or configs
```

### 9. **Multi-Environment**
Confirm dev/staging/prod are properly configured:

```markdown
## Multi-Environment Support

### Dev
- [ ] Dev environment uses smaller clusters (cost optimization)
- [ ] Dev schedule is different from prod (or disabled)
- [ ] Dev data is isolated in dev_[catalog] namespace

### Staging
- [ ] Staging configuration mirrors production
- [ ] Staging uses realistic data volumes
- [ ] Staging schedule may be simpler than production

### Production
- [ ] Production cluster is sized for actual data volumes
- [ ] Production schedule matches business requirements
- [ ] Production data is isolated in prod_[catalog] namespace
- [ ] All targets deployed successfully with bundle
```

### 10. **Sign-Off & Approval**
Final reviews and approvals:

```markdown
## Sign-Off

- [ ] Data engineer review (code quality, correctness)
- [ ] Data architect review (design, performance, scalability)
- [ ] Operations review (deployability, monitoring, runbooks)
- [ ] Business owner approval (requirements met, SLAs acceptable)
- [ ] Security review (access control, data protection)
- [ ] Final smoke test in production (data present, quality checks pass)
```

---

## Example Quality Gate Checks

### Bronze Layer Validation
```
- Row count: Expected [X], Actual [Y], Variance < 1%
- Null checks: [column_1] null%, [column_2] null%
- Ingestion latency: [X] hours from source update
```

### Silver Layer Validation
```
- Duplicates removed: Before [X] rows, After [Y] rows
- Null rate: < 0.1% in critical columns
- Schema validation: All [N] columns present and correct type
```

### Gold Layer Validation
```
- Aggregate sum: Expected [X], Actual [Y], Variance < 0.1%
- Time series completeness: All expected dates present
- Dimension cardinality: Expected [N] unique values
```

---

## Output Format

Create a markdown checklist organized by section:

```markdown
# QA Checklist: [PIPELINE_NAME]

**Date**: [Date]
**Prepared By**: [Name]
**Status**: [ ] Complete / [ ] In Progress / [ ] Blocked

## Functional Requirements
- [ ] All ingestion points work
- [ ] All transformations produce correct output
- [ ] All aggregations are accurate

[Continue through all sections...]

## Overall Status
- [ ] Ready for Production
- [ ] Requires Additional Work
- [ ] Blocked (list blockers)

## Sign-Off
| Role | Name | Date | Approved |
|------|------|------|----------|
| Data Engineer | | | |
| Data Architect | | | |
| Operations | | | |
| Business Owner | | | |
```

---

## Tips for QA

- **Test in isolation first**: Test each stage independently before end-to-end
- **Use realistic data**: QA with production-like data volumes
- **Automate checks**: Implement validation as code, not manual inspection
- **Document deviations**: If something doesn't match checklist, document why
- **Plan for failure**: Test what happens when sources fail or data is wrong
- **Verify monitoring**: Ensure alerts would actually catch real issues
