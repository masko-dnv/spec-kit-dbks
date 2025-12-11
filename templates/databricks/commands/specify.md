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

## Outline

The text the user typed after `/speckit.specify` in the triggering message **is** the pipeline description. Assume you always have it available in this conversation even if `{ARGS}` appears literally below. Do not ask the user to repeat it unless they provided an empty command.

Given that pipeline description, do this:

1. **Generate a concise short name** (2-4 words) for the branch:
   - Analyze the pipeline description and extract the most meaningful keywords
   - Create a 2-4 word short name that captures the essence of the pipeline
   - Use descriptive data-focused format (e.g., "customer-data-pipeline", "sales-etl")
   - Preserve technical terms and data concepts (ETL, CDC, medallion, etc.)
   - Keep it concise but descriptive enough to understand the pipeline at a glance
   - Examples:
     - "Customer data pipeline from CRM" → "customer-crm-pipeline"
     - "Daily sales aggregation ETL" → "daily-sales-etl"
     - "Real-time event processing" → "realtime-events-pipeline"
     - "Bronze to silver transformation for orders" → "orders-bronze-silver"

2. **Check for existing branches before creating new one**:

   a. First, fetch all remote branches to ensure we have the latest information:

      ```bash
      git fetch --all --prune
      ```

   b. Find the highest feature number across all sources for the short-name:
      - Remote branches: `git ls-remote --heads origin | grep -E 'refs/heads/[0-9]+-<short-name>$'`
      - Local branches: `git branch | grep -E '^[* ]*[0-9]+-<short-name>$'`
      - Specs directories: Check for directories matching `specs/[0-9]+-<short-name>`

   c. Determine the next available number:
      - Extract all numbers from all three sources
      - Find the highest number N
      - Use N+1 for the new branch number

   d. Run the script `{SCRIPT}` with the calculated number and short-name:
      - Pass `--number N+1` and `--short-name "your-short-name"` along with the pipeline description
      - Bash example: `{SCRIPT} --json --number 5 --short-name "customer-crm-pipeline" "Customer data pipeline from CRM"`
      - PowerShell example: `{SCRIPT} -Json -Number 5 -ShortName "customer-crm-pipeline" "Customer data pipeline from CRM"`

   **IMPORTANT**:
   - Check all three sources (remote branches, local branches, specs directories) to find the highest number
   - Only match branches/directories with the exact short-name pattern
   - If no existing branches/directories found with this short-name, start with number 1
   - You must only ever run this script once per pipeline
   - The JSON is provided in the terminal as output - always refer to it to get the actual content you're looking for
   - The JSON output will contain BRANCH_NAME and SPEC_FILE paths
   - For single quotes in args like "I'm Groot", use escape syntax: e.g 'I'\''m Groot' (or double-quote if possible: "I'm Groot")

3. Load `templates/databricks/spec-template.md` to understand required sections.

4. Follow this execution flow:

    1. Parse user description from Input
       If empty: ERROR "No pipeline description provided"
    2. Extract key concepts from description
       Identify: data sources, transformation stages, business requirements, SLAs
    3. For unclear aspects:
       - Make informed guesses based on context and data engineering best practices
       - Only mark with [NEEDS CLARIFICATION: specific question] if:
         - The choice significantly impacts pipeline scope or data quality requirements
         - Multiple reasonable interpretations exist with different implications
         - No reasonable default exists
       - **LIMIT: Maximum 3 [NEEDS CLARIFICATION] markers total**
       - Prioritize clarifications by impact: scope > data quality/SLA > performance > technical details
    4. Fill Pipeline Stages section
       If no clear data flow: ERROR "Cannot determine pipeline stages"
    5. Generate Data Quality Requirements
       Each requirement must be measurable and testable
       Use reasonable defaults for unspecified details (document assumptions in Assumptions section)
    6. Define Success Criteria
       Create measurable, technology-agnostic outcomes
       Include both quantitative metrics (freshness, volume, performance) and qualitative measures (data quality, completeness)
       Each criterion must be verifiable without implementation details
    7. Identify Data Sources and Dependencies
    8. Return: SUCCESS (spec ready for planning)

5. Write the specification to SPEC_FILE using the template structure, replacing placeholders with concrete details derived from the pipeline description (arguments) while preserving section order and headings.

