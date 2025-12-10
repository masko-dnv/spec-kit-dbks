---
description: Analyze and validate the completeness and consistency of your Databricks data pipeline specification and implementation.
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

You are performing a **completeness and consistency analysis** of a Databricks data pipeline. Verify that the specification, plan, and implementation are coherent, complete, and free of contradictions.

---

## Analysis Checklist

### 1. **Specification Completeness**

Verify the specification document includes all necessary sections:

```markdown
## Specification Completeness

- [ ] Pipeline Overview (description, objectives, schedule)
- [ ] All Data Sources documented (systems, schemas, volumes)
- [ ] All Pipeline Stages defined (Ingest, Transform, Output)
- [ ] Data Quality Requirements specified (thresholds, validation rules)
- [ ] Deployment Environments described (dev, staging, prod)
- [ ] Dependencies & Integrations documented
- [ ] Acceptance Scenarios for each stage
- [ ] Known constraints and assumptions listed
- [ ] Sign-off section for stakeholders

**Missing Sections**: [List any missing parts]
```

### 2. **Plan-to-Specification Alignment**

Verify the implementation plan matches the specification:

```markdown
## Specification-Plan Alignment

- [ ] All sources in spec have corresponding ingest notebooks in plan
- [ ] All transformations in spec have corresponding notebooks in plan
- [ ] All output metrics in spec have corresponding aggregations in plan
- [ ] Data quality checks in spec are implemented in plan
- [ ] DLB version and compute types are specified in plan
- [ ] Testing strategy aligns with spec requirements
- [ ] Timeline estimates are realistic for scope
- [ ] Resource requirements match specification complexity

**Misalignments Found**: [List any discrepancies]
```

### 3. **Data Lineage Validation**

Verify data flows consistently from source to output:

```markdown
## Data Lineage Validation

- [ ] Every source table in spec appears in Bronze layer
- [ ] Every Bronze table has a corresponding Silver transformation
- [ ] Every Silver table contributes to Gold aggregations
- [ ] All Gold tables are consumed by documented downstream systems
- [ ] No tables are created but never used
- [ ] No required transformations are missing
- [ ] Join/merge operations are documented
- [ ] Data filters are explained (why drop certain records?)

**Lineage Issues**: [List any breaks in flow]
```

### 4. **Schema Consistency**

Verify schema definitions are consistent across layers:

```markdown
## Schema Consistency

- [ ] Bronze table columns match source data schema
- [ ] Silver table deduplication keys are correctly defined
- [ ] Primary keys are consistent across related tables
- [ ] Foreign key relationships are documented
- [ ] Data types are consistent (e.g., customer_id is always STRING or always INT)
- [ ] Nullable constraints are explicitly defined
- [ ] Column names follow consistent naming convention
- [ ] No duplicate or conflicting column definitions

**Schema Issues**: [List inconsistencies]
```

### 5. **Task-to-Plan Alignment**

Verify task list matches implementation plan:

```markdown
## Task-Plan Alignment

- [ ] Every phase in plan corresponds to a phase in task list
- [ ] Task descriptions are specific and actionable
- [ ] Tasks are appropriately sized (4-8 hours each)
- [ ] Task dependencies are documented and correct
- [ ] P1/P2 priorities align with phase importance
- [ ] Estimated effort sums to reasonable timeline
- [ ] No gaps between plan and tasks
- [ ] No duplicate tasks

**Task-Plan Issues**: [List discrepancies]
```

### 6. **Testing Coverage**

Verify testing strategy covers all components:

```markdown
## Testing Coverage

- [ ] Unit tests for each transformation function
- [ ] Unit tests for data quality validators
- [ ] Integration tests for each notebook
- [ ] End-to-end tests for full pipeline
- [ ] Edge case tests (nulls, empty input, duplicates)
- [ ] Error handling tests (source failures, bad data)
- [ ] Performance tests for large data volumes
- [ ] Test fixtures for sample data

**Testing Gaps**: [List uncovered scenarios]
```

### 7. **Quality Requirements Completeness**

Verify all quality checks are specified and measurable:

