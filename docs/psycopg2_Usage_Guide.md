# `psycopg2` Usage Guide

`psycopg2` is the most popular PostgreSQL database adapter for the Python programming language. It is the underlying driver that allows high-level libraries like `SQLAlchemy` and `ipython-sql` to communicate with a PostgreSQL database. Understanding `psycopg2` is useful for writing more complex Python applications that interact with PostgreSQL.

## 1. `psycopg2` vs. `psycopg2-binary`

When you install this library, you have two choices:

- `psycopg2`: This is the source distribution. It requires system-level build tools (like a C compiler and PostgreSQL development headers) to be installed on your machine, as it compiles the library from source. This can be difficult in some environments.
- `psycopg2-binary`: This is a pre-compiled binary version (a Python wheel). It requires no external dependencies and is installed easily via `pip`. 

**For most users, especially in environments like Google Colab, `psycopg2-binary` is the recommended choice.**

## 2. Installation

```python
!pip install psycopg2-binary
```

## 3. Basic Usage

The core workflow involves creating a connection, getting a cursor, executing a query, and fetching the results.

```python
import psycopg2

# --- 1. Connect to the database ---
# Use a try...finally block to ensure the connection is always closed
conn = None
try:
    conn = psycopg2.connect(
        host="localhost",
        database="chinook",
        user="postgres"
        # password="your_password" # Add if you have set a password
    )

    # --- 2. Create a cursor ---
    # Using a `with` statement ensures the cursor is properly closed
    with conn.cursor() as cur:
        # --- 3. Execute a query ---
        cur.execute("SELECT Name, Composer, Milliseconds FROM Track WHERE Composer LIKE %s;", ('%Bach%',))

        # --- 4. Fetch the results ---
        print("Tracks composed by Bach:")
        for record in cur.fetchall():
            name, composer, ms = record
            print(f"- {name} ({ms / 60000:.2f} mins)")

except (Exception, psycopg2.DatabaseError) as error:
    print(error)
finally:
    if conn is not None:
        conn.close()
```

## 4. Important Concepts

### Transactions (Commit and Rollback)

By default, `psycopg2` opens a transaction for every session. Any data-modifying commands (`INSERT`, `UPDATE`, `DELETE`) will not be saved permanently until you explicitly commit the transaction.

- `conn.commit()`: Saves all changes since the last commit.
- `conn.rollback()`: Discards all changes since the last commit.

```python
with conn.cursor() as cur:
    try:
        cur.execute("INSERT INTO system_logs (message) VALUES (%s);", ('A test message',))
        conn.commit() # <-- This makes the change permanent
    except Exception as e:
        print(f"An error occurred: {e}")
        conn.rollback() # <-- This undoes the change
```

For simple queries where you don't need fine-grained transaction control, you can enable `autocommit` mode:

```python
conn.autocommit = True
```

### Passing Parameters Safely (Preventing SQL Injection)

**NEVER** use Python f-strings or string formatting (`%`) to build queries with external data. This exposes your application to SQL injection attacks.

Always use the second argument of the `cursor.execute()` method to pass parameters. `psycopg2` will handle sanitizing the inputs.

```python
# WRONG - DO NOT DO THIS!
user_id = "1 OR 1=1; --"
cur.execute(f"SELECT * FROM users WHERE id = {user_id}")

# CORRECT - ALWAYS DO THIS
user_id = "1"
cur.execute("SELECT * FROM users WHERE id = %s;", (user_id,))
```

## 5. Conclusion

`psycopg2` provides a robust and efficient way to interact with a PostgreSQL database from Python. While you may often use it indirectly through higher-level libraries, understanding its basic principles of connections, cursors, and transaction management is fundamental for any developer working with Python and PostgreSQL.