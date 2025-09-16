# `ipython-sql` Usage Guide

`ipython-sql` is a library that allows you to write SQL queries directly in IPython or Jupyter/Colab notebooks. It provides "magic commands" that make interacting with relational databases seamless within a data analysis workflow.

## 1. Installation

To use `ipython-sql`, you first need to install it via pip. You will also need a "driver" for the specific database you want to connect to. For PostgreSQL, this is `psycopg2-binary`.

```python
!pip install ipython-sql psycopg2-binary
```

## 2. Core Concepts

### Loading the Extension

Before you can use the magic commands, you must load the `sql` extension in your notebook.

```python
%load_ext sql
```

### Connecting to a Database

You connect to your database using a standard database URL format. The connection string is passed to the `%sql` magic command.

```python
# Format: dialect+driver://username:password@host:port/database

# Example for PostgreSQL:
%sql postgresql://postgres@localhost/chinook

# Example for SQLite (in-memory):
# %sql sqlite://

# Example for SQLite (file-based):
# %sql sqlite:///path/to/my_database.db
```

Once connected, you can run queries.

### Querying

**Single-Line Queries**

For queries that fit on a single line, use the `%sql` command:

```python
%sql SELECT * FROM Artist ORDER BY Name LIMIT 5;
```

**Multi-Line Queries**

For longer, more complex queries, start a cell with `%%sql`. This turns the entire cell into an SQL editor.

```python
%%sql
SELECT
    a.Title AS Album,
    t.Name AS Track,
    t.Milliseconds
FROM Album a
JOIN Track t ON a.AlbumId = t.AlbumId
WHERE a.ArtistId = (SELECT ArtistId FROM Artist WHERE Name = 'Queen')
ORDER BY a.Title, t.TrackId;
```

## 3. Advanced Usage

### Saving Results to a Variable

You can assign the results of a query to a Python variable.

```python
my_artists = %sql SELECT Name FROM Artist WHERE Name LIKE 'A%';

print(my_artists)
```

### Converting to a Pandas DataFrame

This is one of the most powerful features of `ipython-sql`. Query results can be easily converted into a Pandas DataFrame for further analysis.

```python
import pandas as pd

# Run the query and get the results
results = %sql SELECT * FROM Invoice WHERE Total > 15;

# Convert to a DataFrame
df = results.DataFrame()

df.info()
```

### Using Python Variables in Queries (Bind Variables)

You can pass Python variables into your SQL queries to make them dynamic. Use a colon (`:`) before the variable name.

```python
country = 'Brazil'

brazil_customers = %sql SELECT * FROM Customer WHERE Country = :country;

print(brazil_customers)
```

## 4. Conclusion

`ipython-sql` is an essential tool for anyone doing data analysis with SQL in a notebook environment. It bridges the gap between SQL and Python, allowing you to seamlessly query databases and bring the results into a familiar Pandas workflow.