6. **Specification Quality Validation**: After writing the initial spec, validate it against quality criteria:

   a. **Create Spec Quality Checklist**: Generate a checklist file at `FEATURE_DIR/checklists/requirements.md` using the checklist template structure with these validation items:

      ```markdown
      # Data Pipeline Specification Quality Checklist: [PIPELINE NAME]
      
      **Purpose**: Validate specification completeness and quality before proceeding to planning
      **Created**: [DATE]
      **Pipeline**: [Link to spec.md]
      
      ## Content Quality
      
      - [ ] No implementation details (Spark APIs, Delta Lake syntax, notebook code)
      - [ ] Focused on data requirements and business needs
      - [ ] Written for non-technical stakeholders
      - [ ] All mandatory sections completed
      
      ## Requirement Completeness
      
      - [ ] No [NEEDS CLARIFICATION] markers remain
      - [ ] Data sources are fully documented with schemas
      - [ ] Pipeline stages have clear inputs and outputs
      - [ ] Data quality requirements are measurable
      - [ ] Success criteria are measurable
      - [ ] Success criteria are technology-agnostic (no Spark/Delta specifics)
      - [ ] All acceptance scenarios are defined
      - [ ] Edge cases and failure modes identified
      - [ ] SLAs and freshness requirements specified
      - [ ] Dependencies and assumptions identified
      
      ## Pipeline Readiness
      
      - [ ] All pipeline stages have clear acceptance criteria
      - [ ] Data flow is complete from source to target
      - [ ] Pipeline meets measurable outcomes defined in Success Criteria
      - [ ] No implementation details leak into specification
      
      ## Notes
      
      - Items marked incomplete require spec updates before `/speckit.clarify` or `/speckit.plan`
      ```

   b. **Run Validation Check**: Review the spec against each checklist item:
      - For each item, determine if it passes or fails
      - Document specific issues found (quote relevant spec sections)

   c. **Handle Validation Results**:

      - **If all items pass**: Mark checklist complete and proceed to step 7

      - **If items fail (excluding [NEEDS CLARIFICATION])**:
        1. List the failing items and specific issues
        2. Update the spec to address each issue
        3. Re-run validation until all items pass (max 3 iterations)
        4. If still failing after 3 iterations, document remaining issues in checklist notes and warn user

      - **If [NEEDS CLARIFICATION] markers remain**:
        1. Extract all [NEEDS CLARIFICATION: ...] markers from the spec
        2. **LIMIT CHECK**: If more than 3 markers exist, keep only the 3 most critical (by scope/data quality/SLA impact) and make informed guesses for the rest
        3. For each clarification needed (max 3), present options to user in this format:

           ```markdown
           ## Question [N]: [Topic]
           
           **Context**: [Quote relevant spec section]
           
           **What we need to know**: [Specific question from NEEDS CLARIFICATION marker]
           
           **Suggested Answers**:
           
           | Option | Answer | Implications |
           |--------|--------|--------------|
           | A      | [First suggested answer] | [What this means for the pipeline] |
           | B      | [Second suggested answer] | [What this means for the pipeline] |
           | C      | [Third suggested answer] | [What this means for the pipeline] |
           | Custom | Provide your own answer | [Explain how to provide custom input] |
           
           **Your choice**: _[Wait for user response]_
           ```

        4. **CRITICAL - Table Formatting**: Ensure markdown tables are properly formatted:
           - Use consistent spacing with pipes aligned
           - Each cell should have spaces around content: `| Content |` not `|Content|`
           - Header separator must have at least 3 dashes: `|--------|`
           - Test that the table renders correctly in markdown preview
        5. Number questions sequentially (Q1, Q2, Q3 - max 3 total)
        6. Present all questions together before waiting for responses
        7. Wait for user to respond with their choices for all questions (e.g., "Q1: A, Q2: Custom - [details], Q3: B")
        8. Update the spec by replacing each [NEEDS CLARIFICATION] marker with the user's selected or provided answer
        9. Re-run validation after all clarifications are resolved

   d. **Update Checklist**: After each validation iteration, update the checklist file with current pass/fail status

7. Report completion with branch name, spec file path, checklist results, and readiness for the next phase (`/speckit.clarify` or `/speckit.plan`).

**NOTE:** The script creates and checks out the new branch and initializes the spec file before writing.

## Data Pipeline Specification Guidelines

### Overview

You are helping create a **data pipeline specification document** for a Databricks data engineering project. This is different from a typical web app feature specification—it focuses on data sources, transformation stages, and data quality requirements rather than user stories.

### Key Focus Areas

