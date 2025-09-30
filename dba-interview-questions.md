# DBA Interview Questions

# Câu hỏi phỏng vấn DBA

## Database Administration Fundamentals / Cơ bản về quản trị cơ sở dữ liệu

### 1. SQL Server Architecture / Kiến trúc SQL Server

**Câu hỏi / Question:** Giải thích SQL Server architecture? / Explain SQL Server architecture?

**Trả lời / Answer:**

- **Protocol Layer:** TDS (Tabular Data Stream)
- **Relational Engine:** Query optimization, execution
- **Storage Engine:** Data storage, retrieval, transaction management
- **SQLOS:** Operating system services

### 2. System Databases / Cơ sở dữ liệu hệ thống

**Câu hỏi / Question:** Các system databases và chức năng? / System databases and functions?

**Trả lời / Answer:**

- **master:** System configuration, login accounts
- **model:** Template for new databases
- **msdb:** SQL Server Agent, backup history
- **tempdb:** Temporary objects, worktables
- **distribution:** Replication metadata (if using replication)

## Installation & Configuration / Cài đặt và cấu hình

### 3. SQL Server Installation / Cài đặt SQL Server

**Câu hỏi / Question:** Best practices cho SQL Server installation? / Best practices for SQL Server installation?

**Trả lời / Answer:**

- Use service accounts instead of built-in accounts / Sử dụng tài khoản dịch vụ thay vì tài khoản tích hợp
- Configure proper file locations (data, log, tempdb) / Cấu hình vị trí tệp phù hợp (dữ liệu, nhật ký, tempdb)
- Set appropriate memory and CPU settings / Thiết lập bộ nhớ và CPU phù hợp
- Enable instant file initialization / Bật khởi tạo tệp tức thì
- Configure proper collation / Cấu hình collation phù hợp

### 4. Server Configuration / Cấu hình máy chủ

**Câu hỏi / Question:** Important server configuration options? / Tùy chọn cấu hình máy chủ quan trọng?

```sql
-- Memory configuration / Cấu hình bộ nhớ
sp_configure 'max server memory (MB)', 8192;
sp_configure 'min server memory (MB)', 1024;

-- Cost threshold for parallelism / Ngưỡng chi phí cho song song
sp_configure 'cost threshold for parallelism', 25;

-- Max degree of parallelism / Mức độ song song tối đa
sp_configure 'max degree of parallelism', 4;

RECONFIGURE;
```

## Backup & Recovery / Sao lưu và khôi phục

### 5. Backup Strategies / Chiến lược sao lưu

**Câu hỏi / Question:** Thiết kế backup strategy cho production? / Design backup strategy for production?

**Trả lời / Answer:**

- **Full Backup:** Daily, weekly / Sao lưu đầy đủ: Hàng ngày, hàng tuần
- **Differential Backup:** Every 4-6 hours / Sao lưu khác biệt: Mỗi 4-6 giờ
- **Log Backup:** Every 15-30 minutes / Sao lưu nhật ký: Mỗi 15-30 phút
- **File/Filegroup Backup:** For VLDBs / Sao lưu tệp/nhóm tệp: Cho VLDB
- **Copy-Only Backup:** For testing / Sao lưu chỉ sao chép: Cho kiểm thử

```sql
-- Automated backup procedure / Thủ tục sao lưu tự động
CREATE PROCEDURE sp_BackupDatabase
    @DatabaseName NVARCHAR(128),
    @BackupType CHAR(1) = 'F' -- F=Full, D=Differential, L=Log
AS
BEGIN
    DECLARE @BackupPath NVARCHAR(500);
    DECLARE @FileName NVARCHAR(500);

    SET @BackupPath = 'C:\Backup\';
    SET @FileName = @DatabaseName + '_' +
                   CASE @BackupType
                       WHEN 'F' THEN 'FULL'
                       WHEN 'D' THEN 'DIFF'
                       WHEN 'L' THEN 'LOG'
                   END + '_' +
                   CONVERT(VARCHAR(8), GETDATE(), 112) + '_' +
                   REPLACE(CONVERT(VARCHAR(8), GETDATE(), 108), ':', '') + '.bak';

    IF @BackupType = 'F'
        BACKUP DATABASE @DatabaseName TO DISK = @BackupPath + @FileName WITH COMPRESSION, CHECKSUM;
    ELSE IF @BackupType = 'D'
        BACKUP DATABASE @DatabaseName TO DISK = @BackupPath + @FileName WITH DIFFERENTIAL, COMPRESSION, CHECKSUM;
    ELSE IF @BackupType = 'L'
        BACKUP LOG @DatabaseName TO DISK = @BackupPath + @FileName WITH COMPRESSION, CHECKSUM;
END;
```

### 6. Disaster Recovery / Khôi phục thảm họa

