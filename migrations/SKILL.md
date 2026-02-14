---
name: migrations
description: "Implement safe, version-controlled database migrations with Alembic, Flyway, and Liquibase for zero-downtime schema changes."
---

# Migrations Skill

Implement safe, version-controlled database migrations for production environments.

## When to Use

Use this skill when the user wants to:
- Implement database schema migrations
- Version control database changes
- Set up Alembic for Python/SQLAlchemy projects
- Configure Flyway for Java/JVM applications
- Implement Liquibase for enterprise migrations
- Create zero-downtime migrations
- Handle data migrations and transformations
- Implement rollback strategies
- Set up multi-environment migrations
- Manage migration dependencies
- Implement blue-green deployments
- Handle large-scale data migrations

## Technology Stack

### Migration Tools

#### Alembic (Python)
- **SQLAlchemy migration tool**
  - Auto-generate migrations
  - Version control
  - Branching and merging
  - Offline mode
  - Custom scripts

#### Flyway (Java/JVM)
- **Database migration tool**
  - SQL-based migrations
  - Java-based migrations
  - Version control
  - Checksum validation
  - Repeatable migrations

#### Liquibase
- **Enterprise migration tool**
  - XML/YAML/JSON/SQL formats
  - Database independence
  - Rollback support
  - Preconditions
  - Contexts and labels

### Other Tools
- **Django Migrations**: Built-in Django ORM migrations
- **Knex.js**: Node.js migration tool
- **Active Record Migrations**: Rails migrations
- **TypeORM**: TypeScript/JavaScript migrations
- **golang-migrate**: Go migration tool

## Alembic (Python) Implementation

### Setup and Configuration

**Initialize Alembic:**
```bash
# Install Alembic
pip install alembic sqlalchemy psycopg2-binary

# Initialize Alembic in project
alembic init alembic

# Create migration
alembic revision --autogenerate -m "Create users table"

# Apply migrations
alembic upgrade head

# Rollback
alembic downgrade -1
```

**Configuration (alembic.ini):**
```ini
[alembic]
script_location = alembic
prepend_sys_path = .
version_path_separator = os

# Database URL from environment
sqlalchemy.url = driver://user:pass@localhost/dbname

# Logging
[loggers]
keys = root,sqlalchemy,alembic

[handlers]
keys = console

[formatters]
keys = generic

[logger_root]
level = WARN
handlers = console
qualname =

[logger_sqlalchemy]
level = WARN
handlers =
qualname = sqlalchemy.engine

[logger_alembic]
level = INFO
handlers =
qualname = alembic

[handler_console]
class = StreamHandler
args = (sys.stderr,)
level = NOTSET
formatter = generic

[formatter_generic]
format = %(levelname)-5.5s [%(name)s] %(message)s
datefmt = %H:%M:%S
```

**Environment Configuration (env.py):**
```python
from logging.config import fileConfig
from sqlalchemy import engine_from_config, pool
from alembic import context
import os
import sys

# Add your model's MetaData object here
sys.path.insert(0, os.path.dirname(os.path.dirname(__file__)))

from app.db.base import Base  # Import your models
from app.core.config import settings

# this is the Alembic Config object
config = context.config

# Override sqlalchemy.url from environment
config.set_main_option(
    'sqlalchemy.url',
    settings.DATABASE_URL
)

# Interpret the config file for Python logging
if config.config_file_name is not None:
    fileConfig(config.config_file_name)

target_metadata = Base.metadata


def run_migrations_offline() -> None:
    """Run migrations in 'offline' mode."""
    url = config.get_main_option("sqlalchemy.url")
    context.configure(
        url=url,
        target_metadata=target_metadata,
        literal_binds=True,
        dialect_opts={"paramstyle": "named"},
        compare_type=True,
        compare_server_default=True,
    )

    with context.begin_transaction():
        context.run_migrations()


def run_migrations_online() -> None:
    """Run migrations in 'online' mode."""
    connectable = engine_from_config(
        config.get_section(config.config_ini_section),
        prefix="sqlalchemy.",
        poolclass=pool.NullPool,
    )

    with connectable.connect() as connection:
        context.configure(
            connection=connection,
            target_metadata=target_metadata,
            compare_type=True,
            compare_server_default=True,
            include_schemas=True,
        )

        with context.begin_transaction():
            context.run_migrations()


if context.is_offline_mode():
    run_migrations_offline()
else:
    run_migrations_online()
```

### Creating Migrations

