# Choice of DB
## Best Choice: AdventureWorks

### Why It's Ideal for DS/DE Interviews:

**1. Realistic Complexity**
- Multiple schemas (Sales, Production, HR, Purchasing)
- 70+ tables with complex relationships
- Fact and dimension tables (data warehouse concepts)
- Temporal data for time-series analysis

**2. Interview-Relevant Scenarios**
```sql
-- Common interview patterns you can practice:

-- Window functions (ranking, running totals)
-- Customer segmentation/RFM analysis  
-- Sales trends and seasonality
-- Product recommendation queries
-- Employee hierarchy (recursive CTEs)
-- Inventory management
-- Customer churn analysis
```

**3. Data Engineering Topics**
- **Data modeling**: Star/snowflake schemas in AdventureWorksDW
- **ETL scenarios**: Raw OLTP → DW transformation practice
- **Data quality**: Nulls, duplicates, inconsistencies to clean
- **Performance**: Large enough for indexing/optimization practice

## Comparison for DS/DE Interviews

| Aspect | AdventureWorks | Sakila | Northwind | Chinook |
|--------|---------------|---------|-----------|----------|
| **Window Functions Practice** | Excellent | Good | Limited | Limited |
| **Time-Series Analysis** | Excellent | Good | Basic | Basic |
| **Complex JOINs** | Many scenarios | Moderate | Simple | Simple |
| **Data Volume** | Large enough | Medium | Too small | Too small |
| **Business Metrics** | Comprehensive | Limited | Basic | Very limited |
| **CTEs/Subqueries** | Complex cases | Moderate | Simple | Simple |
| **Data Warehouse Concepts** | Has DW version | No | No | No |

## Specific Interview Topics You Can Practice

### With AdventureWorks:
```sql
-- 1. Cohort Analysis
-- 2. Customer Lifetime Value
-- 3. Moving Averages/Growth Rates
-- 4. Funnel Analysis (quote→order→ship)
-- 5. AB Testing scenarios (compare store performance)
-- 6. Hierarchical queries (employee/product trees)
-- 7. Data anomaly detection
-- 8. Cross-selling analysis
```

# Choice of DBMS

## PostgreSQL is the Better Choice

### Key Advantages for DS/DE:

**1. Industry Alignment**
- Most modern data companies use PostgreSQL or PostgreSQL-compatible systems (Redshift, Greenplum)
- Cloud warehouses (Snowflake, BigQuery) have syntax closer to PostgreSQL
- Better represents what you'll use in actual DE roles

**2. Superior Analytical Features**
```sql
-- PostgreSQL-specific features crucial for DS/DE:

-- Better window functions
-- Array and JSON operations (common in modern data)
-- GENERATE_SERIES for date scaffolding
-- Better DISTINCT ON
-- Materialized views
-- Better EXPLAIN ANALYZE output
-- CTEs with INSERT/UPDATE/DELETE
```

**3. Both Databases Work Well**
- **AdventureWorks**: Has official PostgreSQL version
- **Sakila**: Use **Pagila** (PostgreSQL port of Sakila)

## Feature Comparison for DS/DE

| Feature | PostgreSQL | MySQL |
|---------|------------|--------|
| **Window Functions** | Excellent (all types) | Good (8.0+) |
| **CTEs** | Recursive + Writable | Recursive only |
| **JSON Support** | Native JSONB | Basic JSON |
| **Arrays** | Native support | No arrays |
| **EXPLAIN Plans** | Detailed | Basic |
| **Pivot Operations** | CROSSTAB extension | Manual only |
| **Date Functions** | Superior | Good |
| **Analytical Extensions** | Many (PostGIS, TimescaleDB) | Limited |

## Setup Instructions

### PostgreSQL Setup:
```bash
# 1. Install PostgreSQL
# macOS: brew install postgresql
# Ubuntu: sudo apt-get install postgresql

# 2. Load AdventureWorks
git clone https://github.com/lorint/AdventureWorks-for-Postgres
psql -c "CREATE DATABASE adventureworks"
psql -d adventureworks -f install.sql

# 3. Load Pagila (Sakila for PostgreSQL)
wget https://github.com/devrimgunduz/pagila/raw/master/pagila-schema.sql
wget https://github.com/devrimgunduz/pagila/raw/master/pagila-data.sql
psql -c "CREATE DATABASE pagila"
psql -d pagila -f pagila-schema.sql
psql -d pagila -f pagila-data.sql
```

### Alternative: Docker Setup (Easier)
```yaml
# docker-compose.yml
version: '3'
services:
  postgres:
    image: postgres:15
    environment:
      POSTGRES_PASSWORD: password
    ports:
      - "5432:5432"
    volumes:
      - ./data:/var/lib/postgresql/data
```

## Interview Relevance

### PostgreSQL-specific patterns to practice:
```sql
-- 1. Window functions with custom frames
SELECT *,
  AVG(amount) OVER (
    ORDER BY date 
    RANGE BETWEEN INTERVAL '30 days' PRECEDING 
    AND CURRENT ROW
  ) as rolling_30_day_avg

-- 2. Arrays for cohort analysis
SELECT 
  array_agg(DISTINCT user_id) as user_cohort,
  date_trunc('month', signup_date) as cohort_month

-- 3. JSONB for semi-structured data
SELECT 
  data->>'product' as product,
  jsonb_array_elements(data->'items')

-- 4. Better DISTINCT ON
SELECT DISTINCT ON (customer_id) 
  customer_id, order_date, amount
ORDER BY customer_id, order_date DESC
```

## Learning Path Recommendation

1. **Start with PostgreSQL** for both databases
2. **Learn PostgreSQL-specific features** that translate to cloud platforms
3. **Optional**: Later practice converting queries to MySQL syntax (good interview skill)

## Cloud Platform Alignment

| Platform | Similarity to PostgreSQL | Similarity to MySQL |
|----------|-------------------------|---------------------|
| **Redshift** | Very High | Low |
| **BigQuery** | High | Medium |
| **Snowflake** | High | Medium |
| **Databricks SQL** | High | Medium |