**Câu hỏi / Question:** DR strategy và RTO/RPO? / DR strategy and RTO/RPO?

**Trả lời / Answer:**

- **RTO (Recovery Time Objective):** Thời gian tối đa để khôi phục / Maximum time to recover
- **RPO (Recovery Point Objective):** Lượng dữ liệu tối đa có thể mất / Maximum data loss acceptable
- **Strategies:** Always On AG, Log Shipping, Mirroring, Replication / Chiến lược: Always On AG, Log Shipping, Mirroring, Replication

## Performance Tuning / Tối ưu hóa hiệu suất

### 7. Performance Monitoring / Giám sát hiệu suất

**Câu hỏi / Question:** Tools và techniques để monitor performance? / Tools and techniques to monitor performance?

**Trả lời / Answer:**

- **SQL Server Profiler:** Real-time monitoring / Giám sát thời gian thực
- **Extended Events:** Lightweight tracing / Theo dõi nhẹ
- **Performance Monitor:** System metrics / Thống kê hệ thống
- **DMVs:** Dynamic Management Views / Khung nhìn quản lý động
- **Query Store:** Query performance history / Lịch sử hiệu suất truy vấn

```sql
-- Performance monitoring queries / Truy vấn giám sát hiệu suất
-- Top 10 queries by CPU / Top 10 truy vấn theo CPU
SELECT TOP 10
    qs.sql_handle,
    qs.execution_count,
    qs.total_worker_time,
    qs.total_elapsed_time,
    SUBSTRING(qt.text, (qs.statement_start_offset/2)+1,
        ((CASE qs.statement_end_offset
            WHEN -1 THEN DATALENGTH(qt.text)
            ELSE qs.statement_end_offset
        END - qs.statement_start_offset)/2) + 1) AS statement_text
FROM sys.dm_exec_query_stats qs
CROSS APPLY sys.dm_exec_sql_text(qs.sql_handle) qt
ORDER BY qs.total_worker_time DESC;

-- Wait statistics / Thống kê chờ
SELECT
    wait_type,
    waiting_tasks_count,
    wait_time_ms,
    signal_wait_time_ms,
    wait_time_ms - signal_wait_time_ms AS resource_wait_time_ms
FROM sys.dm_os_wait_stats
WHERE wait_time_ms > 0
ORDER BY wait_time_ms DESC;
```

### 8. Index Management / Quản lý chỉ mục

**Câu hỏi / Question:** Index maintenance strategy? / Chiến lược bảo trì chỉ mục?

```sql
-- Index fragmentation analysis / Phân tích phân mảnh chỉ mục
SELECT
    OBJECT_NAME(ind.OBJECT_ID) AS TableName,
    ind.name AS IndexName,
    indexstats.avg_fragmentation_in_percent,
    indexstats.page_count
FROM sys.dm_db_index_physical_stats(DB_ID(), NULL, NULL, NULL, NULL) indexstats
INNER JOIN sys.indexes ind ON ind.object_id = indexstats.object_id
    AND ind.index_id = indexstats.index_id
WHERE indexstats.avg_fragmentation_in_percent > 10
ORDER BY indexstats.avg_fragmentation_in_percent DESC;

-- Automated index maintenance / Bảo trì chỉ mục tự động
CREATE PROCEDURE sp_MaintainIndexes
AS
BEGIN
    DECLARE @TableName NVARCHAR(128);
    DECLARE @IndexName NVARCHAR(128);
    DECLARE @Fragmentation FLOAT;
    DECLARE @SQL NVARCHAR(MAX);

    DECLARE IndexCursor CURSOR FOR
    SELECT
        OBJECT_NAME(ind.OBJECT_ID),
        ind.name,
        indexstats.avg_fragmentation_in_percent
    FROM sys.dm_db_index_physical_stats(DB_ID(), NULL, NULL, NULL, NULL) indexstats
    INNER JOIN sys.indexes ind ON ind.object_id = indexstats.object_id
        AND ind.index_id = indexstats.index_id
    WHERE indexstats.avg_fragmentation_in_percent > 10;

    OPEN IndexCursor;
    FETCH NEXT FROM IndexCursor INTO @TableName, @IndexName, @Fragmentation;

    WHILE @@FETCH_STATUS = 0
    BEGIN
        IF @Fragmentation > 30
            SET @SQL = 'ALTER INDEX [' + @IndexName + '] ON [' + @TableName + '] REBUILD';
        ELSE
            SET @SQL = 'ALTER INDEX [' + @IndexName + '] ON [' + @TableName + '] REORGANIZE';

        EXEC sp_executesql @SQL;
        FETCH NEXT FROM IndexCursor INTO @TableName, @IndexName, @Fragmentation;
    END;

    CLOSE IndexCursor;
    DEALLOCATE IndexCursor;
END;
```