**Auto-Generate Migration:**
```python
# Example model
from sqlalchemy import Column, Integer, String, DateTime, Boolean
from sqlalchemy.ext.declarative import declarative_base
from datetime import datetime

Base = declarative_base()

class User(Base):
    __tablename__ = 'users'

    id = Column(Integer, primary_key=True, index=True)
    email = Column(String(255), unique=True, nullable=False, index=True)
    username = Column(String(100), unique=True, nullable=False)
    hashed_password = Column(String(255), nullable=False)
    is_active = Column(Boolean, default=True)
    is_superuser = Column(Boolean, default=False)
    created_at = Column(DateTime, default=datetime.utcnow)
    updated_at = Column(DateTime, default=datetime.utcnow, onupdate=datetime.utcnow)
```

```bash
# Generate migration from models
alembic revision --autogenerate -m "Create users table"
```

**Manual Migration:**
```python
# alembic/versions/xxxx_create_users_table.py
"""Create users table

Revision ID: xxxx
Revises:
Create Date: 2024-01-01 12:00:00.000000

"""
from alembic import op
import sqlalchemy as sa
from sqlalchemy.dialects import postgresql

# revision identifiers
revision = 'xxxx'
down_revision = None
branch_labels = None
depends_on = None


def upgrade() -> None:
    """Create users table with indexes."""
    op.create_table(
        'users',
        sa.Column('id', sa.Integer(), nullable=False),
        sa.Column('email', sa.String(255), nullable=False),
        sa.Column('username', sa.String(100), nullable=False),
        sa.Column('hashed_password', sa.String(255), nullable=False),
        sa.Column('is_active', sa.Boolean(), default=True),
        sa.Column('is_superuser', sa.Boolean(), default=False),
        sa.Column('created_at', sa.DateTime(), server_default=sa.text('now()')),
        sa.Column('updated_at', sa.DateTime(), server_default=sa.text('now()')),
        sa.PrimaryKeyConstraint('id'),
        sa.UniqueConstraint('email'),
        sa.UniqueConstraint('username')
    )

    # Create indexes
    op.create_index('ix_users_email', 'users', ['email'])
    op.create_index('ix_users_username', 'users', ['username'])


def downgrade() -> None:
    """Drop users table."""
    op.drop_index('ix_users_username', table_name='users')
    op.drop_index('ix_users_email', table_name='users')
    op.drop_table('users')
```

### Advanced Migration Patterns

**Data Migration:**
```python
"""Migrate user data

Revision ID: yyyy
Revises: xxxx
Create Date: 2024-01-02 12:00:00.000000

"""
from alembic import op
import sqlalchemy as sa
from sqlalchemy.sql import table, column

revision = 'yyyy'
down_revision = 'xxxx'


def upgrade() -> None:
    """Add role column and migrate existing data."""
    # Add new column
    op.add_column('users', sa.Column('role', sa.String(50), nullable=True))

    # Create temporary table reference
    users_table = table('users',
        column('id', sa.Integer),
        column('is_superuser', sa.Boolean),
        column('role', sa.String)
    )

    # Data migration
    op.execute(
        users_table.update()
        .where(users_table.c.is_superuser == True)
        .values(role='admin')
    )

    op.execute(
        users_table.update()
        .where(users_table.c.is_superuser == False)
        .values(role='user')
    )

    # Make column non-nullable after data migration
    op.alter_column('users', 'role', nullable=False)

    # Drop old column
    op.drop_column('users', 'is_superuser')


def downgrade() -> None:
    """Restore is_superuser column."""
    op.add_column('users', sa.Column('is_superuser', sa.Boolean(), default=False))

    users_table = table('users',
        column('id', sa.Integer),
        column('is_superuser', sa.Boolean),
        column('role', sa.String)
    )

    op.execute(
        users_table.update()
        .where(users_table.c.role == 'admin')
        .values(is_superuser=True)
    )

    op.alter_column('users', 'is_superuser', nullable=False)
    op.drop_column('users', 'role')
```

**Zero-Downtime Migration:**
```python
"""Add email_verified column with zero downtime

Revision ID: zzzz
Revises: yyyy
Create Date: 2024-01-03 12:00:00.000000

"""
from alembic import op
import sqlalchemy as sa

revision = 'zzzz'
down_revision = 'yyyy'


def upgrade() -> None:
    """
    Zero-downtime migration strategy:
    1. Add nullable column
    2. Backfill data
    3. Make non-nullable in next migration
    """
    # Step 1: Add nullable column (safe for existing rows)
    op.add_column(
        'users',
        sa.Column('email_verified', sa.Boolean(), nullable=True)
    )

    # Step 2: Set default for new rows
    op.alter_column(
        'users',
        'email_verified',
        server_default='false'
    )

    # Step 3: Backfill existing data (can be done gradually)
    connection = op.get_bind()
    connection.execute(
        sa.text("UPDATE users SET email_verified = false WHERE email_verified IS NULL")
    )


def downgrade() -> None:
    """Remove email_verified column."""
    op.drop_column('users', 'email_verified')
```