Based on the user's description, create a comprehensive data pipeline specification that covers:

### Key Focus Areas

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

### Section Requirements

- **Mandatory sections**: Must be completed for every pipeline
- **Optional sections**: Include only when relevant to the pipeline
- When a section doesn't apply, remove it entirely (don't leave as "N/A")

### For AI Generation

When creating this spec from a user prompt:

1. **Make informed guesses**: Use context, data engineering best practices, and common patterns to fill gaps
2. **Document assumptions**: Record reasonable defaults in the Assumptions section
3. **Limit clarifications**: Maximum 3 [NEEDS CLARIFICATION] markers - use only for critical decisions that:
   - Significantly impact pipeline scope or data quality requirements
   - Have multiple reasonable interpretations with different implications
   - Lack any reasonable default
4. **Prioritize clarifications**: scope > data quality/SLA > performance > technical details
5. **Think like a data engineer**: Every vague requirement should fail the "measurable and testable" checklist item
6. **Common areas needing clarification** (only if no reasonable default exists):
   - Pipeline scope and data boundaries (include/exclude specific sources)
   - Data freshness and SLA requirements (if legally/financially significant)
   - Data quality thresholds (when business-critical)
   - Upstream/downstream dependencies (if unclear from context)

**Examples of reasonable defaults** (don't ask about these):

- Bronze layer: Full snapshots unless incremental specified
- Silver layer: Standard deduplication and type casting
- Gold layer: Daily aggregations unless specified otherwise
- Data retention: 30 days bronze, 90 days silver, 365 days gold
- Error handling: Log failures and alert on threshold breach
- Schedule: Daily batch unless real-time specified

### Success Criteria Guidelines

Success criteria must be:

1. **Measurable**: Include specific metrics (freshness, data volume, quality thresholds, performance)
2. **Technology-agnostic**: No mention of Spark, Delta Lake, Databricks APIs, or implementation details
3. **Business-focused**: Describe outcomes from business/data consumer perspective, not system internals
4. **Verifiable**: Can be tested/validated without knowing implementation details

**Good examples**:

- "Data is refreshed within 4 hours of source update"
- "Pipeline processes 10M records per day"
- "Duplicate rate is less than 0.1%"
- "All required fields populated for 99.9% of records"
- "Downstream dashboards show data within 24 hours"

**Bad examples** (implementation-focused):

- "Delta merge completes in under 10 minutes" (too technical, use "Data updates complete within SLA")
- "Spark job uses 50 executors" (implementation detail)
- "Unity Catalog permissions configured" (infrastructure, not business outcome)
- "Autoloader handles schema evolution" (technology-specific)

---

## Example Pipeline

If the user says: "I need to build a customer data pipeline that combines customer records from our CRM and web events, deduplicates them, calculates daily metrics for each customer segment, and powers our analytics dashboards. Data comes in daily, must be fresh within 24 hours, and we need to validate that no customer records are missing."

You would create a spec with:

- **Overview**: Daily customer data pipeline, deduplication, segment-level metrics, 24-hour freshness SLA
- **Sources**: CRM API (customer records, ~100K rows/day), web events database (behavioral data, ~5M events/day)
- **Stages**: 
  - Ingest: Raw CRM → bronze.raw_customers, Raw events → bronze.raw_events
  - Transform: Deduplicate & clean → silver.customers, silver.events
  - Aggregate: Calculate metrics → gold.customer_daily_metrics
- **Quality**: No duplicate customer IDs (DQ check), null rate < 0.1% on required fields, all customer records accounted for
- **Success Criteria**: Data fresh within 24 hours, 99.9% data quality, dashboards updated daily
- **Dependencies**: Dashboards consume gold.customer_daily_metrics

---

## Quick Tips

- Focus on **WHAT** data flows and **WHY** it's needed
- Avoid **HOW** to implement (no Spark code, Delta syntax, notebook structure)
- Written for data analysts and business stakeholders, not just engineers
- DO NOT create any checklists that are embedded in the spec. That will be a separate command
- **Be specific about sources**: Include connection types, authentication methods, expected row counts
- **Define each transformation**: Explain deduplication keys, business rules, filtering logic
- **Set realistic thresholds**: Use actual business requirements, not generic defaults
- **Think in layers**: Bronze (raw) → Silver (clean) → Gold (business-ready)
- **Consider testing**: Mention how you'll validate the pipeline works correctly
- **Plan for failure**: Document what happens if quality checks fail
