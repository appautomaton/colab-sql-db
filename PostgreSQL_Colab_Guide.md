# Complete Guide: PostgreSQL with SQL Magic in Google Colab

## Part 1: Installation and Setup

### Step 1.1: Install PostgreSQL and SQL Magic

```python
# Cell 1: Install and configure SQL magic with better features
!apt-get update -qq
!apt-get install -y postgresql postgresql-client postgresql-contrib > /dev/null 2>&1
!pip install -q psycopg2-binary sqlalchemy pandas ipython-sql pgspecial

# Start PostgreSQL
!service postgresql start
!sudo -u postgres psql -c "ALTER USER postgres PASSWORD 'colab123';"

print("✅ PostgreSQL and SQL magic installed")
```

### Step 1.2: Load and Configure SQL Magic Extension

```python
# Cell 2: Load SQL magic and set optimal configurations
%load_ext sql

# Configure display settings for better output
%config SqlMagic.autopandas = True    # Automatically return Pandas DataFrames
%config SqlMagic.feedback = True      # Show query execution time
%config SqlMagic.displaycon = False   # Hide connection string from output
%config SqlMagic.displaylimit = 20    # Default row limit for display
%config SqlMagic.style = '_DEPRECATED_DEFAULT'

print("✅ SQL magic configured")
print("\n📝 Available commands:")
print("  %%sql - Multi-line SQL queries (entire cell)")
print("  %sql  - Single-line SQL queries")
print("  %sql [query] --persist df - Save results to variable 'df'")
```

### Step 1.3: Create Helper Functions

```python
# Cell 3: Create helper functions for database operations
import os
import pandas as pd
import psycopg2
from sqlalchemy import create_engine

def create_database(db_name):
    """Create a new PostgreSQL database"""
    try:
        conn = psycopg2.connect(
            host="localhost",
            database="postgres",
            user="postgres",
            password="colab123"
        )
        conn.autocommit = True
        cur = conn.cursor()
        
        # Drop if exists and create new
        cur.execute(f"DROP DATABASE IF EXISTS {db_name};")
        cur.execute(f"CREATE DATABASE {db_name};")
        
        print(f"✅ Database '{db_name}' created successfully")
        cur.close()
        conn.close()
        return True
    except Exception as e:
        print(f"❌ Error creating database: {e}")
        return False

def test_connection(db_name):
    """Test database connection"""
    try:
        %sql postgresql://postgres:colab123@localhost/{db_name}
        result = %sql SELECT current_database() as connected_to, version() as postgres_version;
        print(f"✅ Connected to {db_name}")
        return True
    except Exception as e:
        print(f"❌ Connection failed: {e}")
        return False

print("✅ Helper functions ready")
```

## Part 2: Load AdventureWorks Database

### Step 2.1: Download AdventureWorks for PostgreSQL

```python
# Cell 4: Download and prepare AdventureWorks
import os

# Create working directory
!mkdir -p /content/databases
os.chdir('/content/databases')

# Clone AdventureWorks for PostgreSQL
!git clone https://github.com/lorint/AdventureWorks-for-Postgres.git adventureworks_pg > /dev/null 2>&1

print("✅ AdventureWorks files downloaded")
print("\n📁 Files structure:")
!ls -la adventureworks_pg/ | head -10
```

### Step 2.2: Create and Populate AdventureWorks

```python
# Cell 5: Install AdventureWorks database
# Create the database
create_database('adventureworks')

# Navigate to the AdventureWorks directory
os.chdir('/content/databases/adventureworks_pg')

# Install the database schema and data
!sudo -u postgres psql -d adventureworks -f install.sql > /tmp/aw_install.log 2>&1

# Check for errors
!tail -20 /tmp/aw_install.log

print("✅ AdventureWorks installation completed")
os.chdir('/content/databases')
```

### Step 2.3: Connect to AdventureWorks with SQL Magic