## Security / Bảo mật

### 9. Security Best Practices / Thực hành bảo mật tốt nhất

**Câu hỏi / Question:** Security measures cho SQL Server? / Security measures for SQL Server?

**Trả lời / Answer:**

- **Principle of Least Privilege** / Nguyên tắc đặc quyền tối thiểu
- **Encryption at rest and in transit** / Mã hóa khi nghỉ và truyền tải
- **Regular security audits** / Kiểm toán bảo mật thường xuyên
- **Strong password policies** / Chính sách mật khẩu mạnh
- **Network security (firewall, VPN)** / Bảo mật mạng (tường lửa, VPN)

```sql
-- Security audit / Kiểm toán bảo mật
-- Check for weak passwords / Kiểm tra mật khẩu yếu
SELECT name, type_desc, is_disabled
FROM sys.sql_logins
WHERE is_disabled = 0;

-- Check for orphaned users / Kiểm tra người dùng mồ côi
SELECT name, type_desc
FROM sys.database_principals
WHERE type = 'S'
AND name NOT IN (SELECT name FROM sys.server_principals);

-- Check for excessive permissions / Kiểm tra quyền quá mức
SELECT
    p.name AS PrincipalName,
    p.type_desc AS PrincipalType,
    o.name AS ObjectName,
    p.permission_name,
    p.state_desc
FROM sys.database_permissions p
INNER JOIN sys.objects o ON p.major_id = o.object_id
WHERE p.permission_name IN ('CONTROL', 'ALTER', 'TAKE OWNERSHIP');
```

### 10. Encryption / Mã hóa

**Câu hỏi / Question:** Implement encryption trong SQL Server? / Implement encryption in SQL Server?

```sql
-- Transparent Data Encryption (TDE) / Mã hóa dữ liệu trong suốt
-- Create database master key / Tạo khóa chính cơ sở dữ liệu
CREATE MASTER KEY ENCRYPTION BY PASSWORD = 'StrongPassword123!';

-- Create certificate / Tạo chứng chỉ
CREATE CERTIFICATE TDECertificate
WITH SUBJECT = 'TDE Certificate';

-- Create database encryption key / Tạo khóa mã hóa cơ sở dữ liệu
CREATE DATABASE ENCRYPTION KEY
WITH ALGORITHM = AES_256
ENCRYPTION BY SERVER CERTIFICATE TDECertificate;

-- Enable TDE / Bật TDE
ALTER DATABASE YourDatabase
SET ENCRYPTION ON;
```

## High Availability / Tính khả dụng cao

### 11. Always On Availability Groups / Nhóm khả dụng Always On

**Câu hỏi / Question:** Setup và manage Always On AG? / Setup and manage Always On AG?

```sql
-- Create availability group / Tạo nhóm khả dụng
CREATE AVAILABILITY GROUP [AG_Production]
FOR DATABASE [YourDatabase]
REPLICA ON
    'Server1' WITH (
        ENDPOINT_URL = 'TCP://Server1:5022',
        AVAILABILITY_MODE = SYNCHRONOUS_COMMIT,
        FAILOVER_MODE = AUTOMATIC,
        BACKUP_PRIORITY = 50,
        SECONDARY_ROLE(ALLOW_CONNECTIONS = READ_ONLY)
    ),
    'Server2' WITH (
        ENDPOINT_URL = 'TCP://Server2:5022',
        AVAILABILITY_MODE = SYNCHRONOUS_COMMIT,
        FAILOVER_MODE = AUTOMATIC,
        BACKUP_PRIORITY = 50,
        SECONDARY_ROLE(ALLOW_CONNECTIONS = READ_ONLY)
    );

-- Add listener / Thêm listener
ALTER AVAILABILITY GROUP [AG_Production]
ADD LISTENER 'AG_Listener' (
    WITH IP ((N'192.168.1.100', N'255.255.255.0')),
    PORT = 1433
);
```

### 12. Log Shipping / Gửi nhật ký

**Câu hỏi / Question:** Configure log shipping? / Cấu hình gửi nhật ký?

```sql
-- Primary server configuration / Cấu hình máy chủ chính
-- Backup job / Công việc sao lưu
BACKUP LOG [YourDatabase]
TO DISK = 'C:\LogShipping\YourDatabase.trn'
WITH COMPRESSION, CHECKSUM;

-- Secondary server configuration / Cấu hình máy chủ phụ
-- Restore job / Công việc khôi phục
RESTORE LOG [YourDatabase]
FROM DISK = 'C:\LogShipping\YourDatabase.trn'
WITH NORECOVERY;
```

## Automation & Maintenance / Tự động hóa và bảo trì

### 13. SQL Server Agent Jobs / Công việc SQL Server Agent

