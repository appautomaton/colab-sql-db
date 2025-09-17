# Install Postgres and Load DB

## Install Required Packages

The user `postgres` is a special account created automatically when you install **PostgreSQL** with the `!apt install postgresql` command.
- System User: The installation creates a Linux system user named `postgres`.
- Database Role: It also creates a default superuser role (a "*user*" inside the database) also named `postgres`.

```python
# STEP 1 & 2
!apt update -qq  # Quiet mode to reduce output
!apt install postgresql postgresql-contrib ruby-full -y -qq
!pip install jupysql psycopg2-binary --quiet

# Start Postgres Service and Set Password (skip if already running)
!service postgresql start
!sudo -u postgres psql --pset=pager=off -c "ALTER USER postgres PASSWORD 'pg123';"
```
