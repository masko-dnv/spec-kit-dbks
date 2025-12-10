# Implementation Plan: [PIPELINE_NAME]

**Prepared by**: [Data Engineer Name]  
**Date**: [Date]  
**Status**: Draft

---

## Technical Context

### Environment & Prerequisites

| Aspect | Value |
|--------|-------|
| **Python Version** | 3.12+ (Databricks Runtime compatible) |
| **Databricks Runtime** | [e.g., "14.3 LTS" or "17.3 LTS"] |
| **Compute Type** | [Job Clusters / All-Purpose / Serverless] |
| **Storage Backend** | Delta Lake on Unity Catalog |
| **Catalog** | [catalog_name] |
| **Schemas** | bronze, silver, gold |
| **Testing Framework** | pytest with databricks-connect |
| **Dependency Manager** | uv (UV package manager) |
| **Bundle Targets** | dev, staging, prod |

### Development Environment Setup

```bash
# Clone repository
git clone [repo-url]
cd [project-name]

# Install dependencies with uv
uv sync

# Install Databricks Connect for your cluster runtime
# (Already in pyproject.toml, matched to your DBR version)
uv pip install databricks-connect==14.3.*

# Configure Databricks connection
databricks config --host https://[workspace].azuredatabricks.net --token [your-token]

# Verify connection
databricks bundle validate
```

---

## Project Structure

```
[project-name]/
├── .github/                    # GitHub Actions & CI/CD workflows
│   └── workflows/
│       └── test-and-deploy.yml # Automated testing & deployment
├── .vscode/                    # VS Code workspace settings
│   └── settings.json           # Databricks Connect config
├── .gitignore                  # Standard Python/Databricks ignores
├── docs/                       # Project documentation
│   ├── README.md               # Project overview
│   ├── ARCHITECTURE.md         # Data pipeline architecture
│   ├── DATA_CATALOG.md         # Table schemas & metadata
│   └── TROUBLESHOOTING.md      # Common issues & solutions
├── fixtures/                   # Test data
│   └── sample_data/            # CSV/Delta files for testing
├── notebooks/                  # Databricks notebooks for orchestration
│   ├── _setup/                 # Setup & teardown notebooks
│   │   └── create_tables.py    # DDL to initialize schema
│   ├── ingest_[source].py      # Data ingestion notebook
│   ├── transform_[entity].py   # Transformation notebook
│   └── output_[table].py       # Final aggregation/output
├── resources/                  # Databricks Asset Bundle configs
│   ├── [pipeline].job.yml      # Job definition with tasks
│   └── [pipeline].workflow.yml # Advanced workflow definition
├── scratch/                    # Exploratory notebooks (NOT deployed)
│   └── exploration.py          # Ad-hoc analysis during development
├── src/                        # Reusable Python modules
│   └── [package_name]/
│       ├── __init__.py
│       ├── transformations/    # Data transformation functions
│       │   └── [entity].py
│       ├── validators/         # Data quality validators
│       │   └── checks.py
│       └── utils/              # Utility functions
│           └── helpers.py
├── tests/                      # Automated tests
│   ├── unit/                   # Unit tests (no Databricks)
│   │   ├── test_transformations.py
│   │   └── test_validators.py
│   └── integration/            # Integration tests (with Databricks)
│       ├── test_ingest.py
│       ├── test_transform.py
│       └── conftest.py         # pytest fixtures
├── .databrickscfg              # Databricks credentials (gitignored)
├── databricks.yml              # Databricks Asset Bundle config
├── pyproject.toml              # Python project & dependency config
├── README.md                   # Project overview
└── CHANGELOG.md                # Version history

```

---

## Detailed Component Design

### 1. Databricks Asset Bundle Configuration

**Purpose**: Infrastructure-as-code for Databricks resources (jobs, clusters, permissions)

**File**: `databricks.yml`