**Câu hỏi / Question:** Automated maintenance jobs? / Công việc bảo trì tự động?

```sql
-- Create maintenance job / Tạo công việc bảo trì
USE msdb;
GO

EXEC dbo.sp_add_job
    @job_name = N'Database Maintenance',
    @enabled = 1;

EXEC dbo.sp_add_jobstep
    @job_name = N'Database Maintenance',
    @step_name = N'Check Database Integrity',
    @subsystem = N'TSQL',
    @command = N'DBCC CHECKDB(''YourDatabase'') WITH NO_INFOMSGS;';

EXEC dbo.sp_add_schedule
    @schedule_name = N'Daily Maintenance',
    @freq_type = 4, -- Daily / Hàng ngày
    @freq_interval = 1,
    @active_start_time = 020000; -- 2:00 AM

EXEC dbo.sp_attach_schedule
    @job_name = N'Database Maintenance',
    @schedule_name = N'Daily Maintenance';
```

### 14. PowerShell Scripting / Script PowerShell

**Câu hỏi / Question:** PowerShell cho DBA tasks? / PowerShell for DBA tasks?

```powershell
# Backup all databases / Sao lưu tất cả cơ sở dữ liệu
$databases = Invoke-Sqlcmd -Query "SELECT name FROM sys.databases WHERE database_id > 4"
foreach ($db in $databases) {
    $backupPath = "C:\Backup\$($db.name)_$(Get-Date -Format 'yyyyMMdd_HHmmss').bak"
    $query = "BACKUP DATABASE [$($db.name)] TO DISK = '$backupPath' WITH COMPRESSION, CHECKSUM"
    Invoke-Sqlcmd -Query $query
}

# Monitor disk space / Giám sát dung lượng ổ đĩa
$disks = Get-WmiObject -Class Win32_LogicalDisk
foreach ($disk in $disks) {
    $freeSpace = [math]::Round($disk.FreeSpace / 1GB, 2)
    $totalSpace = [math]::Round($disk.Size / 1GB, 2)
    Write-Host "$($disk.DeviceID) - Free: $freeSpace GB / Total: $totalSpace GB"
}
```

## Troubleshooting / Xử lý sự cố

### 15. Common Issues / Vấn đề thường gặp

**Câu hỏi / Question:** Troubleshoot common SQL Server issues? / Xử lý sự cố SQL Server thường gặp?

**Trả lời / Answer:**

- **Blocking/Deadlocks:** Use sp_who2, sys.dm_tran_locks / Chặn/Deadlock: Sử dụng sp_who2, sys.dm_tran_locks
- **Memory Pressure:** Check buffer pool, page life expectancy / Áp lực bộ nhớ: Kiểm tra buffer pool, page life expectancy
- **Disk I/O Issues:** Monitor disk queue length, latency / Vấn đề I/O ổ đĩa: Giám sát độ dài hàng đợi ổ đĩa, độ trễ
- **Tempdb Issues:** Check space, contention / Vấn đề Tempdb: Kiểm tra không gian, xung đột
- **Network Issues:** Check connectivity, latency / Vấn đề mạng: Kiểm tra kết nối, độ trễ

```sql
-- Blocking analysis / Phân tích chặn
SELECT
    s.session_id,
    s.login_name,
    s.host_name,
    s.program_name,
    r.command,
    r.status,
    r.wait_type,
    r.wait_time,
    r.blocking_session_id,
    t.text AS sql_text
FROM sys.dm_exec_sessions s
INNER JOIN sys.dm_exec_requests r ON s.session_id = r.session_id
CROSS APPLY sys.dm_exec_sql_text(r.sql_handle) t
WHERE r.blocking_session_id > 0;

-- Memory pressure / Áp lực bộ nhớ
SELECT
    counter_name,
    cntr_value
FROM sys.dm_os_performance_counters
WHERE counter_name IN (
    'Buffer cache hit ratio',
    'Page life expectancy',
    'Memory Grants Pending'
);
```

## Tips cho DBA Interview / Lời khuyên cho phỏng vấn DBA

1. **Understand SQL Server architecture deeply / Hiểu sâu về kiến trúc SQL Server**
2. **Know backup and recovery procedures / Biết quy trình sao lưu và khôi phục**
3. **Be familiar with performance tuning / Quen thuộc với tối ưu hóa hiệu suất**
4. **Understand security best practices / Hiểu thực hành bảo mật tốt nhất**
5. **Know high availability solutions / Biết giải pháp khả dụng cao**
6. **Have automation and scripting skills / Có kỹ năng tự động hóa và script**
7. **Be able to troubleshoot common issues / Có thể xử lý sự cố thường gặp**
8. **Stay updated with latest SQL Server features / Cập nhật tính năng SQL Server mới nhất**