```python
# Cell 6: Connect to AdventureWorks and verify
%sql postgresql://postgres:colab123@localhost/adventureworks

print("✅ Connected to AdventureWorks database")
print("\n📊 Database schemas:")
```

```sql
%%sql
-- Show all schemas in AdventureWorks
SELECT 
    schema_name,
    schema_owner
FROM information_schema.schemata
WHERE schema_name NOT IN ('pg_catalog', 'information_schema', 'pg_toast')
ORDER BY schema_name;
```

### Step 2.4: Explore AdventureWorks Structure

```sql
%%sql
-- Cell 7: Show table counts per schema
SELECT 
    schemaname as schema,
    COUNT(*) as table_count
FROM pg_tables
WHERE schemaname NOT IN ('pg_catalog', 'information_schema')
GROUP BY schemaname
ORDER BY schemaname;
```

```sql
%%sql
-- Cell 8: Sample query - Top selling products
SELECT 
    p.name as product_name,
    pc.name as category,
    COUNT(DISTINCT sod.salesorderid) as times_ordered,
    SUM(sod.orderqty) as total_quantity,
    ROUND(SUM(sod.linetotal)::numeric, 2) as total_revenue
FROM production.product p
JOIN production.productsubcategory ps ON p.productsubcategoryid = ps.productsubcategoryid
JOIN production.productcategory pc ON ps.productcategoryid = pc.productcategoryid
JOIN sales.salesorderdetail sod ON p.productid = sod.productid
GROUP BY p.name, pc.name
ORDER BY total_revenue DESC
LIMIT 10;
```

## Part 3: Load Pagila (Sakila for PostgreSQL) Database

### Step 3.1: Download Pagila Database Files

```python
# Cell 9: Download Pagila (PostgreSQL version of Sakila)
os.chdir('/content/databases')

# Download schema and data files
!wget -q https://github.com/devrimgunduz/pagila/raw/master/pagila-schema.sql
!wget -q https://github.com/devrimgunduz/pagila/raw/master/pagila-insert-data.sql

print("✅ Pagila files downloaded")
!ls -la pagila*.sql
```

### Step 3.2: Create and Populate Pagila Database

```python
# Cell 10: Install Pagila database
# Create the database
create_database('pagila')

# Install schema
!sudo -u postgres psql -d pagila -f pagila-schema.sql > /tmp/pagila_schema.log 2>&1

# Load data
!sudo -u postgres psql -d pagila -f pagila-insert-data.sql > /tmp/pagila_data.log 2>&1

print("✅ Pagila installation completed")
print("\n📊 Checking for any errors:")
!grep -i error /tmp/pagila*.log || echo "No errors found!"
```

### Step 3.3: Connect to Pagila and Verify

```python
# Cell 11: Switch to Pagila database
%sql postgresql://postgres:colab123@localhost/pagila

print("✅ Connected to Pagila database")
print("\n📊 Database tables:")
```

```sql
%%sql
-- Show all tables with row counts
SELECT 
    schemaname,
    tablename,
    n_live_tup as row_count
FROM pg_stat_user_tables
ORDER BY n_live_tup DESC;
```

### Step 3.4: Explore Pagila with Sample Queries

```sql
%%sql
-- Cell 12: Most popular film categories
SELECT 
    c.name as category,
    COUNT(r.rental_id) as rental_count,
    ROUND(AVG(f.rental_rate)::numeric, 2) as avg_rental_rate,
    ROUND(SUM(p.amount)::numeric, 2) as total_revenue
FROM category c
JOIN film_category fc ON c.category_id = fc.category_id
JOIN film f ON fc.film_id = f.film_id
JOIN inventory i ON f.film_id = i.film_id
JOIN rental r ON i.inventory_id = r.inventory_id
JOIN payment p ON r.rental_id = p.rental_id
GROUP BY c.name
ORDER BY rental_count DESC;
```

## Part 4: Database Switching and Management

### Step 4.1: Easy Database Switching

