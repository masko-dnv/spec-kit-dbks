# Data Model: [PIPELINE_NAME]

**Version**: 1.0  
**Last Updated**: [Date]  
**Owner**: [Data Engineer / Team]

---

## Overview

This document defines the complete data schema for the [Pipeline Name] pipeline, including all Bronze, Silver, and Gold layer tables, column definitions, data types, constraints, and transformation rules.

---

## Bronze Layer (Raw Data)

### Table: `[catalog].bronze.raw_[source_1]`

**Description**: Raw data extracted directly from [Source System Name]. No transformations applied.

**Update Frequency**: [Daily / Hourly / Real-time]  
**Retention**: [e.g., "90 days of history"]  
**Load Pattern**: Append / Overwrite / Merge

| Column Name | Data Type | Nullable | Description | Source Field |
|-------------|-----------|----------|-------------|--------------|
| `[column_1]` | string | No | [Description] | `source.[field]` |
| `[column_2]` | int | No | [Description] | `source.[field]` |
| `[column_3]` | decimal(18,2) | Yes | [Description] | `source.[field]` |
| `[column_4]` | timestamp | No | [Description] | `source.[field]` |
| `ingested_at` | timestamp | No | Timestamp when record was ingested | Current timestamp |
| `source_system` | string | No | Source system identifier | Hardcoded: `[SOURCE_NAME]` |

**Data Quality Checks**:
- `[column_1]` MUST NOT be NULL
- `[column_2]` MUST be >= 0
- `[column_3]` MUST be within range [min, max]
- `ingested_at` MUST be within last 24 hours

**Sample Query**:
```sql
SELECT COUNT(*) as row_count,
       COUNT(DISTINCT [column_1]) as unique_values,
       MIN(ingested_at) as earliest_ingest,
       MAX(ingested_at) as latest_ingest
FROM [catalog].bronze.raw_[source_1]
WHERE DATE(ingested_at) = CURRENT_DATE()
```

---

### Table: `[catalog].bronze.raw_[source_2]`

[Repeat pattern above for each source]

---

## Silver Layer (Cleaned & Deduplicated Data)

### Table: `[catalog].silver.transformed_[entity_1]`

**Description**: Deduplicated, cleaned version of [Entity] data. Business rules applied, null handling done, duplicates removed.

**Source Tables**: 
- `bronze.raw_[source_1]`
- `bronze.raw_[source_2]` (if applicable)

**Transformation Logic**:
1. Deduplicate on primary key: `[column_1, column_2]`
2. Filter out records with `is_deleted = true`
3. Apply business rule transformations (see below)
4. Validate data quality constraints

**Primary Key**: `[column_1, column_2]`  
**Partitioning**: [By date / By region / None]  
**Update Frequency**: Daily  
**Retention**: 365 days

| Column Name | Data Type | Nullable | Description | Transformation |
|-------------|-----------|----------|-------------|-----------------|
| `[column_1]` | string | No | [Description] | Deduplicate key |
| `[column_2]` | string | No | [Description] | Deduplicate key |
| `[column_3]` | decimal(18,2) | No | [Description] | COALESCE from sources, convert to numeric |
| `[column_4]` | string | Yes | [Description] | TRIM and LOWER if applicable |
| `[column_5]` | date | No | [Description] | CAST to date, validate range |
| `[column_6]` | boolean | No | [Description] | Derived: IF [condition] THEN true ELSE false |
| `full_name` | string | No | Computed field | CONCAT([first_name], ' ', [last_name]) |
| `transformed_at` | timestamp | No | Transformation timestamp | Current timestamp |

**Data Quality Checks**:
- No duplicate records (deduplicated successfully)
- Null rate in critical columns < 0.1%
- All required fields populated
- Data type consistency
- Business rule validation

**Sample Query**:
```sql
SELECT 
    [column_1],
    [column_2],
    COUNT(*) as duplicate_count
FROM [catalog].silver.transformed_[entity_1]
GROUP BY [column_1], [column_2]
HAVING COUNT(*) > 1
```

