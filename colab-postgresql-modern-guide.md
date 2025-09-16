# PostgreSQL + JupySQL in Google Colab (2025 Refresh)

This guide replaces the legacy `ipython-sql` workflow with the actively maintained [JupySQL](https://jupysql.ploomber.io/) extension, provides a repeatable way to install PostgreSQL 14 (default) or PostgreSQL 16 on fresh Colab runtimes, and fixes the data-loading issues in the original `colab-postgresql-guide.md`.

Each section maps to a single Colab cell so you can copy/paste sequentially without manual edits.

---

## 0. Runtime assumptions
- Colab currently runs Ubuntu 22.04 LTS; `apt-get install postgresql` yields PostgreSQL 14.x from the Ubuntu repo.
- All commands below expect a brand-new runtime. If you restart the runtime you must rerun cells 1–5.
- Replace `colab_secure_pw` with your own superuser password if you need something stronger.

---

## 1. System packages (PostgreSQL 14 by default)
```bash
# Cell 1 — OS packages (PostgreSQL 14)
!sudo apt-get update -qq
!sudo apt-get install -y postgresql postgresql-client postgresql-contrib > /dev/null
```

### Optional: install PostgreSQL 16 from PGDG
Run **instead** of the cell above if you want PostgreSQL 16 features.
```bash
# Cell 1 (variant) — add PGDG repo and install PostgreSQL 16
!sudo apt-get update -qq
!sudo apt-get install -y wget ca-certificates gnupg > /dev/null
!wget -qO- https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo gpg --dearmor -o /usr/share/keyrings/pgdg.gpg
!echo "deb [signed-by=/usr/share/keyrings/pgdg.gpg] http://apt.postgresql.org/pub/repos/apt jammy-pgdg main" | sudo tee /etc/apt/sources.list.d/pgdg.list > /dev/null
!sudo apt-get update -qq
!sudo apt-get install -y postgresql-16 postgresql-client-16 postgresql-contrib-16 > /dev/null
```

> **Tip:** After installation you can confirm the server version with `!psql --version`.

---

## 2. Start service and set password
```bash
# Cell 2 — start PostgreSQL and set a password
!sudo service postgresql start
!sudo -u postgres psql -c "ALTER USER postgres PASSWORD 'colab_secure_pw';" > /dev/null
```

---

## 3. Python packages
```python
# Cell 3 — install Python dependencies
%pip install --quiet --upgrade pip
%pip install --quiet jupysql psycopg2-binary sqlalchemy pandas pgspecial
```

---

## 4. Load and configure JupySQL safely
```python
# Cell 4 — load JupySQL and apply common settings
from pathlib import Path

def init_sql_magic(style: str | None = None):
    ip = get_ipython()
    ip.run_line_magic("load_ext", "sql")
    ip.run_line_magic("config", "SqlMagic.autopandas=True")
    ip.run_line_magic("config", "SqlMagic.feedback=True")
    ip.run_line_magic("config", "SqlMagic.displaycon=False")
    ip.run_line_magic("config", "SqlMagic.displaylimit=20")
    if style:
        ip.run_line_magic("config", f"SqlMagic.style={style}")

init_sql_magic(style="_DEPRECATED_DEFAULT")  # remove style arg if you prefer default look
```

> Why the helper? IPython magics like `%sql` cannot appear literally inside `def` blocks. Using `get_ipython().run_line_magic` avoids the `SyntaxError` that the original guide triggered.

---

## 5. Connect with JupySQL
```python
# Cell 5 — open a connection helper
from urllib.parse import quote_plus

PASSWORD = quote_plus("colab_secure_pw")
CONN = f"postgresql://postgres:{PASSWORD}@localhost/postgres"

get_ipython().run_line_magic("sql", CONN)
```

- Use `%sql` for single-line commands and `%%sql` for multi-line cells.
- To run a query and capture the result as a DataFrame: `df = %sql SELECT 1;`
- To push an existing DataFrame into PostgreSQL use `--persist`, e.g. `%sql --persist df_local`

---

## 6. Create practice databases
```python
# Cell 6 — helper functions for database lifecycle
import psycopg2

PASSWORD = "colab_secure_pw"
DSN = {
    "host": "localhost",
    "user": "postgres",
    "password": PASSWORD,
    "dbname": "postgres",
}

def recreate_database(name: str):
    with psycopg2.connect(**DSN) as conn:
        conn.autocommit = True
        with conn.cursor() as cur:
            cur.execute(f"DROP DATABASE IF EXISTS {name};")
            cur.execute(f"CREATE DATABASE {name};")
        print(f"✅ refreshed database '{name}'")

recreate_database("playground")
```

You can reuse `recreate_database("adventureworks")` or any other name as needed.

---

## 7. AdventureWorks (with CSV download)
Microsoft distributes the data separately from the community Git repo. This cell downloads the official archive, extracts the CSVs, and pipes them into PostgreSQL 14/16. Expect ~80 MB of transfers and 60–90 seconds of import time.

```bash
# Cell 7 — AdventureWorks schema + data
!mkdir -p /content/adventureworks && cd /content/adventureworks && \
  wget -q https://github.com/lorint/AdventureWorks-for-Postgres/archive/refs/heads/master.zip && \
  unzip -qo master.zip && \
  wget -q https://github.com/Microsoft/sql-server-samples/releases/download/adventureworks/AdventureWorks-oltp-install-script.zip && \
  unzip -qo AdventureWorks-oltp-install-script.zip

# copy CSVs into the expected data folder
!cp -v /content/adventureworks/AdventureWorks-oltp-install-script/InstWorks2014/Data/*.csv \
      /content/adventureworks/AdventureWorks-for-Postgres-master/data/ > /dev/null

# recreate database and load
!python - <<'PY'
import psycopg2

PASSWORD = "colab_secure_pw"
with psycopg2.connect(host="localhost", user="postgres", password=PASSWORD, dbname="postgres") as conn:
    conn.autocommit = True
    with conn.cursor() as cur:
        cur.execute("DROP DATABASE IF EXISTS adventureworks;")
        cur.execute("CREATE DATABASE adventureworks;")
PY

!sudo -u postgres psql -d adventureworks -f /content/adventureworks/AdventureWorks-for-Postgres-master/install.sql > /tmp/aw_install.log
!tail -20 /tmp/aw_install.log
```

> If you encounter `psql` prompts, add `PAGER=cat` before the command: `!PAGER=cat sudo -u postgres psql ...`

---

## 8. Pagila sample database
```bash
# Cell 8 — Pagila (PostgreSQL port of Sakila)
!wget -q https://github.com/devrimgunduz/pagila/raw/master/pagila-schema.sql
!wget -q https://github.com/devrimgunduz/pagila/raw/master/pagila-insert-data.sql

!python - <<'PY'
import psycopg2

PASSWORD = "colab_secure_pw"
with psycopg2.connect(host="localhost", user="postgres", password=PASSWORD, dbname="postgres") as conn:
    conn.autocommit = True
    with conn.cursor() as cur:
        cur.execute("DROP DATABASE IF EXISTS pagila;")
        cur.execute("CREATE DATABASE pagila;")
PY

!sudo -u postgres psql -d pagila -f pagila-schema.sql > /tmp/pagila_schema.log
!sudo -u postgres psql -d pagila -f pagila-insert-data.sql > /tmp/pagila_data.log
!tail -5 /tmp/pagila_data.log
```

---

## 9. Switch databases in notebooks
```python
# Cell 9 — convenient connection switchers
from IPython import get_ipython

def use_database(db_name: str):
    PASSWORD = "colab_secure_pw"
    conn = f"postgresql://postgres:{PASSWORD}@localhost/{db_name}"
    get_ipython().run_line_magic("sql", conn)
    print(f"Now connected to '{db_name}'")

def show_current_db():
    return get_ipython().run_line_magic(
        "sql",
        "SELECT current_database() AS database, current_user AS user, version() AS version;",
    )

use_database("adventureworks")
show_current_db()
```

---

## 10. Troubleshooting quick reference
| Symptom | Fix |
| --- | --- |
| `psql: FATAL: password authentication failed` | Rerun Cell 2 with a new password and update the Python snippets using it. |
| `%sql` not found | Ensure Cell 3 ran successfully (`pip install jupysql`), then rerun Cell 4. |
| `Command not found: pg_config` | Re-run Cell 1 to reinstall PostgreSQL packages; Colab VMs reset after idle. |
| `psql` stuck at `(END)` | Prefix commands with `PAGER=cat` or set `
\pset pager off` once inside `psql`. |
| AdventureWorks CSV missing | Confirm Cell 7 ran; Microsoft’s archive must be present before executing `install.sql`. |

---

## 11. Clean up (optional)
```bash
# Drop sample databases to reclaim space
!sudo -u postgres dropdb adventureworks
!sudo -u postgres dropdb pagila
```

Rerun Cells 6–8 at any time to recreate the practice environment.