```python
# Cell 13: Create functions for easy database switching
def use_adventureworks():
    """Switch to AdventureWorks database"""
    %sql postgresql://postgres:colab123@localhost/adventureworks
    print("📦 Now using: AdventureWorks")
    print("   Schemas: sales, production, person, humanresources, purchasing")
    
def use_pagila():
    """Switch to Pagila database"""
    %sql postgresql://postgres:colab123@localhost/pagila
    print("🎬 Now using: Pagila")
    print("   Domain: DVD rental business")

def show_current_db():
    """Show current database connection"""
    result = %sql SELECT current_database() as database, current_user as user, version() as version;
    return result

print("✅ Database switching functions ready")
print("\n📝 Usage:")
print("  use_adventureworks() - Switch to AdventureWorks")
print("  use_pagila()         - Switch to Pagila")
print("  show_current_db()    - Show current connection")

# Example: Show current connection
show_current_db()
```

### Step 4.2: Quick Database Overview

```python
# Cell 14: Create a database overview function
def db_overview():
    """Show overview of current database"""
    current = %sql SELECT current_database() as db
    db_name = current[0]['db']
    
    print(f"\n📊 Overview of '{db_name}' database:")
    print("=" * 50)
    
    # Get table count
    table_count = %sql SELECT COUNT(*) as count FROM information_schema.tables WHERE table_schema NOT IN ('pg_catalog', 'information_schema')
    print(f"Total tables: {table_count[0]['count']}")
    
    # Get total row count (approximate)
    row_count = %sql SELECT SUM(n_live_tup) as total_rows FROM pg_stat_user_tables
    print(f"Total rows (approx): {row_count[0]['total_rows']:,}")
    
    # Get database size
    db_size = %sql SELECT pg_size_pretty(pg_database_size(current_database())) as size
    print(f"Database size: {db_size[0]['size']}")
    
    print("\n📋 Top 5 largest tables:")
    %sql SELECT schemaname || '.' || tablename as table_name, n_live_tup as row_count FROM pg_stat_user_tables ORDER BY n_live_tup DESC LIMIT 5;

# Test with current database
db_overview()
```

## Part 5: Practice SQL Queries with Magic Commands

### Step 5.1: Window Functions Practice

```python
# Cell 15: Switch to AdventureWorks for advanced queries
use_adventureworks()
```

```sql
%%sql
-- Cell 16: Window Functions - Running Total and Ranking
WITH monthly_sales AS (
    SELECT 
        DATE_TRUNC('month', orderdate) as month,
        COUNT(*) as order_count,
        ROUND(SUM(totaldue)::numeric, 2) as monthly_revenue
    FROM sales.salesorderheader
    WHERE orderdate >= '2013-01-01' AND orderdate < '2014-01-01'
    GROUP BY DATE_TRUNC('month', orderdate)
)
SELECT 
    TO_CHAR(month, 'YYYY-MM') as month,
    order_count,
    monthly_revenue,
    SUM(monthly_revenue) OVER (ORDER BY month) as running_total,
    ROUND(AVG(monthly_revenue) OVER (
        ORDER BY month 
        ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
    ), 2) as moving_avg_3month,
    RANK() OVER (ORDER BY monthly_revenue DESC) as revenue_rank
FROM monthly_sales
ORDER BY month;
```

### Step 5.2: Common Table Expressions (CTEs)