```markdown
## Quality Requirements

- [ ] Each critical column has a null rate threshold
- [ ] Duplicate detection logic is defined
- [ ] Aggregation accuracy tolerance is specified (e.g., < 0.1% variance)
- [ ] Data freshness SLA is documented
- [ ] Validation rules are testable
- [ ] Alert thresholds are defined
- [ ] Quality checks are automated (not manual)

**Quality Gaps**: [List undefined thresholds]
```

### 8. **Performance & Resource Alignment**

Verify performance requirements match resource allocation:

```markdown
## Performance-Resource Alignment

- [ ] Cluster size is appropriate for data volume
- [ ] Partition strategy matches query patterns
- [ ] Caching strategy aligns with data access
- [ ] Job timeouts are realistic for data volume
- [ ] SLA latency matches cluster performance
- [ ] Cost estimates are within budget
- [ ] No unnecessarily large clusters for small data

**Performance Issues**: [List misalignments]
```

### 9. **Documentation Completeness**

Verify all documentation is present:

```markdown
## Documentation Completeness

- [ ] README with setup and usage instructions
- [ ] Architecture documentation
- [ ] Data lineage diagram or description
- [ ] Table schema documentation
- [ ] Transformation logic explanations
- [ ] Testing documentation
- [ ] Operations runbook
- [ ] Troubleshooting guide
- [ ] API/connection documentation for sources

**Missing Documentation**: [List undocumented areas]
```

### 10. **Deployment & Operations Readiness**

Verify production deployment is fully planned:

```markdown
## Deployment & Operations Readiness

- [ ] databricks.yml is fully configured
- [ ] Multi-environment configuration is complete (dev/staging/prod)
- [ ] Job definitions have all required parameters
- [ ] Monitoring & alerting is configured
- [ ] Disaster recovery plan exists
- [ ] Rollback procedures are documented
- [ ] On-call procedures are defined
- [ ] Escalation contacts are documented

**Deployment Gaps**: [List missing operational details]
```

---

## Cross-Document Consistency Checks

### Terminology Consistency
- Are entity names consistent across spec, plan, and tasks?
- Are column names spelled the same way everywhere?
- Are acronyms defined and used consistently?

**Inconsistencies**: [List any terminology mismatches]

### Timeline Alignment
- Do task estimates sum to planned phases?
- Are there unrealistic compression factors?
- Is slack time built in for unknowns?

**Timeline Issues**: [List any issues]

### Responsibility Clarity
- Is it clear who owns each component?
- Are dependencies between teams documented?
- Are handoff points clear?

**Clarity Issues**: [List unclear responsibilities]

---

## Issue Categorization

Categorize findings:

```markdown
## Issues Found

### Critical (Blocks Implementation)
1. [Issue]: [Impact]
2. [Issue]: [Impact]

### Important (Should Fix Before Production)
1. [Issue]: [Workaround]
2. [Issue]: [Workaround]

### Nice-to-Have (Post-Launch Improvements)
1. [Issue]: [Rationale]
2. [Issue]: [Rationale]
```

---

## Recommendations

For each issue found:

1. **Recommend a fix**: How to resolve the inconsistency
2. **Estimate effort**: How long to implement fix
3. **Set priority**: Critical, Important, Nice-to-Have
4. **Assign owner**: Who should handle the fix

---

## Sign-Off

```markdown
## Analysis Sign-Off

| Check | Status | Comments |
|-------|--------|----------|
| Specification Complete | [ ] Pass / [ ] Needs Work | |
| Plan Aligns with Spec | [ ] Pass / [ ] Needs Work | |
| Data Lineage Valid | [ ] Pass / [ ] Needs Work | |
| Schema Consistent | [ ] Pass / [ ] Needs Work | |
| Testing Adequate | [ ] Pass / [ ] Needs Work | |
| Documentation Complete | [ ] Pass / [ ] Needs Work | |
| Deployment Ready | [ ] Pass / [ ] Needs Work | |

**Overall Status**: [ ] Ready for Implementation / [ ] Needs Refinement

**Prepared By**: [Name]  
**Date**: [Date]
```

---

## Next Steps

1. Fix critical issues before implementation starts
2. Schedule important issues for early phases
3. Add nice-to-have improvements to backlog
4. Review with team to get alignment
5. Update spec/plan based on feedback
6. Re-analyze if significant changes made
