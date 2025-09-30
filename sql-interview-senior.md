# SQL Interview Questions - Senior Level

# Câu hỏi phỏng vấn SQL - Cấp độ Senior

## Database Architecture & Design / Kiến trúc và thiết kế cơ sở dữ liệu

### 1. Database Normalization / Chuẩn hóa cơ sở dữ liệu

**Câu hỏi / Question:** Giải thích các normal forms và khi nào denormalize? / Explain normal forms and when to denormalize?

**Mục đích / Purpose:** Normalization giúp loại bỏ data redundancy, đảm bảo data integrity, và tạo cơ sở dữ liệu có cấu trúc logic.
**Purpose:** Normalization helps eliminate data redundancy, ensure data integrity, and create logically structured databases.

**Tại sao quan trọng / Why important:**
- Giảm storage space / Reduce storage space
- Tránh anomalies khi insert/update/delete / Avoid insert/update/delete anomalies
- Duy trì consistency / Maintain consistency
- Dễ maintain và modify / Easy to maintain and modify

**Trả lời / Answer:**

- **1NF (First Normal Form):** 
  - **Yêu cầu:** Mỗi cell chỉ chứa một giá trị atomic, không có repeating groups
  - **Requirement:** Each cell contains only one atomic value, no repeating groups
  - **Ví dụ:** Thay vì `Phone: "123-456, 789-012"` → Tạo bảng riêng cho phone numbers

- **2NF (Second Normal Form):**
  - **Yêu cầu:** 1NF + không có partial dependencies (non-key attributes phụ thuộc toàn bộ primary key)
  - **Requirement:** 1NF + no partial dependencies (non-key attributes depend on entire primary key)
  - **Khi áp dụng:** Với composite primary keys
  - **When to apply:** With composite primary keys