**Batch Operations for Large Tables:**
```python
"""Add index on large table

Revision ID: aaaa
Revises: zzzz
Create Date: 2024-01-04 12:00:00.000000

"""
from alembic import op
import sqlalchemy as sa

revision = 'aaaa'
down_revision = 'zzzz'


def upgrade() -> None:
    """Add index concurrently to avoid locking."""
    # PostgreSQL: Create index concurrently (no table lock)
    op.create_index(
        'ix_users_created_at',
        'users',
        ['created_at'],
        postgresql_concurrently=True
    )


def downgrade() -> None:
    """Drop index."""
    op.drop_index('ix_users_created_at', table_name='users')
```

## Flyway (Java/SQL) Implementation

### Setup and Configuration

**Gradle Configuration:**
```gradle
plugins {
    id 'org.flywaydb.flyway' version '9.0.0'
}

dependencies {
    implementation 'org.flywaydb:flyway-core:9.0.0'
    implementation 'org.postgresql:postgresql:42.5.0'
}

flyway {
    url = 'jdbc:postgresql://localhost:5432/mydb'
    user = 'dbuser'
    password = 'dbpass'
    locations = ['classpath:db/migration']
    baselineOnMigrate = true
    outOfOrder = false
    validateOnMigrate = true
}
```

**Application Configuration:**
```java
import org.flywaydb.core.Flyway;

public class DatabaseMigration {
    public static void migrate(String url, String user, String password) {
        Flyway flyway = Flyway.configure()
            .dataSource(url, user, password)
            .locations("classpath:db/migration")
            .baselineOnMigrate(true)
            .load();

        // Run migrations
        flyway.migrate();
    }

    public static void main(String[] args) {
        migrate(
            "jdbc:postgresql://localhost:5432/mydb",
            "dbuser",
            "dbpass"
        );
    }
}
```

### Creating Flyway Migrations

**SQL Migration (V1__Create_users_table.sql):**
```sql
-- V1__Create_users_table.sql
-- Flyway uses naming convention: V{version}__{description}.sql

CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    email VARCHAR(255) NOT NULL UNIQUE,
    username VARCHAR(100) NOT NULL UNIQUE,
    hashed_password VARCHAR(255) NOT NULL,
    is_active BOOLEAN DEFAULT true,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX ix_users_email ON users(email);
CREATE INDEX ix_users_username ON users(username);

-- Add trigger for updated_at
CREATE OR REPLACE FUNCTION update_updated_at_column()
RETURNS TRIGGER AS $$
BEGIN
    NEW.updated_at = CURRENT_TIMESTAMP;
    RETURN NEW;
END;
$$ language 'plpgsql';

CREATE TRIGGER update_users_updated_at
    BEFORE UPDATE ON users
    FOR EACH ROW
    EXECUTE FUNCTION update_updated_at_column();
```

**Java-Based Migration:**
```java
// V2__Add_user_roles.java
package db.migration;

import org.flywaydb.core.api.migration.BaseJavaMigration;
import org.flywaydb.core.api.migration.Context;
import java.sql.Statement;

public class V2__Add_user_roles extends BaseJavaMigration {
    @Override
    public void migrate(Context context) throws Exception {
        try (Statement statement = context.getConnection().createStatement()) {
            // Add role column
            statement.execute(
                "ALTER TABLE users ADD COLUMN role VARCHAR(50)"
            );

            // Set default values
            statement.execute(
                "UPDATE users SET role = 'user' WHERE role IS NULL"
            );

            // Make non-nullable
            statement.execute(
                "ALTER TABLE users ALTER COLUMN role SET NOT NULL"
            );
        }
    }
}
```

**Repeatable Migration (R__Create_views.sql):**
```sql
-- R__Create_views.sql
-- Repeatable migrations run whenever their checksum changes

CREATE OR REPLACE VIEW active_users AS
SELECT id, email, username, created_at
FROM users
WHERE is_active = true;

CREATE OR REPLACE VIEW user_stats AS
SELECT
    DATE_TRUNC('day', created_at) as date,
    COUNT(*) as total_users,
    COUNT(*) FILTER (WHERE is_active = true) as active_users
FROM users
GROUP BY DATE_TRUNC('day', created_at);
```

## Liquibase Implementation

### Setup and Configuration