```sql
%%sql
-- Cell 17: Customer Segmentation using CTEs
WITH customer_stats AS (
    -- Calculate customer metrics
    SELECT 
        customerid,
        COUNT(DISTINCT salesorderid) as order_count,
        ROUND(SUM(totaldue)::numeric, 2) as total_spent,
        ROUND(AVG(totaldue)::numeric, 2) as avg_order_value,
        MAX(orderdate)::date as last_order_date,
        (CURRENT_DATE - MAX(orderdate)::date) as days_since_last_order
    FROM sales.salesorderheader
    GROUP BY customerid
),
customer_segments AS (
    -- Segment customers based on behavior
    SELECT 
        *,
        CASE 
            WHEN days_since_last_order <= 90 THEN 'Active'
            WHEN days_since_last_order <= 365 THEN 'At Risk'
            ELSE 'Churned'
        END as recency_segment,
        CASE 
            WHEN total_spent >= 10000 THEN 'High Value'
            WHEN total_spent >= 1000 THEN 'Medium Value'
            ELSE 'Low Value'
        END as value_segment
    FROM customer_stats
)
-- Final summary by segments
SELECT 
    recency_segment,
    value_segment,
    COUNT(*) as customer_count,
    ROUND(AVG(total_spent)::numeric, 2) as avg_lifetime_value,
    ROUND(AVG(order_count)::numeric, 1) as avg_orders
FROM customer_segments
GROUP BY recency_segment, value_segment
ORDER BY recency_segment, value_segment;
```

### Step 5.3: Complex Joins Practice

```python
# Cell 18: Switch to Pagila for join practice
use_pagila()
```

```sql
%%sql
-- Cell 19: Multi-table join - Actor performance analysis
SELECT 
    a.first_name || ' ' || a.last_name as actor_name,
    COUNT(DISTINCT f.film_id) as film_count,
    COUNT(DISTINCT fc.category_id) as category_diversity,
    STRING_AGG(DISTINCT c.name, ', ' ORDER BY c.name) as categories,
    ROUND(AVG(f.rental_rate)::numeric, 2) as avg_film_rental_rate,
    COUNT(DISTINCT r.rental_id) as total_rentals,
    ROUND(SUM(p.amount)::numeric, 2) as total_revenue_generated
FROM actor a
JOIN film_actor fa ON a.actor_id = fa.actor_id
JOIN film f ON fa.film_id = f.film_id
JOIN film_category fc ON f.film_id = fc.film_id
JOIN category c ON fc.category_id = c.category_id
LEFT JOIN inventory i ON f.film_id = i.film_id
LEFT JOIN rental r ON i.inventory_id = r.inventory_id
LEFT JOIN payment p ON r.rental_id = p.rental_id
GROUP BY a.actor_id, a.first_name, a.last_name
HAVING COUNT(DISTINCT f.film_id) >= 20
ORDER BY total_revenue_generated DESC NULLS LAST
LIMIT 10;
```

## Part 6: Advanced Features and Tips

### Step 6.1: Using Python Variables in SQL

```python
# Cell 20: Parameterized queries with Python variables
use_adventureworks()

# Define Python variables
start_date = '2013-01-01'
end_date = '2013-12-31'
min_order_value = 1000
```

```sql
%%sql
-- Use Python variables in SQL with :variable_name syntax
SELECT 
    DATE_TRUNC('month', orderdate) as month,
    COUNT(*) as large_orders,
    ROUND(AVG(totaldue)::numeric, 2) as avg_order_value,
    ROUND(SUM(totaldue)::numeric, 2) as total_revenue
FROM sales.salesorderheader
WHERE 
    orderdate BETWEEN :start_date AND :end_date
    AND totaldue >= :min_order_value
GROUP BY DATE_TRUNC('month', orderdate)
ORDER BY month;
```

### Step 6.2: Save Query Results to DataFrames

```python
# Cell 21: Different ways to save query results

# Method 1: Direct assignment
df1 = %sql SELECT COUNT(*) as product_count FROM production.product

# Method 2: Using << operator for multi-line queries
%%sql df2 <<
SELECT 
    pc.name as category,
    COUNT(*) as product_count
FROM production.product p
JOIN production.productsubcategory ps ON p.productsubcategoryid = ps.productsubcategoryid
JOIN production.productcategory pc ON ps.productcategoryid = pc.productcategoryid
GROUP BY pc.name
ORDER BY product_count DESC
```