**SQL DDL**:
```sql
CREATE TABLE IF NOT EXISTS [catalog].silver.transformed_[entity_1] (
    [column_1] STRING NOT NULL,
    [column_2] STRING NOT NULL,
    [column_3] DECIMAL(18, 2) NOT NULL,
    [column_4] STRING,
    [column_5] DATE NOT NULL,
    [column_6] BOOLEAN NOT NULL,
    full_name STRING NOT NULL,
    transformed_at TIMESTAMP NOT NULL,
    CONSTRAINT pk_[entity_1] PRIMARY KEY ([column_1], [column_2])
)
USING DELTA
PARTITIONED BY ([column_5])
```

---

### Table: `[catalog].silver.transformed_[entity_2]`

[Repeat pattern above for each entity]

---

## Gold Layer (Business-Ready Data)

### Table: `[catalog].gold.agg_[metric_1]`

**Description**: Daily aggregated metrics for [Business Use Case]. Ready for reporting, BI tools, and ML models.

**Source Tables**:
- `silver.transformed_[entity_1]`
- `silver.transformed_[entity_2]` (if applicable)

**Business Logic**:
- Group by: `[dimension_1], [dimension_2], date`
- Aggregate: SUM([metric_1]), AVG([metric_2]), COUNT(DISTINCT [dimension_3])
- Filter: Only include records from last 90 days
- Validation: Aggregates must match source totals within 0.1%

**Primary Key**: `date, [dimension_1], [dimension_2]`  
**Partitioning**: By `date`  
**Update Frequency**: Daily (after Silver layer complete)  
**Retention**: 730 days (2 years)

| Column Name | Data Type | Nullable | Description | Formula |
|-------------|-----------|----------|-------------|---------|
| `date` | date | No | Metric date | Source date |
| `[dimension_1]` | string | No | [Dimension description] | GROUP BY |
| `[dimension_2]` | string | No | [Dimension description] | GROUP BY |
| `[metric_1]` | long | No | [Metric description] | SUM([column_3]) |
| `[metric_2]` | decimal(18,2) | No | [Metric description] | AVG([column_3]) |
| `[metric_3]` | int | No | [Metric description] | COUNT(DISTINCT [dimension_3]) |
| `record_count` | int | No | Number of records aggregated | COUNT(*) |
| `last_updated` | timestamp | No | When aggregation was computed | Current timestamp |

**Data Quality Checks**:
- Aggregate totals match source tables within 0.1%
- No missing dates in time series (all days present)
- All required dimensions have values
- Metric values within expected ranges

**Sample Query**:
```sql
-- Validation: Compare Gold aggregates to Silver source
SELECT 
    'gold' as source,
    COUNT(*) as record_count,
    SUM([metric_1]) as total_metric_1,
    AVG([metric_2]) as avg_metric_2
FROM [catalog].gold.agg_[metric_1]
WHERE date = CURRENT_DATE() - 1

UNION ALL

SELECT 
    'silver' as source,
    COUNT(*) as record_count,
    SUM([column_3]) as total_metric_1,
    AVG([column_3]) as avg_metric_2
FROM [catalog].silver.transformed_[entity_1]
WHERE DATE(transformed_at) = CURRENT_DATE() - 1
```

**SQL DDL**:
```sql
CREATE TABLE IF NOT EXISTS [catalog].gold.agg_[metric_1] (
    date DATE NOT NULL,
    [dimension_1] STRING NOT NULL,
    [dimension_2] STRING NOT NULL,
    [metric_1] LONG NOT NULL,
    [metric_2] DECIMAL(18, 2) NOT NULL,
    [metric_3] INT NOT NULL,
    record_count INT NOT NULL,
    last_updated TIMESTAMP NOT NULL,
    CONSTRAINT pk_agg_[metric_1] PRIMARY KEY (date, [dimension_1], [dimension_2])
)
USING DELTA
PARTITIONED BY (date)
```

---

### Table: `[catalog].gold.agg_[metric_2]`

[Repeat pattern above for each business metric]

---

## Data Lineage & Dependencies

```
bronze.raw_[source_1]
    ↓
silver.transformed_[entity_1]
    ↓
gold.agg_[metric_1]

bronze.raw_[source_2]
    ↓
silver.transformed_[entity_2]
    ↓
gold.agg_[metric_1] (joins with entity_1)
gold.agg_[metric_2]
```

