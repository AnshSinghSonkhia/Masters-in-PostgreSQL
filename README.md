# 🐘 Masters in PostgreSQL

> A comprehensive guide to master PostgreSQL for interviews and real-world applications

[![PostgreSQL](https://img.shields.io/badge/PostgreSQL-316192?style=for-the-badge&logo=postgresql&logoColor=white)](https://www.postgresql.org/)

---

## 📚 Table of Contents

- [Introduction](#-introduction)
- [Architecture & Core Concepts](#-architecture--core-concepts)
- [SQL Fundamentals](#-sql-fundamentals)
- [Advanced SQL](#-advanced-sql)
- [Indexes & Performance](#-indexes--performance)
- [Transactions & Concurrency](#-transactions--concurrency)
- [Query Optimization](#-query-optimization)
- [Backup & Recovery](#-backup--recovery)
- [Replication & High Availability](#-replication--high-availability)
- [Security](#-security)
- [JSON & NoSQL Features](#-json--nosql-features)
- [Extensions & Advanced Features](#-extensions--advanced-features)
- [Administration & Monitoring](#-administration--monitoring)
- [Common Interview Questions](#-common-interview-questions)
- [Best Practices](#-best-practices)
- [Troubleshooting](#-troubleshooting)

---

## 🌟 Introduction

### What is PostgreSQL?
- **Open-source Relational Database** - Most advanced open-source database
- **ACID Compliant** - Ensures data integrity
- **Object-Relational** - Supports objects, classes, inheritance
- **Extensible** - Custom data types, functions, operators
- **Standards Compliant** - SQL:2011 compliant

### Why PostgreSQL?
- ✅ **ACID Compliance** - Data consistency guaranteed
- ✅ **Complex Queries** - Advanced SQL features
- ✅ **Extensibility** - Custom functions and types
- ✅ **JSON Support** - NoSQL capabilities
- ✅ **Full-Text Search** - Built-in FTS
- ✅ **Geospatial** - PostGIS extension
- ✅ **Scalability** - Horizontal and vertical scaling
- ✅ **Community** - Active development and support

---

## 🏗️ Architecture & Core Concepts

### PostgreSQL Architecture

#### Process Architecture
```
┌─────────────────────────────────────┐
│         Client Application          │
└──────────────┬──────────────────────┘
               │
┌──────────────▼──────────────────────┐
│          Postmaster                 │
│    (Main PostgreSQL Process)        │
└──────────────┬──────────────────────┘
               │
     ┌─────────┴─────────┬─────────────┐
     │                   │             │
┌────▼──────┐  ┌────────▼────┐  ┌────▼────┐
│  Backend  │  │   Backend   │  │ Backend │
│  Process  │  │   Process   │  │ Process │
└───────────┘  └─────────────┘  └─────────┘
```

#### Memory Architecture
- **Shared Memory**
  - Shared buffers
  - WAL buffers
  - CLOG buffers
  - Lock space

- **Process Memory**
  - Work memory (work_mem)
  - Maintenance work memory
  - Temp buffers

#### Storage Architecture
```
Database Cluster
    ├── base/           # Database files
    ├── global/         # Cluster-wide tables
    ├── pg_wal/         # Write-Ahead Logs
    ├── pg_xact/        # Transaction commit status
    └── postgresql.conf # Main configuration
```

### Core Components

#### 1. **Postmaster Process**
- Main PostgreSQL daemon
- Handles connection requests
- Spawns backend processes
- Manages shared memory

#### 2. **Backend Processes**
- One per client connection
- Executes SQL queries
- Manages transactions
- Accesses shared buffers

#### 3. **Background Processes**
- **Writer** - Writes dirty buffers
- **Checkpointer** - Creates checkpoints
- **Autovacuum** - Automatic maintenance
- **WAL Writer** - Writes WAL buffers
- **Stats Collector** - Collects statistics

#### 4. **Shared Buffers**
- Cache for data pages
- Reduces disk I/O
- LRU replacement algorithm
- Typical size: 25% of RAM

---

## 📝 SQL Fundamentals

### Data Types

#### Numeric Types
```sql
- INTEGER (INT)      -- 4 bytes, -2^31 to 2^31-1
- BIGINT            -- 8 bytes, -2^63 to 2^63-1
- DECIMAL/NUMERIC   -- Variable precision
- REAL              -- 4 bytes, 6 decimal digits
- DOUBLE PRECISION  -- 8 bytes, 15 decimal digits
- SERIAL/BIGSERIAL  -- Auto-incrementing integer
```

#### Character Types
```sql
- CHAR(n)           -- Fixed-length
- VARCHAR(n)        -- Variable-length with limit
- TEXT              -- Variable unlimited length
```

#### Date/Time Types
```sql
- DATE              -- Date only
- TIME              -- Time only
- TIMESTAMP         -- Date and time
- TIMESTAMPTZ       -- Timestamp with timezone
- INTERVAL          -- Time interval
```

#### Boolean & Binary
```sql
- BOOLEAN           -- true/false/null
- BYTEA             -- Binary data
```

#### Special Types
```sql
- UUID              -- Universally unique identifier
- JSON/JSONB        -- JSON data
- ARRAY             -- Arrays of any type
- HSTORE            -- Key-value pairs
- INET/CIDR         -- Network addresses
```

### DDL (Data Definition Language)

#### Tables
```sql
-- Create table
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    email VARCHAR(255) UNIQUE NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Alter table
ALTER TABLE users ADD COLUMN age INTEGER;
ALTER TABLE users DROP COLUMN age;
ALTER TABLE users RENAME COLUMN email TO email_address;

-- Drop table
DROP TABLE users;
DROP TABLE IF EXISTS users CASCADE;
```

#### Constraints
```sql
- PRIMARY KEY       -- Unique identifier
- FOREIGN KEY       -- Referential integrity
- UNIQUE           -- Unique values
- NOT NULL         -- Required field
- CHECK            -- Custom validation
- DEFAULT          -- Default value
```

### DML (Data Manipulation Language)

#### Basic Operations
```sql
-- INSERT
INSERT INTO users (email, name) VALUES ('test@example.com', 'John');
INSERT INTO users (email, name) 
VALUES ('user1@example.com', 'User 1'),
       ('user2@example.com', 'User 2');

-- UPDATE
UPDATE users SET name = 'Jane' WHERE id = 1;
UPDATE users SET updated_at = NOW() WHERE created_at < '2023-01-01';

-- DELETE
DELETE FROM users WHERE id = 1;
DELETE FROM users WHERE created_at < NOW() - INTERVAL '1 year';

-- SELECT
SELECT * FROM users;
SELECT id, email FROM users WHERE active = true;
SELECT COUNT(*) FROM users GROUP BY country;
```

#### UPSERT (INSERT ON CONFLICT)
```sql
INSERT INTO users (email, name) 
VALUES ('test@example.com', 'John')
ON CONFLICT (email) 
DO UPDATE SET name = EXCLUDED.name, updated_at = NOW();
```

---

## 🚀 Advanced SQL

### Window Functions

#### Ranking Functions
```sql
-- ROW_NUMBER() - Unique number for each row
SELECT name, salary,
       ROW_NUMBER() OVER (ORDER BY salary DESC) as rank
FROM employees;

-- RANK() - Same rank for ties, gaps in sequence
SELECT name, salary,
       RANK() OVER (ORDER BY salary DESC) as rank
FROM employees;

-- DENSE_RANK() - Same rank for ties, no gaps
SELECT name, salary,
       DENSE_RANK() OVER (ORDER BY salary DESC) as rank
FROM employees;

-- NTILE(n) - Divide rows into n groups
SELECT name, salary,
       NTILE(4) OVER (ORDER BY salary) as quartile
FROM employees;
```

#### Analytical Functions
```sql
-- LAG/LEAD - Access previous/next row
SELECT date, sales,
       LAG(sales, 1) OVER (ORDER BY date) as prev_sales,
       LEAD(sales, 1) OVER (ORDER BY date) as next_sales
FROM daily_sales;

-- Running totals
SELECT date, amount,
       SUM(amount) OVER (ORDER BY date) as running_total
FROM transactions;

-- Moving average
SELECT date, price,
       AVG(price) OVER (
           ORDER BY date 
           ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
       ) as moving_avg_3
FROM stock_prices;
```

### Common Table Expressions (CTEs)

#### Basic CTE
```sql
WITH active_users AS (
    SELECT * FROM users WHERE active = true
)
SELECT * FROM active_users WHERE created_at > '2023-01-01';
```

#### Recursive CTE
```sql
-- Employee hierarchy
WITH RECURSIVE employee_hierarchy AS (
    -- Anchor: CEO
    SELECT id, name, manager_id, 1 as level
    FROM employees
    WHERE manager_id IS NULL
    
    UNION ALL
    
    -- Recursive part
    SELECT e.id, e.name, e.manager_id, h.level + 1
    FROM employees e
    JOIN employee_hierarchy h ON e.manager_id = h.id
)
SELECT * FROM employee_hierarchy ORDER BY level;
```

### Subqueries

#### Types of Subqueries
```sql
-- Scalar subquery (returns single value)
SELECT name, salary 
FROM employees 
WHERE salary > (SELECT AVG(salary) FROM employees);

-- Row subquery
SELECT * FROM products 
WHERE (category_id, price) = (
    SELECT category_id, MAX(price) 
    FROM products 
    GROUP BY category_id
);

-- Table subquery
SELECT * FROM (
    SELECT category, SUM(sales) as total 
    FROM sales 
    GROUP BY category
) t WHERE total > 1000;
```

### JOINs

#### Types of JOINs
```sql
-- INNER JOIN - Matching records only
SELECT o.*, c.name 
FROM orders o
INNER JOIN customers c ON o.customer_id = c.id;

-- LEFT JOIN - All from left table
SELECT c.*, o.order_date 
FROM customers c
LEFT JOIN orders o ON c.id = o.customer_id;

-- RIGHT JOIN - All from right table
SELECT o.*, c.name 
FROM orders o
RIGHT JOIN customers c ON o.customer_id = c.id;

-- FULL OUTER JOIN - All from both tables
SELECT * 
FROM table1 t1
FULL OUTER JOIN table2 t2 ON t1.id = t2.id;

-- CROSS JOIN - Cartesian product
SELECT * FROM colors CROSS JOIN sizes;

-- Self JOIN
SELECT e1.name as employee, e2.name as manager
FROM employees e1
JOIN employees e2 ON e1.manager_id = e2.id;
```

---

## 🔍 Indexes & Performance

### Index Types

#### 1. **B-Tree Index** (Default)
```sql
-- Best for: Equality and range queries
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_orders_date ON orders(order_date);

-- Composite index
CREATE INDEX idx_users_name_email ON users(last_name, first_name);
```

#### 2. **Hash Index**
```sql
-- Best for: Equality comparisons only
CREATE INDEX idx_users_id_hash ON users USING HASH (id);
```

#### 3. **GiST Index**
```sql
-- Best for: Geometric data, full-text search
CREATE INDEX idx_locations ON places USING GIST (location);
```

#### 4. **GIN Index**
```sql
-- Best for: Arrays, JSONB, full-text search
CREATE INDEX idx_tags ON posts USING GIN (tags);
```

### 5. **BRIN Index**

```sql
-- Best for: Large tables with natural ordering
CREATE INDEX idx_logs_created ON logs USING BRIN (created_at);
```

#### 6. **SP-GiST Index**
```sql
-- Best for: Space-partitioned data structures
CREATE INDEX idx_phone ON contacts USING SPGIST (phone_number);
```

### Index Strategies

#### Partial Indexes
```sql
-- Index only subset of data
CREATE INDEX idx_active_users ON users (email) 
WHERE active = true;

CREATE INDEX idx_recent_orders ON orders (customer_id) 
WHERE created_at > '2023-01-01';
```

#### Expression Indexes
```sql
-- Index on computed values
CREATE INDEX idx_users_lower_email ON users (LOWER(email));
CREATE INDEX idx_orders_year ON orders (EXTRACT(YEAR FROM created_at));
```

#### Covering Indexes (INCLUDE)
```sql
-- Include non-key columns
CREATE INDEX idx_products_category 
ON products (category_id) 
INCLUDE (name, price);
```

### Query Performance

#### EXPLAIN & EXPLAIN ANALYZE
```sql
-- Show query plan
EXPLAIN SELECT * FROM users WHERE email = 'test@example.com';

-- Show plan with execution statistics
EXPLAIN ANALYZE SELECT * FROM users WHERE email = 'test@example.com';

-- Verbose output
EXPLAIN (ANALYZE, VERBOSE, BUFFERS) 
SELECT * FROM users WHERE email = 'test@example.com';
```

#### Query Plan Nodes
```
- Seq Scan          -- Full table scan
- Index Scan        -- Index lookup
- Index Only Scan   -- Data from index only
- Bitmap Scan       -- Multiple index combine
- Nested Loop       -- For each row, scan other table
- Hash Join         -- Build hash table
- Merge Join        -- Sort both sides and merge
```

---

## 🔐 Transactions & Concurrency

### ACID Properties
- **Atomicity** - All or nothing
- **Consistency** - Valid state transitions
- **Isolation** - Concurrent execution
- **Durability** - Survives failures

### Transaction Control
```sql
-- Basic transaction
BEGIN;
INSERT INTO accounts (name, balance) VALUES ('John', 1000);
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;
COMMIT;

-- Rollback on error
BEGIN;
-- operations
ROLLBACK;

-- Savepoints
BEGIN;
INSERT INTO orders (customer_id) VALUES (1);
SAVEPOINT sp1;
INSERT INTO order_items (order_id, product_id) VALUES (1, 100);
-- If error:
ROLLBACK TO sp1;
COMMIT;
```

### Isolation Levels

#### 1. **READ UNCOMMITTED**
```sql
-- Dirty reads possible (not supported in PostgreSQL)
SET TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;
```

#### 2. **READ COMMITTED** (Default)
```sql
-- No dirty reads, possible non-repeatable reads
SET TRANSACTION ISOLATION LEVEL READ COMMITTED;
```

#### 3. **REPEATABLE READ**
```sql
-- No dirty/non-repeatable reads, possible phantom reads
SET TRANSACTION ISOLATION LEVEL REPEATABLE READ;
```

#### 4. **SERIALIZABLE**
```sql
-- Full isolation, no anomalies
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;
```

### Locking

#### Row-Level Locks
```sql
-- FOR UPDATE - Exclusive lock
SELECT * FROM accounts WHERE id = 1 FOR UPDATE;

-- FOR NO KEY UPDATE - Allows foreign key checks
SELECT * FROM accounts WHERE id = 1 FOR NO KEY UPDATE;

-- FOR SHARE - Shared lock
SELECT * FROM accounts WHERE id = 1 FOR SHARE;

-- FOR KEY SHARE - Weakest lock
SELECT * FROM accounts WHERE id = 1 FOR KEY SHARE;
```

#### Table-Level Locks
```sql
-- Explicit table lock
LOCK TABLE accounts IN ACCESS EXCLUSIVE MODE;

-- Lock modes:
-- ACCESS SHARE
-- ROW SHARE
-- ROW EXCLUSIVE
-- SHARE UPDATE EXCLUSIVE
-- SHARE
-- SHARE ROW EXCLUSIVE
-- EXCLUSIVE
-- ACCESS EXCLUSIVE
```

#### Advisory Locks
```sql
-- Session-level advisory lock
SELECT pg_advisory_lock(12345);
SELECT pg_advisory_unlock(12345);

-- Transaction-level advisory lock
SELECT pg_advisory_xact_lock(12345);
```

### MVCC (Multi-Version Concurrency Control)
- **No read locks** - Readers don't block writers
- **Version chains** - Multiple versions of rows
- **Transaction ID (XID)** - Identifies transactions
- **Visibility rules** - Which version to see

---

## ⚡ Query Optimization

### Optimization Techniques

#### 1. **Index Usage**
```sql
-- Ensure indexes are used
-- Bad: Function on indexed column
SELECT * FROM users WHERE LOWER(email) = 'test@example.com';

-- Good: Use expression index
CREATE INDEX idx_lower_email ON users (LOWER(email));
```

#### 2. **Join Optimization**
```sql
-- Join order matters
-- Let PostgreSQL optimize with JOIN
SELECT * FROM small_table 
JOIN large_table ON small_table.id = large_table.foreign_id;

-- Force join order if needed
SET join_collapse_limit = 1;
```

#### 3. **Subquery vs JOIN**
```sql
-- Often JOIN is faster than subquery
-- Subquery
SELECT * FROM orders 
WHERE customer_id IN (SELECT id FROM customers WHERE country = 'USA');

-- JOIN (usually better)
SELECT o.* FROM orders o
JOIN customers c ON o.customer_id = c.id
WHERE c.country = 'USA';
```

#### 4. **LIMIT Optimization**
```sql
-- Use LIMIT early
-- Bad: Get all then limit
SELECT * FROM (
    SELECT * FROM large_table ORDER BY created_at DESC
) t LIMIT 10;

-- Good: Limit in main query
SELECT * FROM large_table ORDER BY created_at DESC LIMIT 10;
```

### Statistics & Vacuum

#### Table Statistics
```sql
-- Update statistics
ANALYZE table_name;

-- View statistics
SELECT * FROM pg_stats WHERE tablename = 'users';

-- Auto-analyze settings
ALTER TABLE users SET (autovacuum_analyze_scale_factor = 0.01);
```

#### VACUUM
```sql
-- Remove dead tuples
VACUUM table_name;

-- VACUUM and update statistics
VACUUM ANALYZE table_name;

-- Full vacuum (locks table)
VACUUM FULL table_name;

-- Auto-vacuum settings
ALTER TABLE users SET (
    autovacuum_vacuum_scale_factor = 0.1,
    autovacuum_vacuum_threshold = 50
);
```

### Query Hints

#### Planner Configuration
```sql
-- Disable specific plan types (for testing)
SET enable_seqscan = OFF;
SET enable_hashjoin = OFF;
SET enable_nestloop = OFF;

-- Cost parameters
SET random_page_cost = 1.1;  -- SSD optimization
SET work_mem = '256MB';      -- More memory for sorts
```

---

## 💾 Backup & Recovery

### Backup Types

#### 1. **SQL Dump (pg_dump)**
```bash
# Basic backup
pg_dump dbname > backup.sql

# Compressed backup
pg_dump -Fc dbname > backup.dump

# Schema only
pg_dump --schema-only dbname > schema.sql

# Data only
pg_dump --data-only dbname > data.sql

# Specific tables
pg_dump -t users -t orders dbname > tables.sql
```

#### 2. **Physical Backup (File System)**
```bash
# Stop server and copy data directory
systemctl stop postgresql
cp -r /var/lib/postgresql/data /backup/

# Or use pg_basebackup
pg_basebackup -D /backup/base -Ft -z -P
```

#### 3. **Continuous Archiving (WAL)**
```sql
-- Enable archive mode
archive_mode = on
archive_command = 'cp %p /archive/%f'

-- For streaming replication
wal_level = replica
max_wal_senders = 3
```

### Recovery

#### Point-in-Time Recovery (PITR)
```sql
-- recovery.conf settings
restore_command = 'cp /archive/%f %p'
recovery_target_time = '2023-12-01 12:00:00'
recovery_target_action = 'promote'
```

#### Restore Commands
```bash
# Restore from SQL dump
psql dbname < backup.sql

# Restore from custom format
pg_restore -d dbname backup.dump

# Restore specific tables
pg_restore -d dbname -t users backup.dump

# Parallel restore
pg_restore -d dbname -j 4 backup.dump
```

---

## 🔄 Replication & High Availability

### Replication Types

#### 1. **Streaming Replication**
```sql
-- Primary server config
wal_level = replica
max_wal_senders = 3
wal_keep_segments = 64

-- Standby server recovery.conf
standby_mode = on
primary_conninfo = 'host=primary_host user=repl_user'
```

#### 2. **Logical Replication**
```sql
-- Publisher
CREATE PUBLICATION my_pub FOR TABLE users, orders;

-- Subscriber
CREATE SUBSCRIPTION my_sub 
CONNECTION 'host=publisher_host dbname=mydb' 
PUBLICATION my_pub;
```

#### 3. **Synchronous Replication**
```sql
-- Ensure data is written to standby
synchronous_commit = on
synchronous_standby_names = 'standby1,standby2'
```

### High Availability Solutions

#### Connection Pooling
```yaml
# PgBouncer configuration
[databases]
mydb = host=localhost dbname=mydb

[pgbouncer]
pool_mode = transaction
max_client_conn = 1000
default_pool_size = 25
```

#### Load Balancing
- **HAProxy** - TCP load balancing
- **pgpool-II** - Connection pooling + load balancing
- **Patroni** - HA solution with automatic failover
- **repmgr** - Replication manager

### Failover Strategies

#### Automatic Failover
```bash
# Using pg_auto_failover
pg_autoctl create monitor
pg_autoctl create postgres --monitor --auth trust
pg_autoctl run
```

#### Manual Failover
```bash
# Promote standby to primary
pg_ctl promote -D /var/lib/postgresql/data

# Or using SQL
SELECT pg_promote();
```

---

## 🔒 Security

### Authentication Methods

#### pg_hba.conf Configuration
```conf
# TYPE  DATABASE    USER        ADDRESS         METHOD
local   all         postgres                    peer
host    all         all         127.0.0.1/32    md5
host    all         all         ::1/128         md5
host    mydb        myuser      192.168.1.0/24  scram-sha-256
```

#### Authentication Types
- **trust** - No authentication
- **peer** - OS user mapping
- **md5** - Password with MD5
- **scram-sha-256** - Secure password (recommended)
- **cert** - SSL certificate
- **ldap** - LDAP authentication

### User & Role Management

#### Creating Users and Roles
```sql
-- Create user
CREATE USER app_user WITH PASSWORD 'secure_password';

-- Create role
CREATE ROLE read_only;
CREATE ROLE read_write;

-- Grant privileges
GRANT SELECT ON ALL TABLES IN SCHEMA public TO read_only;
GRANT ALL PRIVILEGES ON DATABASE mydb TO admin_user;

-- Role inheritance
CREATE ROLE dev_team;
GRANT read_write TO dev_team;
CREATE USER developer IN ROLE dev_team;
```

#### Row-Level Security (RLS)
```sql
-- Enable RLS
ALTER TABLE orders ENABLE ROW LEVEL SECURITY;

-- Create policies
CREATE POLICY user_orders ON orders
    FOR ALL
    TO application_users
    USING (user_id = current_user_id());

-- Force RLS for table owner
ALTER TABLE orders FORCE ROW LEVEL SECURITY;
```

### Encryption

#### SSL/TLS Configuration
```conf
# postgresql.conf
ssl = on
ssl_cert_file = 'server.crt'
ssl_key_file = 'server.key'
ssl_ca_file = 'root.crt'
```

#### Data Encryption
```sql
-- Using pgcrypto extension
CREATE EXTENSION pgcrypto;

-- Encrypt data
INSERT INTO users (email, password) 
VALUES ('user@example.com', crypt('password', gen_salt('bf')));

-- Verify password
SELECT * FROM users 
WHERE email = 'user@example.com' 
AND password = crypt('password', password);
```

### Security Best Practices
- ✅ Use strong passwords
- ✅ Enable SSL/TLS
- ✅ Implement least privilege
- ✅ Regular security updates
- ✅ Audit logging
- ✅ Network isolation
- ✅ Regular backups

---

## 📊 JSON & NoSQL Features

### JSON vs JSONB

#### JSON Type
```sql
-- Stores exact copy of input
-- Preserves formatting
-- No indexing support
-- Slower processing

CREATE TABLE logs (
    id SERIAL PRIMARY KEY,
    data JSON
);
```

#### JSONB Type (Recommended)
```sql
-- Binary storage
-- No duplicate keys
-- Supports indexing
-- Faster processing

CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    data JSONB
);
```

### JSON Operations

#### Basic Operations
```sql
-- Insert JSON
INSERT INTO products (data) VALUES 
('{"name": "Laptop", "price": 999, "specs": {"ram": "16GB", "cpu": "i7"}}');

-- Query JSON
SELECT data->>'name' as name FROM products;
SELECT data->'specs'->>'ram' as ram FROM products;

-- Query with conditions
SELECT * FROM products WHERE data->>'price'::int > 500;
SELECT * FROM products WHERE data->'specs' ? 'ram';
```

#### JSON Operators
```sql
-> 	 -- Get JSON object field
->>  -- Get JSON object field as text
#>   -- Get JSON object at path
#>>  -- Get JSON object at path as text
@>   -- Contains
<@   -- Is contained by
?    -- Key exists
?|   -- Any key exists
?&   -- All keys exist
```

#### JSON Functions
```sql
-- jsonb_set - Update value
UPDATE products 
SET data = jsonb_set(data, '{price}', '899');

-- jsonb_insert - Insert value
UPDATE products 
SET data = jsonb_insert(data, '{specs,storage}', '"512GB"');

-- jsonb_pretty - Format JSON
SELECT jsonb_pretty(data) FROM products;

-- jsonb_agg - Aggregate to JSON array
SELECT jsonb_agg(data) FROM products WHERE data->>'category' = 'Electronics';
```

### JSON Indexing

#### GIN Index for JSONB
```sql
-- Index entire JSONB column
CREATE INDEX idx_products_data ON products USING GIN (data);

-- Index specific path
CREATE INDEX idx_products_specs ON products USING GIN ((data->'specs'));

-- Use jsonb_path_ops for better performance
CREATE INDEX idx_products_data_path ON products USING GIN (data jsonb_path_ops);
```

---

## 🔧 Extensions & Advanced Features

### Popular Extensions

#### 1. **PostGIS** - Spatial Database
```sql
CREATE EXTENSION postgis;

-- Spatial data types
CREATE TABLE locations (
    id SERIAL PRIMARY KEY,
    name TEXT,
    geom GEOMETRY(Point, 4326)
);

-- Spatial queries
SELECT * FROM locations 
WHERE ST_DWithin(geom, ST_MakePoint(-73.9857, 40.7484), 1000);
```

#### 2. **pg_trgm** - Trigram Matching
```sql
CREATE EXTENSION pg_trgm;

-- Fuzzy text search
SELECT * FROM users 
WHERE name % 'John Doe';  -- Similar to

-- Index for fast similarity search
CREATE INDEX idx_users_name_trgm ON users USING GIN (name gin_trgm_ops);
```

#### 3. **uuid-ossp** - UUID Generation
```sql
CREATE EXTENSION "uuid-ossp";

-- Generate UUID
SELECT uuid_generate_v4();

-- Use in table
CREATE TABLE events (
    id UUID DEFAULT uuid_generate_v4() PRIMARY KEY,
    name TEXT
);
```

#### 4. **hstore** - Key-Value Store
```sql
CREATE EXTENSION hstore;

-- Create hstore column
ALTER TABLE products ADD COLUMN attributes HSTORE;

-- Insert data
UPDATE products 
SET attributes = 'color=>red,size=>medium,weight=>2kg';

-- Query hstore
SELECT * FROM products WHERE attributes->'color' = 'red';
```

### Full-Text Search

#### Text Search Configuration
```sql
-- Create text search column
ALTER TABLE articles ADD COLUMN tsv TSVECTOR;

-- Update tsvector
UPDATE articles 
SET tsv = to_tsvector('english', title || ' ' || body);

-- Create index
CREATE INDEX idx_articles_tsv ON articles USING GIN (tsv);

-- Search
SELECT * FROM articles 
WHERE tsv @@ to_tsquery('english', 'postgresql & performance');
```

#### Search Ranking
```sql
SELECT title, 
       ts_rank(tsv, query) AS rank
FROM articles,
     to_tsquery('english', 'database') query
WHERE tsv @@ query
ORDER BY rank DESC;
```

### Table Partitioning

#### Range Partitioning
```sql
-- Parent table
CREATE TABLE sales (
    id SERIAL,
    sale_date DATE,
    amount DECIMAL
) PARTITION BY RANGE (sale_date);

-- Create partitions
CREATE TABLE sales_2023_q1 PARTITION OF sales
    FOR VALUES FROM ('2023-01-01') TO ('2023-04-01');
    
CREATE TABLE sales_2023_q2 PARTITION OF sales
    FOR VALUES FROM ('2023-04-01') TO ('2023-07-01');
```

#### List Partitioning
```sql
CREATE TABLE customers (
    id SERIAL,
    name TEXT,
    country TEXT
) PARTITION BY LIST (country);

CREATE TABLE customers_usa PARTITION OF customers
    FOR VALUES IN ('USA', 'Canada');
    
CREATE TABLE customers_europe PARTITION OF customers
    FOR VALUES IN ('UK', 'Germany', 'France');
```

---

## 📈 Administration & Monitoring

### Configuration Management

#### Key Configuration Parameters
```conf
# Memory
shared_buffers = 256MB          # 25% of RAM
work_mem = 4MB                  # Per operation
maintenance_work_mem = 64MB     # For VACUUM, CREATE INDEX

# Checkpoint
checkpoint_timeout = 5min
checkpoint_completion_target = 0.7

# WAL
wal_buffers = 16MB
wal_level

```conf
wal_level = replica
max_wal_size = 1GB
min_wal_size = 80MB

# Query Planning
random_page_cost = 1.1          # For SSD
effective_cache_size = 4GB      # Total memory for caching

# Logging
log_statement = 'all'           # none, ddl, mod, all
log_duration = on
log_min_duration_statement = 1000  # Log queries > 1s

# Connections
max_connections = 100
```

### System Catalogs

#### Important System Tables
```sql
-- Database information
SELECT * FROM pg_database;

-- Table information
SELECT * FROM pg_tables WHERE schemaname = 'public';

-- Index information
SELECT * FROM pg_indexes WHERE tablename = 'users';

-- User information
SELECT * FROM pg_user;

-- Current activity
SELECT * FROM pg_stat_activity;

-- Table statistics
SELECT * FROM pg_stat_user_tables;

-- Index usage
SELECT * FROM pg_stat_user_indexes;
```

### Performance Monitoring

#### Key Monitoring Queries
```sql
-- Long running queries
SELECT pid, age(clock_timestamp(), query_start), usename, query 
FROM pg_stat_activity 
WHERE query != '<IDLE>' AND query NOT ILIKE '%pg_stat_activity%' 
ORDER BY query_start DESC;

-- Table bloat
SELECT schemaname, tablename, 
       pg_size_pretty(pg_total_relation_size(schemaname||'.'||tablename)) AS size,
       n_live_tup, n_dead_tup,
       round(n_dead_tup::numeric / NULLIF(n_live_tup, 0), 4) AS dead_ratio
FROM pg_stat_user_tables
ORDER BY n_dead_tup DESC;

-- Index usage
SELECT schemaname, tablename, indexname, idx_scan, idx_tup_read
FROM pg_stat_user_indexes
ORDER BY idx_scan;

-- Cache hit ratio
SELECT 
    sum(heap_blks_hit) / nullif(sum(heap_blks_hit) + sum(heap_blks_read), 0) AS cache_hit_ratio
FROM pg_statio_user_tables;
```

### Maintenance Tasks

#### Regular Maintenance
```sql
-- VACUUM (should run automatically)
VACUUM ANALYZE;  -- Clean and update statistics

-- REINDEX for bloated indexes
REINDEX INDEX idx_users_email;
REINDEX TABLE users;
REINDEX DATABASE mydb;

-- Update table statistics
ANALYZE users;

-- Reset statistics
SELECT pg_stat_reset();
```

### Connection Management

#### Connection Pooling Best Practices
```yaml
# PgBouncer configuration
[databases]
mydb = host=localhost port=5432 dbname=mydb

[pgbouncer]
listen_port = 6432
listen_addr = *
auth_type = md5
auth_file = /etc/pgbouncer/userlist.txt
pool_mode = transaction
max_client_conn = 1000
default_pool_size = 25
min_pool_size = 5
reserve_pool_size = 5
```

---

## 💡 Common Interview Questions

### Basic Level

1. **What is PostgreSQL?**
   - Object-relational database system
   - ACID compliant
   - Open source
   - Extensible

2. **ACID Properties?**
   - Atomicity - All or nothing
   - Consistency - Valid states only
   - Isolation - Concurrent execution
   - Durability - Survives failures

3. **Primary Key vs Unique Key?**
   - Primary: Not null, one per table, clustered
   - Unique: Can be null, multiple per table

4. **DELETE vs TRUNCATE?**
   - DELETE: Row by row, logged, can rollback, triggers fire
   - TRUNCATE: Remove all rows, minimal logging, faster, no triggers

5. **Types of Indexes?**
   - B-Tree, Hash, GIN, GiST, BRIN, SP-GiST

### Intermediate Level

1. **Explain MVCC**
   - Multi-Version Concurrency Control
   - No read locks
   - Multiple row versions
   - Readers don't block writers

2. **What is a View?**
   ```sql
   -- Simple view
   CREATE VIEW active_users AS
   SELECT * FROM users WHERE active = true;
   
   -- Materialized view
   CREATE MATERIALIZED VIEW sales_summary AS
   SELECT date, SUM(amount) FROM sales GROUP BY date;
   ```

3. **Explain Normalization**
   - 1NF: Atomic values, unique rows
   - 2NF: No partial dependencies
   - 3NF: No transitive dependencies
   - BCNF: Every determinant is a key

4. **What is a Transaction?**
   - Unit of work
   - ACID properties
   - BEGIN, COMMIT, ROLLBACK
   - Isolation levels

5. **Index Selection Criteria?**
   - Cardinality
   - Query patterns
   - Write vs read ratio
   - Column selectivity

### Advanced Level

1. **Query Optimization Techniques?**
   - Proper indexing
   - EXPLAIN ANALYZE
   - Statistics update
   - Query rewriting
   - Partitioning

2. **Replication vs Clustering?**
   - Replication: Data copies, read scaling
   - Clustering: Shared storage, high availability

3. **Explain Vacuum Process**
   - Removes dead tuples
   - Updates statistics
   - Prevents transaction ID wraparound
   - Autovacuum vs manual

4. **Connection Pooling Benefits?**
   - Reduced connection overhead
   - Better resource utilization
   - Connection reuse
   - Improved performance

5. **Partitioning Strategies?**
   - Range: Time-based data
   - List: Categorical data
   - Hash: Even distribution
   - Benefits: Performance, maintenance

### Scenario-Based Questions

1. **Slow Query - How to Debug?**
   ```sql
   -- Check execution plan
   EXPLAIN (ANALYZE, BUFFERS) SELECT...;
   
   -- Check missing indexes
   -- Update statistics
   -- Check table bloat
   -- Review query logic
   ```

2. **High Database Load - What to Check?**
   - Active connections
   - Long-running queries
   - Lock contention
   - IO wait
   - Memory usage

3. **Design a Schema for E-commerce**
   ```sql
   -- Users, Products, Orders, OrderItems
   -- Proper relationships
   -- Indexes on foreign keys
   -- Consider partitioning for orders
   ```

4. **Implement Audit Trail**
   ```sql
   -- Trigger-based
   -- Separate audit tables
   -- Include timestamp, user, operation
   -- Consider performance impact
   ```

5. **Handle Concurrent Updates**
   - Use appropriate isolation level
   - Implement optimistic locking
   - Use SELECT FOR UPDATE
   - Handle deadlocks

---

## ✅ Best Practices

### Schema Design
- 📋 Use proper data types
- 📋 Normalize appropriately (3NF usually enough)
- 📋 Use constraints (FK, CHECK, NOT NULL)
- 📋 Create indexes on foreign keys
- 📋 Avoid EAV (Entity-Attribute-Value) pattern

### Query Writing
- ✍️ Use explicit JOINs over implicit
- ✍️ Avoid SELECT *
- ✍️ Use LIMIT for large results
- ✍️ Parameterize queries (prevent SQL injection)
- ✍️ Use CTEs for readability

### Indexing Strategy
- 🔍 Index foreign keys
- 🔍 Index columns in WHERE, ORDER BY, GROUP BY
- 🔍 Consider covering indexes
- 🔍 Monitor index usage
- 🔍 Remove unused indexes

### Performance
- ⚡ Regular VACUUM and ANALYZE
- ⚡ Monitor slow queries
- ⚡ Use connection pooling
- ⚡ Partition large tables
- ⚡ Archive old data

### Security
- 🔒 Use least privilege principle
- 🔒 Strong passwords
- 🔒 Enable SSL/TLS
- 🔒 Regular security updates
- 🔒 Audit sensitive operations

---

## 🔧 Troubleshooting

### Common Issues

#### 1. **Connection Refused**
```bash
# Check if PostgreSQL is running
systemctl status postgresql

# Check listening address
grep listen_addresses postgresql.conf

# Check pg_hba.conf
```

#### 2. **Out of Memory**
```sql
-- Check memory settings
SHOW shared_buffers;
SHOW work_mem;

-- Reduce work_mem for session
SET work_mem = '64MB';
```

#### 3. **Disk Space Issues**
```sql
-- Check database sizes
SELECT pg_database.datname,
       pg_size_pretty(pg_database_size(pg_database.datname)) AS size
FROM pg_database;

-- Find large tables
SELECT schemaname, tablename,
       pg_size_pretty(pg_total_relation_size(schemaname||'.'||tablename)) AS size
FROM pg_tables
ORDER BY pg_total_relation_size(schemaname||'.'||tablename) DESC;
```

#### 4. **Lock Contention**
```sql
-- View current locks
SELECT * FROM pg_locks WHERE NOT granted;

-- Find blocking queries
SELECT blocked_locks.pid AS blocked_pid,
       blocking_locks.pid AS blocking_pid,
       blocked_activity.query AS blocked_query,
       blocking_activity.query AS blocking_query
FROM pg_catalog.pg_locks blocked_locks
JOIN pg_catalog.pg_stat_activity blocked_activity ON blocked_activity.pid = blocked_locks.pid
JOIN pg_catalog.pg_locks blocking_locks ON blocking_locks.locktype = blocked_locks.locktype
JOIN pg_catalog.pg_stat_activity blocking_activity ON blocking_activity.pid = blocking_locks.pid
WHERE NOT blocked_locks.granted;
```

### Performance Tuning Checklist
- ✓ Update PostgreSQL version
- ✓ Configure memory settings
- ✓ Enable query logging
- ✓ Regular maintenance (VACUUM, ANALYZE)
- ✓ Monitor slow queries
- ✓ Review and optimize indexes
- ✓ Consider partitioning
- ✓ Use connection pooling
- ✓ Archive old data
- ✓ Monitor system resources

---

## 📚 Resources

### Official Documentation
- [PostgreSQL Documentation](https://www.postgresql.org/docs/)
- [PostgreSQL Wiki](https://wiki.postgresql.org/)
- [PostgreSQL Tutorial](https://www.postgresqltutorial.com/)

### Books
- "PostgreSQL: Up and Running" - Practical guide
- "Mastering PostgreSQL" - Advanced techniques
- "PostgreSQL High Performance" - Optimization

### Tools
- **pgAdmin** - GUI administration tool
- **psql** - Command-line interface
- **pg_stat_statements** - Query performance
- **pgBadger** - Log analyzer
- **pgtune** - Configuration wizard

### Community
- [PostgreSQL Mailing Lists](https://www.postgresql.org/list/)
- [PostgreSQL Slack](https://postgres