**liquibase.properties:**
```properties
changeLogFile=db/changelog/db.changelog-master.xml
url=jdbc:postgresql://localhost:5432/mydb
username=dbuser
password=dbpass
driver=org.postgresql.Driver
classpath=postgresql-42.5.0.jar
```

**Master Changelog (db.changelog-master.xml):**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<databaseChangeLog
    xmlns="http://www.liquibase.org/xml/ns/dbchangelog"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.liquibase.org/xml/ns/dbchangelog
    http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-4.0.xsd">

    <include file="db/changelog/changes/v1.0/01-create-users-table.xml"/>
    <include file="db/changelog/changes/v1.0/02-add-user-roles.xml"/>
    <include file="db/changelog/changes/v1.1/01-add-user-preferences.xml"/>
</databaseChangeLog>
```

### Creating Liquibase Changesets

**XML Changeset:**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<databaseChangeLog
    xmlns="http://www.liquibase.org/xml/ns/dbchangelog"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.liquibase.org/xml/ns/dbchangelog
    http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-4.0.xsd">

    <changeSet id="1" author="developer">
        <createTable tableName="users">
            <column name="id" type="bigint" autoIncrement="true">
                <constraints primaryKey="true" nullable="false"/>
            </column>
            <column name="email" type="varchar(255)">
                <constraints nullable="false" unique="true"/>
            </column>
            <column name="username" type="varchar(100)">
                <constraints nullable="false" unique="true"/>
            </column>
            <column name="hashed_password" type="varchar(255)">
                <constraints nullable="false"/>
            </column>
            <column name="is_active" type="boolean" defaultValueBoolean="true"/>
            <column name="created_at" type="timestamp" defaultValueComputed="CURRENT_TIMESTAMP"/>
            <column name="updated_at" type="timestamp" defaultValueComputed="CURRENT_TIMESTAMP"/>
        </createTable>

        <createIndex indexName="ix_users_email" tableName="users">
            <column name="email"/>
        </createIndex>

        <rollback>
            <dropTable tableName="users"/>
        </rollback>
    </changeSet>

    <changeSet id="2" author="developer">
        <addColumn tableName="users">
            <column name="role" type="varchar(50)" defaultValue="user">
                <constraints nullable="false"/>
            </column>
        </addColumn>

        <rollback>
            <dropColumn tableName="users" columnName="role"/>
        </rollback>
    </changeSet>
</databaseChangeLog>
```

**YAML Changeset:**
```yaml
databaseChangeLog:
  - changeSet:
      id: 1
      author: developer
      changes:
        - createTable:
            tableName: users
            columns:
              - column:
                  name: id
                  type: bigint
                  autoIncrement: true
                  constraints:
                    primaryKey: true
                    nullable: false
              - column:
                  name: email
                  type: varchar(255)
                  constraints:
                    nullable: false
                    unique: true
              - column:
                  name: username
                  type: varchar(100)
                  constraints:
                    nullable: false
                    unique: true
              - column:
                  name: hashed_password
                  type: varchar(255)
                  constraints:
                    nullable: false
        - createIndex:
            indexName: ix_users_email
            tableName: users
            columns:
              - column:
                  name: email
      rollback:
        - dropTable:
            tableName: users
```

## Migration Strategies

### 1. Expand-Contract Pattern

**Phase 1: Expand (Add new column):**
```python
def upgrade() -> None:
    """Add new email_address column."""
    # Add nullable column
    op.add_column('users', sa.Column('email_address', sa.String(255), nullable=True))

    # Copy data from old column
    op.execute("UPDATE users SET email_address = email")

    # Application now writes to both columns
```

**Phase 2: Contract (Remove old column):**
```python
def upgrade() -> None:
    """Remove old email column after migration period."""
    # Make new column non-nullable
    op.alter_column('users', 'email_address', nullable=False)

    # Add constraints
    op.create_unique_constraint('uq_users_email_address', 'users', ['email_address'])

    # Drop old column
    op.drop_column('users', 'email')
```

### 2. Blue-Green Migrations

**Backwards Compatible Changes:**
```python
def upgrade() -> None:
    """
    Add new column that old code can ignore.
    Both old and new code can run simultaneously.
    """
    # Add column with default
    op.add_column(
        'users',
        sa.Column('email_verified_at', sa.DateTime(), nullable=True)
    )

    # Old code ignores this column
    # New code uses it
    # No downtime required
```

### 3. Large Table Migrations