---

## Transformation Rules & Business Logic

### Deduplication Rules

**Primary Key**: `[column_1, column_2]`

When duplicates are detected:
- Keep the most recent record (by `ingested_at`)
- If timestamps are identical, keep the one with highest `[column_3]` value
- Log duplicates for investigation

```python
def deduplicate(df):
    from pyspark.sql.functions import row_number, desc
    from pyspark.sql.window import Window
    
    window = Window.partitionBy("column_1", "column_2").orderBy(
        desc("ingested_at"), 
        desc("column_3")
    )
    
    return df.withColumn("rn", row_number().over(window)) \
        .filter("rn = 1") \
        .drop("rn")
```

### Data Type Conversions

| Source Type | Target Type | Logic |
|------------|------------|-------|
| string | int | CAST([column] AS INT), validate >= 0 |
| string | decimal | CAST([column] AS DECIMAL(18,2)), validate range |
| string | date | CAST([column] AS DATE), validate in valid range |
| int | string | CAST([column] AS STRING), TRIM |

### Derived Columns

**full_name** (Silver layer):
```sql
CONCAT(TRIM(first_name), ' ', TRIM(last_name))
```

**[custom_metric]** (Gold layer):
```sql
CASE 
    WHEN [column_1] > threshold THEN 'high'
    WHEN [column_1] > threshold/2 THEN 'medium'
    ELSE 'low'
END
```

---

## Historical Data Retention

| Layer | Retention Period | Rationale |
|-------|-----------------|-----------|
| Bronze | 90 days | Raw data is large; keep for audit trail |
| Silver | 365 days | Cleaned data; useful for historical analysis |
| Gold | 730 days | Business metrics; needed for year-over-year analysis |

---

## Access Control (Unity Catalog)

```sql
-- Grant read access to analysts
GRANT SELECT ON TABLE [catalog].gold.agg_[metric_1] 
TO `analysts@company.com`

-- Grant read/write to data engineers
GRANT SELECT, MODIFY ON TABLE [catalog].silver.transformed_[entity_1] 
TO `data-engineers@company.com`

-- Restrict Bronze layer to data engineers only
GRANT SELECT ON TABLE [catalog].bronze.raw_[source_1] 
TO `data-engineers@company.com`
```

---

## Performance Considerations

### Partitioning Strategy

- **Bronze**: Partition by `ingested_date` to prune old data quickly
- **Silver**: Partition by business date to align with reporting periods
- **Gold**: Partition by `date` for monthly/yearly queries

### Clustering Strategy

For tables > 1GB:
- **Silver**: Cluster on deduplicate keys `[column_1, column_2]`
- **Gold**: Cluster on `[dimension_1]` (high-cardinality dimension)

### Z-Ordering

For columnar optimization (optional, for tables > 10GB):
```sql
-- Optimize table for queries on dimension_1 and date
OPTIMIZE [catalog].gold.agg_[metric_1]
ZORDER BY [dimension_1], date
```

---

## Testing & Validation

### Unit Tests

```python
# tests/unit/test_schema.py

def test_silver_deduplication():
    """Ensure duplicates are removed correctly"""
    input_df = spark.createDataFrame([
        ("A", "1", 100),
        ("A", "1", 200),  # Duplicate on PK, keep this one (higher value)
        ("B", "2", 150),
    ], schema="col1 STRING, col2 STRING, col3 INT")
    
    result = deduplicate(input_df)
    assert result.count() == 2
    assert result.filter("col1='A' AND col3=200").count() == 1

def test_gold_aggregation():
    """Ensure aggregations match source data"""
    source_total = 1000
    agg_total = calculate_aggregation(...)
    assert abs(source_total - agg_total) / source_total < 0.001
```

---

## References

- [Databricks Delta Guide](https://docs.databricks.com/en/delta/index.html)
- [Unity Catalog Best Practices](https://docs.databricks.com/en/data-governance/unity-catalog/index.html)
- [Specification Document](./spec-template.md)