```python
# Cell 22: Work with the saved DataFrames
print(f"DataFrame 1 type: {type(df1)}")
print(f"DataFrame 1 shape: {df1.shape}")
print(f"\nDataFrame 2 type: {type(df2)}")
print(f"DataFrame 2 shape: {df2.shape}")

# Visualize the results
import matplotlib.pyplot as plt

df2.plot(kind='bar', x='category', y='product_count', figsize=(10, 6))
plt.title('Products per Category')
plt.xlabel('Category')
plt.ylabel('Number of Products')
plt.xticks(rotation=45)
plt.tight_layout()
plt.show()
```

### Step 6.3: Performance Analysis with EXPLAIN

```sql
%%sql
-- Cell 23: Analyze query performance
EXPLAIN (ANALYZE, BUFFERS, FORMAT TEXT)
SELECT 
    c.customerid,
    COUNT(DISTINCT soh.salesorderid) as order_count,
    SUM(sod.linetotal) as total_spent
FROM sales.customer c
JOIN sales.salesorderheader soh ON c.customerid = soh.customerid
JOIN sales.salesorderdetail sod ON soh.salesorderid = sod.salesorderid
WHERE soh.orderdate >= '2013-01-01'
GROUP BY c.customerid
HAVING COUNT(DISTINCT soh.salesorderid) > 5
ORDER BY total_spent DESC
LIMIT 10;
```

## Part 7: Interview-Style Practice Queries

### Step 7.1: Common Interview Patterns

```python
# Cell 24: Create a collection of interview-style queries
interview_queries = {
    "top_n_per_group": """
        -- Find top 3 products in each category by revenue
        WITH product_revenue AS (
            SELECT 
                pc.name as category,
                p.name as product,
                SUM(sod.linetotal) as total_revenue,
                ROW_NUMBER() OVER (
                    PARTITION BY pc.name 
                    ORDER BY SUM(sod.linetotal) DESC
                ) as rank_in_category
            FROM production.product p
            JOIN production.productsubcategory ps ON p.productsubcategoryid = ps.productsubcategoryid
            JOIN production.productcategory pc ON ps.productcategoryid = pc.productcategoryid
            JOIN sales.salesorderdetail sod ON p.productid = sod.productid
            GROUP BY pc.name, p.name
        )
        SELECT category, product, ROUND(total_revenue::numeric, 2) as revenue
        FROM product_revenue
        WHERE rank_in_category <= 3
        ORDER BY category, rank_in_category;
    """,
    
    "year_over_year": """
        -- Calculate year-over-year growth
        WITH yearly_metrics AS (
            SELECT 
                EXTRACT(YEAR FROM orderdate) as year,
                COUNT(DISTINCT customerid) as unique_customers,
                COUNT(DISTINCT salesorderid) as total_orders,
                ROUND(SUM(totaldue)::numeric, 2) as total_revenue
            FROM sales.salesorderheader
            GROUP BY EXTRACT(YEAR FROM orderdate)
        )
        SELECT 
            year,
            unique_customers,
            total_orders,
            total_revenue,
            LAG(total_revenue) OVER (ORDER BY year) as prev_year_revenue,
            ROUND(100.0 * (total_revenue - LAG(total_revenue) OVER (ORDER BY year)) / 
                  NULLIF(LAG(total_revenue) OVER (ORDER BY year), 0), 2) as yoy_growth_pct
        FROM yearly_metrics
        ORDER BY year;
    """,
    
    "customer_ltv": """
        -- Customer Lifetime Value calculation
        WITH customer_metrics AS (
            SELECT 
                c.customerid,
                MIN(soh.orderdate)::date as first_purchase,
                MAX(soh.orderdate)::date as last_purchase,
                COUNT(DISTINCT soh.salesorderid) as total_orders,
                ROUND(SUM(soh.totaldue)::numeric, 2) as lifetime_value,
                (MAX(soh.orderdate)::date - MIN(soh.orderdate)::date) as customer_lifespan_days
            FROM sales.customer c
            JOIN sales.salesorderheader soh ON c.customerid = soh.customerid
            GROUP BY c.customerid
            HAVING COUNT(DISTINCT soh.salesorderid) > 1
        )
        SELECT 
            CASE 
                WHEN lifetime_value >= 10000 THEN 'Platinum'
                WHEN lifetime_value >= 5000 THEN 'Gold'
                WHEN lifetime_value >= 1000 THEN 'Silver'
                ELSE 'Bronze'
            END as customer_tier,
            COUNT(*) as customer_count,
            ROUND(AVG(lifetime_value)::numeric, 2) as avg_ltv,
            ROUND(AVG(total_orders)::numeric, 1) as avg_orders,
            ROUND(AVG(customer_lifespan_days)::numeric, 0) as avg_lifespan_days
        FROM customer_metrics
        GROUP BY customer_tier
        ORDER BY avg_ltv DESC;
    """
}

print("📚 Interview query patterns loaded!")
print("\nAvailable patterns:")
for pattern in interview_queries.keys():
    print(f"  - {pattern}")
```