**Chunked Data Migration:**
```python
def upgrade() -> None:
    """Migrate data in chunks to avoid locks."""
    connection = op.get_bind()

    chunk_size = 10000
    offset = 0

    while True:
        result = connection.execute(
            sa.text(f"""
                UPDATE users
                SET normalized_email = LOWER(email)
                WHERE id > :offset AND id <= :offset + :chunk_size
                AND normalized_email IS NULL
            """),
            {"offset": offset, "chunk_size": chunk_size}
        )

        if result.rowcount == 0:
            break

        offset += chunk_size

        # Small delay to reduce load
        import time
        time.sleep(0.1)
```

## Testing Migrations

### Migration Testing Framework

**Python Test:**
```python
import pytest
from alembic import command
from alembic.config import Config
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker

class TestMigrations:
    @pytest.fixture
    def alembic_config(self):
        """Create Alembic configuration."""
        config = Config("alembic.ini")
        config.set_main_option("sqlalchemy.url", "postgresql://test:test@localhost/test_db")
        return config

    @pytest.fixture
    def db_engine(self):
        """Create test database engine."""
        engine = create_engine("postgresql://test:test@localhost/test_db")
        yield engine
        engine.dispose()

    def test_upgrade_downgrade(self, alembic_config, db_engine):
        """Test migration up and down."""
        # Upgrade to head
        command.upgrade(alembic_config, "head")

        # Verify tables exist
        from sqlalchemy import inspect
        inspector = inspect(db_engine)
        assert 'users' in inspector.get_table_names()

        # Downgrade
        command.downgrade(alembic_config, "base")

        # Verify tables removed
        inspector = inspect(db_engine)
        assert 'users' not in inspector.get_table_names()

    def test_migration_idempotent(self, alembic_config):
        """Test running migrations twice."""
        command.upgrade(alembic_config, "head")
        # Should not fail on second run
        command.upgrade(alembic_config, "head")
```

## Best Practices

### 1. Version Control
- Commit migrations with code changes
- Never modify existing migrations
- Use descriptive migration names
- Include both upgrade and downgrade
- Tag releases with migration versions

### 2. Safety
- Test migrations on staging first
- Always write downgrade scripts
- Use transactions where possible
- Avoid DDL in transactions (varies by database)
- Take backups before major migrations

### 3. Zero-Downtime
- Add columns as nullable first
- Backfill data separately
- Make non-nullable in later migration
- Use expand-contract pattern
- Create indexes concurrently

### 4. Performance
- Batch large data updates
- Use appropriate chunk sizes
- Create indexes concurrently
- Monitor migration duration
- Plan for maintenance windows

### 5. Data Integrity
- Add constraints in separate migrations
- Validate data before constraints
- Use check constraints
- Implement foreign keys properly
- Test rollback scenarios

### 6. Multi-Environment
- Use environment-specific configurations
- Parameterize database URLs
- Test in all environments
- Maintain separate migration states
- Document environment differences

### 7. Team Coordination
- Communicate migration plans
- Avoid migration conflicts
- Merge migrations carefully
- Use migration branches
- Document breaking changes

## Common Anti-Patterns to Avoid

1. **Modifying existing migrations**: Always create new migrations
2. **No rollback**: Always implement downgrade scripts
3. **Large data changes**: Break into smaller chunks
4. **No testing**: Test migrations before production
5. **Ignoring indexes**: Create indexes for foreign keys
6. **Breaking changes**: Use expand-contract pattern
7. **No backups**: Always backup before migrations

## Deliverables

When implementing migrations, ensure you deliver:

1. **Migration Configuration**
   - Tool setup (Alembic/Flyway/Liquibase)
   - Environment configurations
   - Connection settings

2. **Migration Scripts**
   - Schema migrations
   - Data migrations
   - Rollback scripts
   - Seed data scripts

3. **Documentation**
   - Migration strategy document
   - Rollback procedures
   - Testing procedures
   - Troubleshooting guide

4. **CI/CD Integration**
   - Automated migration tests
   - Deployment scripts
   - Health checks
   - Rollback automation

5. **Monitoring**
   - Migration tracking
   - Performance metrics
   - Error alerting
   - Audit logs

## Quality Checklist

Before completing a migration implementation, verify:

- [ ] Migration tool properly configured
- [ ] All migrations have upgrade and downgrade
- [ ] Migrations tested on development environment
- [ ] Migrations tested on staging environment
- [ ] Data integrity verified after migration
- [ ] Rollback procedures tested
- [ ] Performance impact assessed
- [ ] Indexes created for foreign keys
- [ ] Constraints validated
- [ ] Backup strategy in place
- [ ] Team notified of migration schedule
- [ ] Documentation updated
- [ ] CI/CD pipeline includes migrations
- [ ] Monitoring configured
- [ ] Emergency rollback plan documented
