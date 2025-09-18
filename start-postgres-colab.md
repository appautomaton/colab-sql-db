# Install Postgres and Load DB

## Install Required Packages

The user `postgres` is a special account created automatically when you install **PostgreSQL** with the `!apt install postgresql` command.
- System User: The installation creates a Linux system user named `postgres`.
- Database Role: It also creates a default superuser role (a "*user*" inside the database) also named `postgres`.

```python
# Step 1: Install Dependencies (if not already installed)
!apt update -qq  # Quiet mode to reduce output
!apt install postgresql postgresql-contrib ruby-full -y -qq
!pip install jupysql psycopg2-binary --quiet

# Step 2: Start Postgres and Set Password (skip if running)
!service postgresql start
!sudo -u postgres psql --pset=pager=off -c "ALTER USER postgres PASSWORD 'pg123';"
```

> [!TIP]
> **Peer Authentication**: 
> By default, when you install `PostgreSQL` on a Linux system, it uses a security method called `peer authentication` for *local* connections --- `Peer authentication` checks if the system username matches the requested database username (`postgres`)
>
> Run as the `postgres` User -> `!sudo -u postgres psql -l -P pager=off`

## Load the AdventureWorks

```python
# Step 3: Clone the AdventureWorks-for-Postgres Repo (contains install.sql and update_csvs.rb)
# If dir exists, remove and re-clone for freshness (optional)
!rm -rf AdventureWorks-for-Postgres
!git clone https://github.com/lorint/AdventureWorks-for-Postgres.git -q
%cd AdventureWorks-for-Postgres

# Step 4: Download the Microsoft AdventureWorks OLTP Script Zip
!wget https://github.com/microsoft/sql-server-samples/releases/download/adventureworks/AdventureWorks-oltp-install-script.zip -q

# Step 5: Unzip the Script (extracts CSVs and instawdb.sql directly into current directory)
!unzip -q AdventureWorks-oltp-install-script.zip
!rm AdventureWorks-oltp-install-script.zip instawdb.sql  # Clean up unnecessary files (instawdb.sql is SQL Server specific)

# Make files readable by everyone (for safety, though may not be needed)
!chmod -R a+r .

# Step 6: Update CSVs for Postgres Compatibility
!ruby update_csvs.rb

# Step 7: Drop Existing Database (if any) and Create the AdventureWorks Database
!sudo -u postgres psql --pset=pager=off -c "DROP DATABASE IF EXISTS adventureworks;"
!sudo -u postgres psql --pset=pager=off -c "CREATE DATABASE adventureworks;"
```