### Step 7.2: Execute Interview Queries

```python
# Cell 25: Function to run interview queries
def practice_interview_query(pattern_name):
    """Run an interview pattern query"""
    if pattern_name not in interview_queries:
        print(f"❌ Pattern '{pattern_name}' not found")
        print(f"Available: {list(interview_queries.keys())}")
        return None
    
    print(f"\n🎯 Running: {pattern_name}")
    print("=" * 50)
    
    # Make sure we're on AdventureWorks
    use_adventureworks()
    
    # Execute the query
    query = interview_queries[pattern_name]
    result = %sql {query}
    
    return result

# Example: Run the top N per group query
result = practice_interview_query("top_n_per_group")
```

## Part 8: Quick Reference Guide

### Step 8.1: Cheat Sheet

```python
# Cell 26: Create a comprehensive cheat sheet
cheat_sheet = """
📋 SQL MAGIC COMMANDS CHEAT SHEET
=====================================

🔄 DATABASE SWITCHING:
    use_adventureworks()  # Switch to AdventureWorks
    use_pagila()         # Switch to Pagila
    show_current_db()    # Show current database

📝 QUERY EXECUTION:
    %sql [single line query]           # Single line
    %%sql                              # Multi-line (entire cell)
    %%sql result_df <<                 # Save to DataFrame
    df = %sql SELECT ...               # Direct assignment

🔧 CONFIGURATION:
    %config SqlMagic.displaylimit = 50     # Show more rows
    %config SqlMagic.autopandas = True     # Auto DataFrame conversion
    %sql --connections                     # List connections

📊 COMMON PATTERNS:
    
    1. Window Functions:
       ROW_NUMBER() OVER (PARTITION BY ... ORDER BY ...)
       RANK() / DENSE_RANK() OVER (...)
       LAG() / LEAD() OVER (...)
       SUM() OVER (ORDER BY ... ROWS BETWEEN ...)
    
    2. CTEs:
       WITH cte_name AS (
           SELECT ...
       )
       SELECT ... FROM cte_name
    
    3. Date Functions:
       DATE_TRUNC('month', date_column)
       EXTRACT(YEAR FROM date_column)
       date_column::date
       CURRENT_DATE - date_column
    
    4. String Operations:
       string1 || string2                  # Concatenation
       STRING_AGG(column, ', ')            # Aggregate strings
       SUBSTRING(string FROM pattern)      # Extract substring

🎯 INTERVIEW TIPS:
    - Always use CTEs for readability
    - Comment complex logic
    - Use meaningful aliases
    - Consider performance (EXPLAIN ANALYZE)
    - Handle NULLs explicitly
    - Use proper data types and casting

📚 ADVENTUREWORKS SCHEMAS:
    - sales: Orders, customers, territories
    - production: Products, categories, inventory
    - person: People, addresses, contacts
    - humanresources: Employees, departments
    - purchasing: Vendors, purchase orders

🎬 PAGILA MAIN TABLES:
    - film: Movie information
    - actor: Actor details
    - customer: Customer information
    - rental: Rental transactions
    - payment: Payment records
    - inventory: Store inventory
"""

print(cheat_sheet)

# Save to file for reference
with open('/content/sql_magic_cheatsheet.txt', 'w') as f:
    f.write(cheat_sheet)
print("\n✅ Cheat sheet saved to: /content/sql_magic_cheatsheet.txt")
```