```yaml
bundle:
  name: [project-name]
  version: 0.1.0

variables:
  catalog:
    description: Unity Catalog name
    default: dev_[project_name]
  
  schema:
    description: Primary schema name
    default: pipeline

resources:
  jobs:
    daily_ingest_job:
      name: "[Project] Daily Data Ingest"
      description: "Load raw data from source systems"
      tasks:
        - task_key: ingest_[source]
          notebook_task:
            notebook_path: ./notebooks/ingest_[source]
          job_cluster_key: pipeline_cluster
          
    daily_transform_job:
      name: "[Project] Daily Transformations"
      description: "Apply business logic and create refined datasets"
      tasks:
        - task_key: transform_[entity]
          depends_on:
            - task_key: ingest_[source]
          notebook_task:
            notebook_path: ./notebooks/transform_[entity]
          job_cluster_key: pipeline_cluster
      
      job_clusters:
        - job_cluster_key: pipeline_cluster
          new_cluster:
            spark_version: 14.3.x-scala2.12
            node_type_id: [e.g., Standard_DS3_v2]
            num_workers: 1
            spark_conf:
              "spark.sql.shuffle.partitions": "8"

targets:
  dev:
    default: true
    variables:
      catalog: dev_[project_name]
  
  staging:
    variables:
      catalog: staging_[project_name]
  
  prod:
    variables:
      catalog: prod_[project_name]
```

**Key Features**:
- Parameterized targets (dev/staging/prod)
- Job definitions with task dependencies
- Cluster configurations
- Deployment with `databricks bundle deploy -t <target>`

### 2. Python Dependencies (uv)

**File**: `pyproject.toml`

Separates production dependencies (deployed to Databricks) from dev-only dependencies.

```toml
[project]
name = "[project-name]"
version = "0.1.0"
description = "Databricks data engineering pipeline"
requires-python = ">=3.12"
dependencies = [
    # Production dependencies only - deployed to Databricks cluster
    # Keep this minimal; most packages are pre-installed in Databricks Runtime
]

[dependency-groups]
dev = [
    # Databricks local development
    "databricks-connect==14.3.*",  # Match your cluster runtime
    "databricks-sdk>=0.30.0",
    
    # Testing
    "pytest>=7.4.0",
    "pytest-cov>=4.1.0",
    
    # Notebooks & interactive development
    "jupyter>=1.0.0",
    "notebook>=7.0.0",
    
    # Code quality
    "ruff>=0.1.0",
    "black>=23.0.0",
    
    # Utilities
    "python-dotenv>=1.0.0",
    "pandas>=2.0.0",  # For local testing
]

[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"

[tool.hatch.build.targets.wheel]
packages = ["src/[package_name]"]

[tool.pytest.ini_options]
pythonpath = "src"
testpaths = ["tests"]
markers = [
    "integration: marks tests as integration tests (requires Databricks connection)",
]

[tool.ruff]
line-length = 120
target-version = "py312"

# Databricks built-in globals - prevents F821 undefined name errors
builtins = [
    "spark",
    "dbutils",
    "display",
    "displayHTML",
    "sc",
    "sqlContext",
    "table",
    "udf",
    "getArgument",
]

exclude = [
    ".venv",
    "venv",
    "build",
    "dist",
    "__pycache__",
    "docs",
    "resources",
    "scratch",
    ".vscode",
    ".git",
    "*.ipynb",
]

[tool.ruff.lint.per-file-ignores]
"src/**/__init__.py" = ["F401"]  # Allow unused imports in __init__.py
```

### 3. Notebook Structure

**Data Ingestion**: `notebooks/ingest_[source].py`

```python
# Databricks notebook source

from pyspark.sql import SparkSession
from pyspark.sql.functions import current_timestamp, lit

spark = SparkSession.builder.getOrCreate()

# Read from source
df = spark.read.format("jdbc").load(...)  # or .format("parquet") or API call

# Add metadata columns
df = df.withColumn("ingested_at", current_timestamp())
df = df.withColumn("source_system", lit("[SOURCE_NAME]"))

# Write to Bronze layer
df.write \
    .format("delta") \
    .mode("append") \  # or "overwrite"
    .option("mergeSchema", "true") \
    .saveAsTable("[catalog].[schema].raw_[entity]")

print(f"✓ Ingested {df.count()} records from [SOURCE_NAME]")
```

**Transformation**: `notebooks/transform_[entity].py`

```python
# Databricks notebook source

from pyspark.sql.functions import col, when, coalesce
from pyspark.sql.types import StructType, StructField, StringType, IntegerType

# Read from Bronze
df = spark.table("[catalog].bronze.raw_[entity]")

# Apply transformations
df = df \
    .dropDuplicates([col_key]) \
    .filter(col("is_deleted") == False) \
    .withColumn("full_name", when(...)) \
    .select(...)

# Validate data quality
assert df.count() > 0, "Output table must have data"
null_pct = df.filter(col("[critical_col]").isNull()).count() / df.count()
assert null_pct < 0.001, f"Null rate {null_pct} exceeds threshold"

# Write to Silver
df.write \
    .format("delta") \
    .mode("overwrite") \
    .option("mergeSchema", "true") \
    .saveAsTable("[catalog].[schema].transformed_[entity]")
```

