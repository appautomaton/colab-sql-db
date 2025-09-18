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