- **3NF (Third Normal Form):**
  - **Yêu cầu:** 2NF + không có transitive dependencies (non-key attributes không phụ thuộc vào non-key attributes khác)
  - **Requirement:** 2NF + no transitive dependencies (non-key attributes don't depend on other non-key attributes)
  - **Ví dụ:** Employee table không nên chứa cả EmployeeID, DepartmentID, và DepartmentName

- **BCNF (Boyce-Codd Normal Form):**
  - **Yêu cầu:** 3NF + mọi determinant là candidate key
  - **Requirement:** 3NF + every determinant is a candidate key
  - **Khi cần:** Với complex business rules và multiple candidate keys

**Khi nào Denormalize / When to Denormalize:**

1. **Read-heavy workloads / Khối lượng công việc đọc nhiều:**
   - Data warehouses / Kho dữ liệu
   - Reporting systems / Hệ thống báo cáo
   - OLAP systems / Hệ thống OLAP

2. **Performance critical applications / Ứng dụng quan trọng về hiệu suất:**
   - Khi JOIN operations quá chậm / When JOIN operations are too slow
   - High-frequency queries / Truy vấn tần suất cao

3. **Real-world scenarios / Tình huống thực tế:**
   ```sql
   -- Normalized (slower for reporting)
   SELECT c.CustomerName, SUM(o.Amount)
   FROM Customer c
   JOIN Orders o ON c.CustomerID = o.CustomerID
   GROUP BY c.CustomerName;
   
   -- Denormalized (faster for reporting)
   SELECT CustomerName, SUM(Amount)
   FROM OrdersWithCustomer  -- Includes CustomerName
   GROUP BY CustomerName;
   ```

### 2. Partitioning Strategies / Chiến lược phân vùng

**Câu hỏi / Question:** Các loại partitioning và khi nào sử dụng? / Types of partitioning and when to use?

**Mục đích / Purpose:** Partitioning chia bảng lớn thành các phần nhỏ hơn để cải thiện performance, quản lý dễ dàng hơn, và tối ưu maintenance operations.
**Purpose:** Partitioning divides large tables into smaller parts to improve performance, easier management, and optimize maintenance operations.

**Tại sao cần Partitioning / Why need Partitioning:**
1. **Query Performance:** Query chỉ scan partition liên quan (partition elimination)
2. **Maintenance:** Backup, index rebuild có thể thực hiện theo partition
3. **Parallel Processing:** Multiple partitions có thể xử lý song song
4. **Data Management:** Dễ dàng archive/delete data cũ
5. **Storage:** Có thể đặt partitions trên different filegroups

**Các loại Partitioning / Types of Partitioning:**

#### 1. Range Partitioning / Phân vùng theo khoảng:
**Khi sử dụng / When to use:**
- Time-series data (orders by date, logs by timestamp)
- Sequential data (ID ranges, age groups)
- Historical data archiving

**Ưu điểm / Advantages:**
- Partition elimination for range queries
- Easy to add/remove old partitions
- Intuitive for time-based data

```sql
-- Range Partitioning / Phân vùng theo khoảng
CREATE PARTITION FUNCTION PF_DateRange (DATETIME)
AS RANGE RIGHT FOR VALUES ('2023-01-01', '2023-07-01', '2024-01-01');

CREATE PARTITION SCHEME PS_DateRange
AS PARTITION PF_DateRange
TO (FileGroup1, FileGroup2, FileGroup3, FileGroup4);

-- Ví dụ sử dụng / Usage example
CREATE TABLE Orders (
    OrderID INT,
    OrderDate DATETIME,
    CustomerID INT,
    Amount DECIMAL(10,2)
) ON PS_DateRange(OrderDate);
```

#### 2. Hash Partitioning / Phân vùng theo băm:
**Khi sử dụng / When to use:**
- Even data distribution needed
- No natural range boundaries
- Load balancing across partitions

**Ưu điểm / Advantages:**
- Even distribution of data
- Good for parallel processing
- Eliminates hotspots

```sql
-- Hash Partitioning (simulated) / Phân vùng theo băm (mô phỏng)
CREATE PARTITION FUNCTION PF_Hash (INT)
AS RANGE RIGHT FOR VALUES (0, 1, 2, 3);

-- Use computed column for hash
ALTER TABLE Customer
ADD HashValue AS (CustomerID % 4) PERSISTED;
```

#### 3. List Partitioning / Phân vùng theo danh sách:
**Khi sử dụng / When to use:**
- Categorical data (regions, departments, status)
- Known discrete values
- Business logic separation

```sql
-- List Partitioning example / Ví dụ phân vùng theo danh sách
CREATE PARTITION FUNCTION PF_Region (VARCHAR(50))
AS RANGE RIGHT FOR VALUES ('North', 'South', 'East', 'West');
```

**Best Practices / Thực hành tốt nhất:**

1. **Choose right partition key / Chọn partition key phù hợp:**
   - Should be in most WHERE clauses
   - Evenly distributed data
   - Rarely updated (to avoid partition moves)

2. **Monitor partition sizes / Giám sát kích thước partition:**
   ```sql
   SELECT 
       p.partition_number,
       f.name AS FileGroupName,
       p.rows,
       a.total_pages * 8 / 1024 AS SizeMB
   FROM sys.partitions p
   JOIN sys.allocation_units a ON p.partition_id = a.container_id
   JOIN sys.filegroups f ON a.data_space_id = f.data_space_id
   WHERE p.object_id = OBJECT_ID('Orders');
   ```

3. **Partition maintenance / Bảo trì partition:**
   ```sql
   -- Split partition / Tách partition
   ALTER PARTITION SCHEME PS_DateRange
   NEXT USED FileGroup5;
   
   ALTER PARTITION FUNCTION PF_DateRange()
   SPLIT RANGE ('2024-07-01');
   
   -- Merge partition / Gộp partition
   ALTER PARTITION FUNCTION PF_DateRange()
   MERGE RANGE ('2023-01-01');
   ```

## Advanced Performance Tuning / Tối ưu hóa hiệu suất nâng cao

### 3. Query Store / Cửa hàng truy vấn

**Câu hỏi / Question:** Làm thế nào để monitor và optimize query performance? / How to monitor and optimize query performance?

**Mục đích / Purpose:** Query Store là feature của SQL Server giúp capture và persist query execution plans và performance statistics để phân tích và optimize performance.
**Purpose:** Query Store is a SQL Server feature that captures and persists query execution plans and performance statistics for analysis and performance optimization.

**Tại sao cần Query Store / Why need Query Store:**

1. **Performance Regression Detection / Phát hiện suy giảm hiệu suất:**
   - Tự động phát hiện khi query performance giảm
   - So sánh performance giữa các time periods
   - Identify problematic queries

2. **Plan Forcing / Ép buộc kế hoạch:**
   - Force stable execution plans
   - Prevent plan regression
   - Override optimizer decisions when needed

3. **Historical Analysis / Phân tích lịch sử:**
   - Track performance over time
   - Understand workload patterns
   - Capacity planning

4. **A/B Testing / Kiểm tra A/B:**
   - Compare different query versions
   - Test index impact
   - Validate optimizations

**Khi nào sử dụng / When to use:**
- Production databases with performance issues
- Systems requiring performance stability
- Environments with frequent plan changes
- Applications with varying workloads

**Các thành phần chính / Main components:**

1. **Query Store Catalog Views:**
   - `sys.query_store_query` - Query information
   - `sys.query_store_plan` - Execution plans
   - `sys.query_store_runtime_stats` - Runtime statistics

2. **Query Store Settings:**
   - `OPERATION_MODE` - READ_WRITE, READ_ONLY, OFF
   - `DATA_FLUSH_INTERVAL_SECONDS` - How often to persist data
   - `STALE_QUERY_THRESHOLD_DAYS` - When to remove old data

```sql
-- Enable Query Store / Bật Query Store
ALTER DATABASE YourDatabase
SET QUERY_STORE = ON
(
    OPERATION_MODE = READ_WRITE,
    CLEANUP_POLICY = (STALE_QUERY_THRESHOLD_DAYS = 30),
    DATA_FLUSH_INTERVAL_SECONDS = 3000,
    MAX_STORAGE_SIZE_MB = 1024,
    INTERVAL_LENGTH_MINUTES = 60
);

-- Analyze query performance / Phân tích hiệu suất truy vấn
SELECT
    qs.query_id,
    qt.query_sql_text,
    qp.plan_id,
    rs.avg_duration / 1000.0 AS avg_duration_ms,
    rs.avg_cpu_time / 1000.0 AS avg_cpu_time_ms,
    rs.avg_logical_io_reads,
    rs.execution_count,
    rs.last_execution_time
FROM sys.query_store_query qs
INNER JOIN sys.query_store_query_text qt ON qs.query_text_id = qt.query_text_id
INNER JOIN sys.query_store_plan qp ON qs.query_id = qp.query_id
INNER JOIN sys.query_store_runtime_stats rs ON qp.plan_id = rs.plan_id
WHERE rs.avg_duration > 1000000  -- > 1 second
ORDER BY rs.avg_duration DESC;

-- Force a specific plan / Ép buộc một kế hoạch cụ thể
EXEC sp_query_store_force_plan @query_id = 123, @plan_id = 456;

-- Remove plan forcing / Loại bỏ ép buộc kế hoạch
EXEC sp_query_store_unforce_plan @query_id = 123, @plan_id = 456;

-- Find regressed queries / Tìm queries bị suy giảm
SELECT
    qs.query_id,
    qt.query_sql_text,
    qp.plan_id,
    rs_recent.avg_duration / 1000.0 AS recent_avg_duration_ms,
    rs_history.avg_duration / 1000.0 AS history_avg_duration_ms,
    (rs_recent.avg_duration - rs_history.avg_duration) / 1000.0 AS regression_ms
FROM sys.query_store_query qs
INNER JOIN sys.query_store_query_text qt ON qs.query_text_id = qt.query_text_id
INNER JOIN sys.query_store_plan qp ON qs.query_id = qp.query_id
INNER JOIN sys.query_store_runtime_stats rs_recent ON qp.plan_id = rs_recent.plan_id
INNER JOIN sys.query_store_runtime_stats rs_history ON qp.plan_id = rs_history.plan_id
WHERE rs_recent.runtime_stats_interval_id > rs_history.runtime_stats_interval_id
  AND rs_recent.avg_duration > rs_history.avg_duration * 1.5  -- 50% slower
ORDER BY regression_ms DESC;

-- Clean up Query Store / Dọn dẹp Query Store
ALTER DATABASE YourDatabase
SET QUERY_STORE CLEAR;
```

**Best Practices / Thực hành tốt nhất:**

1. **Monitoring / Giám sát:**
   - Regularly check Query Store size
   - Monitor for plan forcing effectiveness
   - Review top resource consuming queries

2. **Maintenance / Bảo trì:**
   - Set appropriate retention policies
   - Clean up old data periodically
   - Monitor storage usage

3. **Troubleshooting / Xử lý sự cố:**
   - Use for plan regression analysis
   - Identify parameter sniffing issues
   - Compare performance before/after changes

### 4. In-Memory OLTP / OLTP trong bộ nhớ

**Câu hỏi / Question:** Khi nào sử dụng In-Memory OLTP? / When to use In-Memory OLTP?

**Mục đích / Purpose:** In-Memory OLTP (Hekaton) là engine của SQL Server được tối ưu cho high-performance OLTP workloads bằng cách lưu trữ data trong memory và sử dụng lock-free algorithms.
**Purpose:** In-Memory OLTP (Hekaton) is a SQL Server engine optimized for high-performance OLTP workloads by storing data in memory and using lock-free algorithms.

**Tại sao cần In-Memory OLTP / Why need In-Memory OLTP:**

1. **Performance Benefits / Lợi ích hiệu suất:**
   - 5-20x faster than disk-based tables
   - Lock-free và latch-free access
   - Optimized for CPU efficiency
   - Reduced lock contention

2. **Concurrency Improvements / Cải thiện đồng thời:**
   - MVCC (Multi-Version Concurrency Control)
   - No blocking between readers and writers
   - Optimistic concurrency control

3. **Predictable Performance / Hiệu suất dự đoán được:**
   - Consistent response times
   - No I/O bottlenecks
   - Deterministic garbage collection

**Khi nào sử dụng / When to use:**

✅ **Good candidates / Ứng cử viên tốt:**
- High-frequency INSERT/UPDATE/DELETE operations
- Session state management
- Real-time analytics
- Shopping carts, gaming leaderboards
- IoT data ingestion
- Financial trading systems

❌ **Not suitable for / Không phù hợp cho:**
- Large tables (> few GB)
- Reporting workloads (mostly SELECTs)
- Ad-hoc queries
- Tables with many indexes

**Limitations / Hạn chế:**

1. **Feature Limitations:**
   - No FOREIGN KEY constraints
   - No CHECK constraints with subqueries
   - Limited data types (no LOB types)
   - No ALTER TABLE for schema changes

2. **Size Limitations:**
   - Memory-optimized data size should be < 2x physical RAM
   - Individual row size < 8060 bytes

```sql
-- Enable In-Memory OLTP / Bật In-Memory OLTP
-- 1. Add memory-optimized filegroup / Thêm filegroup tối ưu bộ nhớ
ALTER DATABASE YourDatabase
ADD FILEGROUP MemoryOptimized_FG CONTAINS MEMORY_OPTIMIZED_DATA;

ALTER DATABASE YourDatabase
ADD FILE (NAME = 'MemoryOptimized_File', 
          FILENAME = 'C:\Data\MemoryOptimized.ndf')
TO FILEGROUP MemoryOptimized_FG;

-- 2. Create memory-optimized table / Tạo bảng tối ưu bộ nhớ
CREATE TABLE InMemorySessionState
(
    SessionID UNIQUEIDENTIFIER NOT NULL PRIMARY KEY NONCLUSTERED HASH WITH (BUCKET_COUNT = 1000000),
    UserID INT NOT NULL,
    LastActivity DATETIME2 NOT NULL,
    SessionData NVARCHAR(4000),
    CreatedDate DATETIME2 NOT NULL DEFAULT SYSUTCDATETIME(),
    
    INDEX IX_UserID NONCLUSTERED (UserID),
    INDEX IX_LastActivity NONCLUSTERED (LastActivity)
)
WITH (MEMORY_OPTIMIZED = ON, DURABILITY = SCHEMA_AND_DATA);

-- 3. Create natively compiled stored procedure / Tạo stored procedure biên dịch gốc
CREATE PROCEDURE UpdateSessionState
    @SessionID UNIQUEIDENTIFIER,
    @UserID INT,
    @SessionData NVARCHAR(4000)
WITH NATIVE_COMPILATION, SCHEMABINDING
AS
BEGIN ATOMIC WITH (
    TRANSACTION_ISOLATION_LEVEL = SNAPSHOT, 
    LANGUAGE = 'us_english'
)
    UPDATE InMemorySessionState
    SET SessionData = @SessionData,
        LastActivity = SYSUTCDATETIME()
    WHERE SessionID = @SessionID;
    
    IF @@ROWCOUNT = 0
    BEGIN
        INSERT INTO InMemorySessionState (SessionID, UserID, SessionData, LastActivity)
        VALUES (@SessionID, @UserID, @SessionData, SYSUTCDATETIME());
    END
END;

-- 4. Monitor memory usage / Giám sát sử dụng bộ nhớ
SELECT 
    object_name,
    memory_allocated_for_table_kb,
    memory_allocated_for_indexes_kb,
    memory_allocated_for_table_kb + memory_allocated_for_indexes_kb AS total_memory_kb
FROM sys.dm_db_xtp_table_memory_stats
WHERE object_id = OBJECT_ID('InMemorySessionState');

-- 5. Performance comparison / So sánh hiệu suất
-- Traditional table
CREATE TABLE DiskBasedSessionState
(
    SessionID UNIQUEIDENTIFIER NOT NULL PRIMARY KEY,
    UserID INT NOT NULL,
    LastActivity DATETIME2 NOT NULL,
    SessionData NVARCHAR(4000),
    CreatedDate DATETIME2 NOT NULL DEFAULT SYSUTCDATETIME()
);

-- Compare INSERT performance / So sánh hiệu suất INSERT
DECLARE @StartTime DATETIME2 = SYSUTCDATETIME();
DECLARE @i INT = 1;

-- Test In-Memory table
WHILE @i <= 100000
BEGIN
    INSERT INTO InMemorySessionState (SessionID, UserID, SessionData)
    VALUES (NEWID(), @i, 'Sample session data');
    SET @i = @i + 1;
END

SELECT DATEDIFF(millisecond, @StartTime, SYSUTCDATETIME()) AS InMemoryInsertTime_ms;
```

**Best Practices / Thực hành tốt nhất:**

1. **Table Design / Thiết kế bảng:**
   ```sql
   -- Use appropriate index types / Sử dụng loại index phù hợp
   -- HASH indexes for equality lookups
   PRIMARY KEY NONCLUSTERED HASH (ID) WITH (BUCKET_COUNT = 2000000),
   
   -- NONCLUSTERED indexes for range queries
   INDEX IX_Date NONCLUSTERED (CreatedDate)
   ```

2. **Bucket Count Sizing / Định kích thước Bucket Count:**
   ```sql
   -- Monitor hash index efficiency / Giám sát hiệu quả hash index
   SELECT 
       i.name AS IndexName,
       h.total_bucket_count,
       h.empty_bucket_count,
       h.avg_chain_length,
       h.max_chain_length
   FROM sys.dm_db_xtp_hash_index_stats h
   JOIN sys.indexes i ON h.object_id = i.object_id AND h.index_id = i.index_id
   WHERE h.object_id = OBJECT_ID('InMemorySessionState');
   ```

3. **Memory Monitoring / Giám sát bộ nhớ:**
   ```sql
   -- Check overall In-Memory OLTP memory usage
   SELECT 
       type_desc,
       allocated_bytes / 1024 / 1024 AS allocated_mb,
       used_bytes / 1024 / 1024 AS used_mb
   FROM sys.dm_db_xtp_memory_consumers
   WHERE database_id = DB_ID();
   ```

## High Availability & Disaster Recovery / Tính khả dụng cao và khôi phục thảm họa

### 5. Always On Availability Groups / Nhóm khả dụng Always On

**Câu hỏi / Question:** Setup và configure Always On AG? / Setup and configure Always On AG?

**Mục đích / Purpose:** Always On Availability Groups cung cấp high availability và disaster recovery solution cho SQL Server databases bằng cách maintain multiple copies của database trên different servers.
**Purpose:** Always On Availability Groups provides high availability and disaster recovery solution for SQL Server databases by maintaining multiple copies of databases on different servers.

**Tại sao cần Always On AG / Why need Always On AG:**

1. **High Availability / Tính khả dụng cao:**
   - Automatic failover trong vài giây
   - Multiple secondary replicas
   - Minimal data loss (RPO ≈ 0)
   - Fast recovery time (RTO < 30 seconds)

2. **Disaster Recovery / Khôi phục thảm họa:**
   - Geographic distribution of replicas
   - Asynchronous replication for DR sites
   - Flexible failover options

3. **Read Workload Offloading / Giảm tải read workload:**
   - Readable secondary replicas
   - Backup offloading to secondaries
   - Reporting workload distribution

4. **Maintenance Benefits / Lợi ích bảo trì:**
   - Rolling upgrades
   - Patching without downtime
   - Index maintenance on secondaries

**Khi nào sử dụng / When to use:**

✅ **Best for / Tốt nhất cho:**
- Mission-critical applications requiring 99.9%+ uptime
- Multi-database applications
- Geographically distributed environments
- Applications requiring read scale-out
- Environments requiring rolling upgrades

❌ **Consider alternatives for / Cân nhắc thay thế cho:**
- Single database environments (consider Failover Clustering)
- Budget constraints (requires Enterprise Edition)
- Simple backup/restore is sufficient

**Architecture Components / Thành phần kiến trúc:**

1. **Primary Replica:** Read-write copy của database
2. **Secondary Replicas:** Read-only copies (up to 8 total replicas)
3. **Availability Group Listener:** Virtual network name for client connections
4. **WSFC (Windows Server Failover Clustering):** Underlying cluster infrastructure

**Configuration Steps / Các bước cấu hình:**

```sql
-- Step 1: Enable Always On / Bước 1: Bật Always On
-- (Must be done via SQL Server Configuration Manager or PowerShell)
-- Enable-SqlAlwaysOn -ServerInstance "Server1" -Force

-- Step 2: Create endpoints / Bước 2: Tạo endpoints
CREATE ENDPOINT [Hadr_endpoint]
    AS TCP (LISTENER_PORT = 5022, LISTENER_IP = ALL)
    FOR DATA_MIRRORING (
        ROLE = ALL,
        AUTHENTICATION = WINDOWS NEGOTIATE,
        ENCRYPTION = REQUIRED ALGORITHM AES
    );

-- Step 3: Create availability group / Bước 3: Tạo nhóm khả dụng
CREATE AVAILABILITY GROUP [AG_Production]
WITH (
    AUTOMATED_BACKUP_PREFERENCE = SECONDARY,
    DB_FAILOVER = ON,
    DTC_SUPPORT = NONE
)
FOR DATABASE [ProductionDB], [ReportingDB]
REPLICA ON
    'Server1' WITH (
        ENDPOINT_URL = 'TCP://Server1.domain.com:5022',
        AVAILABILITY_MODE = SYNCHRONOUS_COMMIT,
        FAILOVER_MODE = AUTOMATIC,
        BACKUP_PRIORITY = 30,
        READABLE_SECONDARY = READ_INTENT_ONLY
    ),
    'Server2' WITH (
        ENDPOINT_URL = 'TCP://Server2.domain.com:5022',
        AVAILABILITY_MODE = SYNCHRONOUS_COMMIT,
        FAILOVER_MODE = AUTOMATIC,
        BACKUP_PRIORITY = 30,
        READABLE_SECONDARY = YES
    ),
    'Server3' WITH (
        ENDPOINT_URL = 'TCP://Server3.domain.com:5022',
        AVAILABILITY_MODE = ASYNCHRONOUS_COMMIT,
        FAILOVER_MODE = MANUAL,
        BACKUP_PRIORITY = 70,
        READABLE_SECONDARY = YES
    );

-- Step 4: Create listener / Bước 4: Tạo listener
ALTER AVAILABILITY GROUP [AG_Production]
ADD LISTENER 'AG_Production_Listener' (
    WITH IP (
        ('192.168.1.100', '255.255.255.0'),
        ('10.0.0.100', '255.255.255.0')
    ),
    PORT = 1433
);

-- Step 5: Join secondary replicas / Bước 5: Tham gia replicas phụ
-- (Run on each secondary server)
ALTER AVAILABILITY GROUP [AG_Production] JOIN;

-- Step 6: Add databases to secondary / Bước 6: Thêm databases vào secondary
-- (Run on each secondary server after restoring backups)
ALTER DATABASE [ProductionDB] SET HADR AVAILABILITY GROUP = [AG_Production];
ALTER DATABASE [ReportingDB] SET HADR AVAILABILITY GROUP = [AG_Production];
```

**Monitoring và Management / Giám sát và quản lý:**

```sql
-- Monitor AG health / Giám sát tình trạng AG
SELECT 
    ag.name AS AGName,
    ar.replica_server_name,
    ar.availability_mode_desc,
    ar.failover_mode_desc,
    ars.role_desc,
    ars.operational_state_desc,
    ars.connected_state_desc,
    ars.synchronization_health_desc
FROM sys.availability_groups ag
JOIN sys.availability_replicas ar ON ag.group_id = ar.group_id
JOIN sys.dm_hadr_availability_replica_states ars ON ar.replica_id = ars.replica_id;

-- Monitor database synchronization / Giám sát đồng bộ database
SELECT 
    db_name(drs.database_id) AS DatabaseName,
    drs.synchronization_state_desc,
    drs.synchronization_health_desc,
    drs.log_send_queue_size,
    drs.log_send_rate,
    drs.redo_queue_size,
    drs.redo_rate,
    drs.last_commit_time
FROM sys.dm_hadr_database_replica_states drs
WHERE drs.is_local = 1;

-- Manual failover / Chuyển đổi thủ công
ALTER AVAILABILITY GROUP [AG_Production] FAILOVER;

-- Add new database to AG / Thêm database mới vào AG
ALTER AVAILABILITY GROUP [AG_Production]
ADD DATABASE [NewDatabase];

-- Remove database from AG / Loại bỏ database khỏi AG
ALTER AVAILABILITY GROUP [AG_Production]
REMOVE DATABASE [OldDatabase];
```

**Best Practices / Thực hành tốt nhất:**

1. **Network Configuration / Cấu hình mạng:**
   - Dedicated network for AG traffic
   - Multiple network paths for redundancy
   - Proper DNS configuration

2. **Storage Considerations / Cân nhắc lưu trữ:**
   - Similar performance across replicas
   - Adequate disk space on all replicas
   - Transaction log sizing

3. **Monitoring / Giám sát:**
   ```sql
   -- Create alerts for AG health issues
   -- Setup monitoring for synchronization lag
   -- Monitor network latency between replicas
   ```

4. **Backup Strategy / Chiến lược sao lưu:**
   ```sql
   -- Configure backup preferences
   ALTER AVAILABILITY GROUP [AG_Production]
   MODIFY REPLICA ON 'Server3'
   WITH (BACKUP_PRIORITY = 80);
   ```

5. **Connection String Configuration:**
   ```
   -- For applications
   Server=AG_Production_Listener;Database=ProductionDB;
   
   -- For read-only access
   Server=AG_Production_Listener;Database=ProductionDB;ApplicationIntent=ReadOnly;
   ```

### 6. Backup Strategies / Chiến lược sao lưu

**Câu hỏi / Question:** Thiết kế backup strategy cho production database? / Design backup strategy for production database?

**Mục đích / Purpose:** Backup strategy đảm bảo data protection, minimize data loss, và enable quick recovery trong trường hợp failures, corruption, hoặc disasters.
**Purpose:** Backup strategy ensures data protection, minimizes data loss, and enables quick recovery in case of failures, corruption, or disasters.

**Tại sao cần Backup Strategy / Why need Backup Strategy:**

1. **Data Protection / Bảo vệ dữ liệu:**
   - Hardware failures
   - Software corruption
   - Human errors
   - Natural disasters

2. **Business Continuity / Liên tục kinh doanh:**
   - Meet RTO (Recovery Time Objective)
   - Meet RPO (Recovery Point Objective)
   - Regulatory compliance
   - Audit requirements

3. **Operational Flexibility / Linh hoạt vận hành:**
   - Point-in-time recovery
   - Partial restores
   - Database migrations
   - Development/testing environments

**Recovery Models / Mô hình khôi phục:**

1. **Simple Recovery Model:**
   - **Khi sử dụng:** Development, staging, data warehouses
   - **Backup types:** Full và Differential only
   - **Data loss:** Since last backup
   - **Log management:** Auto-truncated

2. **Full Recovery Model:**
   - **Khi sử dụng:** Production OLTP systems
   - **Backup types:** Full, Differential, Log
   - **Data loss:** Minimal (last log backup)
   - **Log management:** Manual truncation after backup

3. **Bulk-Logged Recovery Model:**
   - **Khi sử dụng:** During bulk operations
   - **Backup types:** Full, Differential, Log
   - **Data loss:** Minimal, but bulk ops may cause larger loss
   - **Log management:** Minimal logging for bulk operations

**Backup Types và Strategy / Các loại sao lưu và chiến lược:**

```sql
-- 1. Full Backup / Sao lưu đầy đủ
-- Frequency: Weekly (Sunday night)
-- Purpose: Complete database backup, base for differential/log restores
BACKUP DATABASE [ProductionDB]
TO DISK = 'C:\Backup\ProductionDB_Full_20250729.bak'
WITH 
    COMPRESSION,           -- Reduce backup size (30-50% smaller)
    CHECKSUM,             -- Verify backup integrity
    VERIFY_ONLY,          -- Optional: verify backup after creation
    STATS = 10,           -- Progress reporting every 10%
    NAME = 'ProductionDB Full Backup',
    DESCRIPTION = 'Weekly full backup for ProductionDB',
    EXPIREDATE = '2025-08-29';  -- Backup expiration

-- 2. Differential Backup / Sao lưu khác biệt
-- Frequency: Daily (except Sunday)
-- Purpose: Backup changes since last full backup
BACKUP DATABASE [ProductionDB]
TO DISK = 'C:\Backup\ProductionDB_Diff_20250729.bak'
WITH 
    DIFFERENTIAL,
    COMPRESSION,
    CHECKSUM,
    STATS = 10,
    NAME = 'ProductionDB Differential Backup',
    DESCRIPTION = 'Daily differential backup';

-- 3. Transaction Log Backup / Sao lưu nhật ký giao dịch
-- Frequency: Every 15 minutes
-- Purpose: Minimize data loss, enable point-in-time recovery
BACKUP LOG [ProductionDB]
TO DISK = 'C:\Backup\ProductionDB_Log_202507291200.trn'
WITH 
    COMPRESSION,
    CHECKSUM,
    NAME = 'ProductionDB Log Backup',
    DESCRIPTION = 'Transaction log backup';
```

**Production Backup Strategy Example / Ví dụ chiến lược sao lưu Production:**

```sql
-- Advanced backup strategy với multiple destinations
-- Strategy: Full (Weekly) + Differential (Daily) + Log (15 minutes)

-- 1. Full backup to multiple locations / Sao lưu đầy đủ đến nhiều vị trí
BACKUP DATABASE [ProductionDB]
TO DISK = 'C:\Backup\Local\ProductionDB_Full.bak',
   DISK = 'D:\Backup\Secondary\ProductionDB_Full.bak',
   URL = 'https://storageaccount.blob.core.windows.net/backups/ProductionDB_Full.bak'
WITH 
    COMPRESSION,
    CHECKSUM,
    FORMAT,               -- Overwrite existing backup set
    MEDIANAME = 'ProductionDB_BackupSet',
    NAME = 'ProductionDB Full Backup - Weekly',
    STATS = 5;

-- 2. Backup verification / Xác minh sao lưu
RESTORE VERIFYONLY 
FROM DISK = 'C:\Backup\Local\ProductionDB_Full.bak'
WITH CHECKSUM;

-- 3. Copy-only backup (doesn't affect backup chain)
-- Use for ad-hoc backups, migrations
BACKUP DATABASE [ProductionDB]
TO DISK = 'C:\Backup\Migration\ProductionDB_CopyOnly.bak'
WITH 
    COPY_ONLY,
    COMPRESSION,
    CHECKSUM;

-- 4. Automated backup script với error handling
DECLARE @BackupPath NVARCHAR(255);
DECLARE @FileName NVARCHAR(255);
DECLARE @SQL NVARCHAR(MAX);

SET @BackupPath = 'C:\Backup\';
SET @FileName = 'ProductionDB_' + 
                REPLACE(REPLACE(REPLACE(CONVERT(VARCHAR, GETDATE(), 121), ':', ''), '-', ''), ' ', '_') + 
                '.bak';

SET @SQL = 'BACKUP DATABASE [ProductionDB] TO DISK = ''' + @BackupPath + @FileName + ''' 
            WITH COMPRESSION, CHECKSUM, STATS = 10';

BEGIN TRY
    EXEC sp_executesql @SQL;
    PRINT 'Backup completed successfully: ' + @FileName;
END TRY
BEGIN CATCH
    PRINT 'Backup failed: ' + ERROR_MESSAGE();
    -- Send alert or log error
END CATCH;
```

**Backup Monitoring và Validation / Giám sát và xác thực sao lưu:**

```sql
-- 1. Check backup history / Kiểm tra lịch sử sao lưu
SELECT 
    bs.database_name,
    bs.backup_start_date,
    bs.backup_finish_date,
    bs.type,
    CASE bs.type
        WHEN 'D' THEN 'Full'
        WHEN 'I' THEN 'Differential'
        WHEN 'L' THEN 'Log'
    END AS backup_type,
    bs.backup_size / 1024 / 1024 AS backup_size_mb,
    bs.compressed_backup_size / 1024 / 1024 AS compressed_size_mb,
    bmf.physical_device_name
FROM msdb.dbo.backupset bs
JOIN msdb.dbo.backupmediafamily bmf ON bs.media_set_id = bmf.media_set_id
WHERE bs.database_name = 'ProductionDB'
  AND bs.backup_start_date >= DATEADD(day, -7, GETDATE())
ORDER BY bs.backup_start_date DESC;

-- 2. Check for missing backups / Kiểm tra sao lưu bị thiếu
WITH BackupGaps AS (
    SELECT 
        database_name,
        backup_finish_date,
        LEAD(backup_start_date) OVER (ORDER BY backup_start_date) AS next_backup_start,
        type
    FROM msdb.dbo.backupset
    WHERE database_name = 'ProductionDB' AND type = 'L'
)
SELECT *
FROM BackupGaps
WHERE DATEDIFF(minute, backup_finish_date, next_backup_start) > 20;  -- Gap > 20 minutes

-- 3. Backup integrity check / Kiểm tra tính toàn vẹn sao lưu
DECLARE @BackupFile NVARCHAR(255) = 'C:\Backup\ProductionDB_Full.bak';

RESTORE HEADERONLY FROM DISK = @BackupFile;  -- Check backup header
RESTORE FILELISTONLY FROM DISK = @BackupFile;  -- List files in backup
RESTORE VERIFYONLY FROM DISK = @BackupFile WITH CHECKSUM;  -- Verify integrity
```

**Recovery Scenarios / Kịch bản khôi phục:**

```sql
-- 1. Complete database restore / Khôi phục hoàn toàn database
-- Scenario: Database corruption, need full restore

-- Step 1: Tail-log backup (if possible)
BACKUP LOG [ProductionDB] 
TO DISK = 'C:\Backup\ProductionDB_TailLog.trn'
WITH NO_TRUNCATE, NORECOVERY;

-- Step 2: Restore full backup
RESTORE DATABASE [ProductionDB] 
FROM DISK = 'C:\Backup\ProductionDB_Full.bak'
WITH NORECOVERY, REPLACE;

-- Step 3: Restore differential backup
RESTORE DATABASE [ProductionDB] 
FROM DISK = 'C:\Backup\ProductionDB_Diff.bak'
WITH NORECOVERY;

-- Step 4: Restore log backups
RESTORE LOG [ProductionDB] 
FROM DISK = 'C:\Backup\ProductionDB_Log1.trn'
WITH NORECOVERY;

RESTORE LOG [ProductionDB] 
FROM DISK = 'C:\Backup\ProductionDB_TailLog.trn'
WITH RECOVERY;

-- 2. Point-in-time restore / Khôi phục theo thời điểm
RESTORE DATABASE [ProductionDB_PIT] 
FROM DISK = 'C:\Backup\ProductionDB_Full.bak'
WITH NORECOVERY, REPLACE,
MOVE 'ProductionDB' TO 'C:\Data\ProductionDB_PIT.mdf',
MOVE 'ProductionDB_Log' TO 'C:\Logs\ProductionDB_PIT.ldf';

RESTORE LOG [ProductionDB_PIT] 
FROM DISK = 'C:\Backup\ProductionDB_Log.trn'
WITH RECOVERY, STOPAT = '2025-07-29 14:30:00';
```

**Best Practices / Thực hành tốt nhất:**

1. **3-2-1 Rule:**
   - 3 copies of data
   - 2 different media types
   - 1 offsite copy

2. **Testing Strategy / Chiến lược kiểm tra:**
   - Regular restore tests
   - Automated backup validation
   - Disaster recovery drills

3. **Performance Optimization / Tối ưu hiệu suất:**
   ```sql
   -- Use multiple backup devices for faster backups
   BACKUP DATABASE [LargeDB]
   TO DISK = 'C:\Backup\LargeDB_1.bak',
      DISK = 'D:\Backup\LargeDB_2.bak',
      DISK = 'E:\Backup\LargeDB_3.bak'
   WITH COMPRESSION, MAXTRANSFERSIZE = 4194304, BUFFERCOUNT = 128;
   ```

4. **Retention Policies / Chính sách lưu trữ:**
   - Full backups: 3 months
   - Differential backups: 1 month  
   - Log backups: 2 weeks
   - Automated cleanup scripts

## Security & Compliance / Bảo mật và tuân thủ

### 7. Row-Level Security / Bảo mật cấp độ hàng

**Câu hỏi / Question:** Implement Row-Level Security? / Implement Row-Level Security?

```sql
-- Create security policy / Tạo chính sách bảo mật
CREATE SECURITY POLICY EmployeeSecurityPolicy
ADD FILTER PREDICATE dbo.fn_securitypredicate(DepartmentID)
ON dbo.Employee;

-- Security function / Hàm bảo mật
CREATE FUNCTION dbo.fn_securitypredicate(@DepartmentID INT)
RETURNS TABLE
WITH SCHEMABINDING
AS
RETURN SELECT 1 AS fn_securitypredicate_result
WHERE @DepartmentID = USER_ID();
```

### 8. Always Encrypted / Luôn mã hóa

**Câu hỏi / Question:** Setup Always Encrypted cho sensitive data? / Setup Always Encrypted for sensitive data?

```sql
-- Create column master key / Tạo khóa chính cột
CREATE COLUMN MASTER KEY CMK_Auto1
WITH (
    KEY_STORE_PROVIDER_NAME = 'MSSQL_CERTIFICATE_STORE',
    KEY_PATH = 'CurrentUser/My/A66C0F6DD70BDFF02B62D0F87E340288E6F9305'
);

-- Create column encryption key / Tạo khóa mã hóa cột
CREATE COLUMN ENCRYPTION KEY CEK_Auto1
WITH VALUES (
    COLUMN_MASTER_KEY = CMK_Auto1,
    ALGORITHM = 'RSA_OAEP',
    ENCRYPTED_VALUE = 0x01700000016C006F00630061006C006D0061006300680069006E0065002F006D0079002F0032006600610066006400380031003200310034003400340035006200310031003200650037003600630065002D0037003200610038002D0034003300650038002D0039006100310036002D0031003600640038003000390030003100630039003200
);
```

## Advanced Monitoring & Troubleshooting / Giám sát và xử lý sự cố nâng cao

### 9. Extended Events / Sự kiện mở rộng

**Câu hỏi / Question:** Setup monitoring với Extended Events? / Setup monitoring with Extended Events?

```sql
-- Create event session / Tạo phiên sự kiện
CREATE EVENT SESSION [Deadlock_Monitoring] ON SERVER
ADD EVENT sqlserver.xml_deadlock_report
ADD TARGET package0.event_file(SET filename=N'C:\XEvents\deadlock.xel')
WITH (MAX_MEMORY=4096 KB, EVENT_RETENTION_MODE=ALLOW_SINGLE_EVENT_LOSS);

-- Start session / Bắt đầu phiên
ALTER EVENT SESSION [Deadlock_Monitoring] ON SERVER STATE = START;
```

### 10. DMVs for Performance Analysis / DMV cho phân tích hiệu suất

**Câu hỏi / Question:** Sử dụng DMVs để analyze performance? / Use DMVs to analyze performance?

```sql
-- Check index usage / Kiểm tra sử dụng chỉ mục
SELECT
    OBJECT_NAME(i.object_id) AS TableName,
    i.name AS IndexName,
    ius.user_seeks,
    ius.user_scans,
    ius.user_lookups,
    ius.user_updates
FROM sys.dm_db_index_usage_stats ius
INNER JOIN sys.indexes i ON ius.object_id = i.object_id
    AND ius.index_id = i.index_id
WHERE ius.database_id = DB_ID();

-- Check wait statistics / Kiểm tra thống kê chờ
SELECT
    wait_type,
    waiting_tasks_count,
    wait_time_ms,
    max_wait_time_ms,
    signal_wait_time_ms
FROM sys.dm_os_wait_stats
WHERE wait_time_ms > 0
ORDER BY wait_time_ms DESC;
```

## Advanced SQL Features / Tính năng SQL nâng cao

### 11. Columnstore Indexes / Chỉ mục cột

**Câu hỏi / Question:** Khi nào sử dụng Columnstore Indexes? / When to use Columnstore Indexes?

```sql
-- Create clustered columnstore index / Tạo chỉ mục cột cụm
CREATE CLUSTERED COLUMNSTORE INDEX CCI_Employee
ON Employee;

-- Create nonclustered columnstore index / Tạo chỉ mục cột không cụm
CREATE NONCLUSTERED COLUMNSTORE INDEX NCCI_Employee
ON Employee (EmployeeID, FirstName, LastName, Salary);
```

### 12. Graph Database Features / Tính năng cơ sở dữ liệu đồ thị

**Câu hỏi / Question:** Implement graph relationships trong SQL Server? / Implement graph relationships in SQL Server?

```sql
-- Create node table / Tạo bảng nút
CREATE TABLE Person (
    ID INTEGER PRIMARY KEY,
    name VARCHAR(100)
) AS NODE;

-- Create edge table / Tạo bảng cạnh
CREATE TABLE Knows (
    since DATE
) AS EDGE;

-- Query graph data / Truy vấn dữ liệu đồ thị
SELECT
    Person1.name AS Person1Name,
    Person2.name AS Person2Name,
    Knows.since
FROM Person Person1, Knows, Person Person2
WHERE MATCH(Person1-(Knows)->Person2);
```

## Data Warehouse & ETL / Kho dữ liệu và ETL

### 13. SSIS Package Design / Thiết kế gói SSIS

**Câu hỏi / Question:** Thiết kế ETL process với SSIS? / Design ETL process with SSIS?

```sql
-- Example of data flow in SSIS / Ví dụ luồng dữ liệu trong SSIS
-- Source: Flat file or database / Nguồn: Tệp phẳng hoặc cơ sở dữ liệu
-- Transform: Data cleansing, validation / Biến đổi: Làm sạch dữ liệu, xác thực
-- Destination: Data warehouse / Đích: Kho dữ liệu

-- Control flow example / Ví dụ luồng điều khiển
-- 1. Truncate staging tables / Cắt bảng staging
-- 2. Load data to staging / Tải dữ liệu vào staging
-- 3. Transform and load to warehouse / Biến đổi và tải vào kho
-- 4. Update statistics / Cập nhật thống kê
-- 5. Send notification / Gửi thông báo
```

### 14. Data Warehouse Design / Thiết kế kho dữ liệu

**Câu hỏi / Question:** Thiết kế data warehouse schema? / Design data warehouse schema?

```sql
-- Star Schema example / Ví dụ lược đồ sao
-- Fact table / Bảng sự kiện
CREATE TABLE FactSales (
    SaleID INT PRIMARY KEY,
    ProductID INT,
    CustomerID INT,
    DateID INT,
    Quantity INT,
    Amount DECIMAL(10,2)
);

-- Dimension tables / Bảng chiều
CREATE TABLE DimProduct (
    ProductID INT PRIMARY KEY,
    ProductName VARCHAR(100),
    Category VARCHAR(50)
);

CREATE TABLE DimCustomer (
    CustomerID INT PRIMARY KEY,
    CustomerName VARCHAR(100),
    City VARCHAR(50)
);
```

## Advanced Administration / Quản trị nâng cao

### 15. Resource Governor / Bộ quản lý tài nguyên

**Câu hỏi / Question:** Configure Resource Governor? / Configure Resource Governor?

```sql
-- Create resource pool / Tạo nhóm tài nguyên
CREATE RESOURCE POOL Pool_Reporting
WITH (
    MIN_CPU_PERCENT = 20,
    MAX_CPU_PERCENT = 50,
    MIN_MEMORY_PERCENT = 20,
    MAX_MEMORY_PERCENT = 50
);

-- Create workload group / Tạo nhóm khối lượng công việc
CREATE WORKLOAD GROUP WG_Reporting
USING Pool_Reporting;

-- Create classifier function / Tạo hàm phân loại
CREATE FUNCTION dbo.ClassifierFunction()
RETURNS SYSNAME
WITH SCHEMABINDING
AS
BEGIN
    DECLARE @GroupName SYSNAME;

    IF SUSER_NAME() = 'ReportingUser'
        SET @GroupName = 'WG_Reporting';
    ELSE
        SET @GroupName = 'default';

    RETURN @GroupName;
END;
```

### 16. Database Maintenance / Bảo trì cơ sở dữ liệu

**Câu hỏi / Question:** Automated database maintenance? / Automated database maintenance?

```sql
-- Create maintenance plan / Tạo kế hoạch bảo trì
-- 1. Check database integrity / Kiểm tra tính toàn vẹn cơ sở dữ liệu
DBCC CHECKDB('YourDatabase') WITH NO_INFOMSGS;

-- 2. Update statistics / Cập nhật thống kê
UPDATE STATISTICS dbo.Employee WITH FULLSCAN;

-- 3. Rebuild/reorganize indexes / Xây dựng lại/tổ chức lại chỉ mục
ALTER INDEX ALL ON dbo.Employee REBUILD;

-- 4. Clean up old backup files / Dọn dẹp tệp sao lưu cũ
-- PowerShell script to remove old backups / Script PowerShell để xóa sao lưu cũ
```

## Tips cho Senior Level / Lời khuyên cho cấp độ Senior

1. **Deep understanding of SQL Server architecture / Hiểu sâu về kiến trúc SQL Server**
2. **Expert in performance tuning and optimization / Chuyên gia về tối ưu hóa hiệu suất**
3. **Experience with high availability solutions / Kinh nghiệm với giải pháp khả dụng cao**
4. **Knowledge of security best practices / Kiến thức về thực hành bảo mật tốt nhất**
5. **Ability to design complex database solutions / Khả năng thiết kế giải pháp cơ sở dữ liệu phức tạp**
6. **Experience with data warehouse and ETL processes / Kinh nghiệm với kho dữ liệu và quy trình ETL**
7. **Strong troubleshooting and problem-solving skills / Kỹ năng xử lý sự cố và giải quyết vấn đề mạnh mẽ**
8. **Understanding of business requirements and translating to technical solutions / Hiểu yêu cầu kinh doanh và chuyển đổi thành giải pháp kỹ thuật**