### 4. Testing Strategy

**Unit Tests**: `tests/unit/test_transformations.py`

```python
import pytest
from src.transformations import transform_entity

def test_transform_removes_duplicates():
    input_df = spark.createDataFrame([...], schema="...")
    result = transform_entity(input_df)
    assert result.count() < input_df.count()

def test_transform_validates_schema():
    input_df = spark.createDataFrame([...], schema="...")
    result = transform_entity(input_df)
    assert "[critical_col]" in result.columns
```

**Integration Tests**: `tests/integration/test_transform.py`

```python
import pytest
from databricks.sdk import WorkspaceClient

@pytest.mark.integration
def test_end_to_end_pipeline(spark):
    """Test full pipeline with real Databricks Connect"""
    # Run ingest notebook
    dbx = WorkspaceClient()
    run = dbx.jobs.run_now(job_id=12345)
    
    # Verify output
    df = spark.table("[catalog].[schema].output_table")
    assert df.count() > 0
```

---

## Implementation Phases

### Phase 1: Bundle Setup & Environment (Week 1)

- [ ] Clone repository & initialize `databricks.yml`
- [ ] Configure Databricks CLI & authentication
- [ ] Set up local Python environment with `uv sync`
- [ ] Install Databricks Connect for local development
- [ ] Create VS Code workspace settings

**Deliverable**: Development environment ready for testing

### Phase 2: Schema & Infrastructure (Week 1-2)

- [ ] Create setup notebook (`_setup/create_tables.py`)
- [ ] Define Delta table schemas for Bronze, Silver, Gold layers
- [ ] Create utility/helper modules in `src/`
- [ ] Set up test data in `fixtures/sample_data/`

**Deliverable**: Empty schemas ready for data ingestion

### Phase 3: Data Ingestion (Week 2)

- [ ] Create `ingest_[source].py` notebooks
- [ ] Implement source-specific connection logic
- [ ] Add data quality checks for raw data
- [ ] Create unit tests for ingestion logic
- [ ] Configure job definition in `resources/`

**Deliverable**: Data successfully loading into Bronze layer

### Phase 4: Transformations (Week 3)

- [ ] Create `transform_[entity].py` notebooks
- [ ] Implement business logic per specification
- [ ] Write transformation modules in `src/transformations/`
- [ ] Create comprehensive unit tests
- [ ] Add integration tests with Databricks Connect

**Deliverable**: Cleaned, deduplicated data in Silver layer

### Phase 5: Aggregations & Output (Week 3-4)

- [ ] Create output/aggregation notebooks
- [ ] Implement business metrics & features
- [ ] Write data quality validators
- [ ] Create final integration tests

**Deliverable**: Business-ready data in Gold layer

### Phase 6: Orchestration & CI/CD (Week 4)

- [ ] Configure job workflows in `databricks.yml`
- [ ] Set up GitHub Actions for automated testing
- [ ] Implement deployment pipelines (dev → staging → prod)
- [ ] Add monitoring & alerting

**Deliverable**: Production-ready automated pipeline

---

## Success Metrics

| Metric | Target | Measurement |
|--------|--------|-------------|
| Data Freshness | < 24 hours | Compare ingestion timestamp to current time |
| Data Quality | > 99.9% valid records | Count null/invalid records vs total |
| Pipeline Reliability | > 99% execution success | Monitor job success rate over time |
| Query Performance | < 30 seconds | Measure query execution time on Gold tables |
| Documentation | 100% coverage | Code comments + docs/ folder |
| Test Coverage | > 80% | pytest coverage report |

---

## Known Constraints & Assumptions

- Assumes [DBR version] or later
- Assumes Unity Catalog is enabled in workspace
- Assumes adequate cluster resources for [expected data volume]
- Assumes source systems have stable APIs/connections
- Assumes [X] hours is acceptable latency for pipeline execution

---

## Next Steps

1. Review this plan with data architect
2. Prepare development environment (Phase 1)
3. Create schema definitions (Phase 2)
4. Begin data ingestion implementation (Phase 3)
5. Establish testing & monitoring framework
6. Plan production deployment strategy

---

## References

- [Specification Document](./spec-template.md)
- [Databricks Documentation](https://docs.databricks.com/)
- [Delta Lake Guide](https://docs.databricks.com/en/delta/index.html)
- [Databricks Asset Bundles](https://docs.databricks.com/en/dev-tools/bundles/index.html)