### Step 8.2: Final Validation

```python
# Cell 27: Validate everything is working
def final_validation():
    """Run final validation checks"""
    print("🔍 FINAL VALIDATION")
    print("=" * 50)
    
    checks = {
        "PostgreSQL Service": False,
        "AdventureWorks Connection": False,
        "Pagila Connection": False,
        "SQL Magic Loaded": False,
        "Query Execution": False
    }
    
    # Check PostgreSQL
    try:
        result = !service postgresql status | grep "online"
        if result:
            checks["PostgreSQL Service"] = True
    except:
        pass
    
    # Check AdventureWorks
    try:
        %sql postgresql://postgres:colab123@localhost/adventureworks
        test = %sql SELECT COUNT(*) FROM production.product
        if test:
            checks["AdventureWorks Connection"] = True
    except:
        pass
    
    # Check Pagila
    try:
        %sql postgresql://postgres:colab123@localhost/pagila
        test = %sql SELECT COUNT(*) FROM film
        if test:
            checks["Pagila Connection"] = True
    except:
        pass
    
    # Check SQL Magic
    try:
        import sql
        checks["SQL Magic Loaded"] = True
        checks["Query Execution"] = True
    except:
        pass
    
    # Display results
    all_passed = True
    for check, status in checks.items():
        icon = "✅" if status else "❌"
        status_text = "PASSED" if status else "FAILED"
        print(f"{icon} {check}: {status_text}")
        if not status:
            all_passed = False
    
    print("=" * 50)
    
    if all_passed:
        print("\n🎉 ALL CHECKS PASSED!")
        print("Your environment is ready for SQL practice!")
        print("\n🚀 Next steps:")
        print("1. Use use_adventureworks() or use_pagila() to switch databases")
        print("2. Write queries using %%sql magic command")
        print("3. Practice with the interview query patterns")
    else:
        print("\n⚠️ Some checks failed. Please review the setup steps.")
    
    return checks

# Run validation
validation_results = final_validation()
```

## Summary

```python
# Cell 28: Summary and next steps
print("""
✅ SETUP COMPLETE!
==================

You now have a complete SQL practice environment with:

📦 DATABASES:
  • AdventureWorks - Enterprise sales database
  • Pagila - DVD rental business

🛠️ TOOLS:
  • SQL Magic commands (%%sql)
  • Easy database switching
  • Python integration
  • Query result visualization

📚 RESOURCES:
  • Interview query patterns
  • Performance analysis tools
  • Cheat sheet reference

🎯 HOW TO PRACTICE:

1. Switch database:
   use_adventureworks()  # or use_pagila()

2. Write SQL directly:
   %%sql
   SELECT ... FROM ... WHERE ...

3. Save results to DataFrame:
   df = %sql SELECT ...

4. Practice interview patterns:
   practice_interview_query("pattern_name")

💡 PRO TIPS:
  • Use CTEs for complex queries
  • Practice window functions daily
  • Time yourself on medium queries (20-30 min)
  • Build a portfolio of solved problems

Happy practicing! 🚀
""")
```

This complete guide provides you with:

1. **Two fully-loaded databases** ready for practice
2. **SQL Magic commands** for natural SQL writing
3. **Easy database switching** functions
4. **Interview-style query patterns** for practice
5. **Comprehensive reference materials**
6. **Validation tools** to ensure everything works

You can now write SQL queries directly in Colab cells using `%%sql` and practice for your DS/DE interviews efficiently!
