# SQL Interview Questions - Middle Level

# Câu hỏi phỏng vấn SQL - Cấp độ Middle

## Advanced SQL Concepts / Khái niệm SQL nâng cao

### 1. Window Functions / Hàm cửa sổ

**Câu hỏi / Question:** Giải thích về Window Functions và cho ví dụ? / Explain Window Functions and give examples?

**Trả lời / Answer:** Window Functions cho phép thực hiện tính toán trên một tập hợp các dòng liên quan đến dòng hiện tại mà không làm giảm số dòng kết quả.
**Answer:** Window Functions allow performing calculations on a set of rows related to the current row without reducing the number of result rows.

```sql
-- Ví dụ: Tính lương trung bình của phòng ban cho mỗi nhân viên
-- Example: Calculate average salary of department for each employee
SELECT
    FirstName,
    LastName,
    DepartmentID,
    Salary,
    AVG(Salary) OVER (PARTITION BY DepartmentID) as AvgDeptSalary,
    ROW_NUMBER() OVER (PARTITION BY DepartmentID ORDER BY Salary DESC) as SalaryRank
FROM Employee;
```

### 2. Common Table Expressions (CTE) / Biểu thức bảng chung

**Câu hỏi / Question:** CTE là gì và khi nào sử dụng? / What is CTE and when to use it?

**Trả lời / Answer:** CTE là bảng tạm thời được định nghĩa trong phạm vi của một câu lệnh SELECT, INSERT, UPDATE, DELETE hoặc MERGE.
**Answer:** CTE is a temporary table defined within the scope of a SELECT, INSERT, UPDATE, DELETE or MERGE statement.

```sql
-- Ví dụ: Tìm nhân viên có lương cao thứ 2 trong mỗi phòng ban
-- Example: Find employee with 2nd highest salary in each department
WITH RankedEmployees AS (
    SELECT
        FirstName,
        LastName,
        DepartmentID,
        Salary,
        ROW_NUMBER() OVER (PARTITION BY DepartmentID ORDER BY Salary DESC) as Rank
    FROM Employee
)
SELECT * FROM RankedEmployees WHERE Rank = 2;
```

### 3. Recursive CTE / CTE đệ quy

**Câu hỏi / Question:** Recursive CTE hoạt động như thế nào? / How does Recursive CTE work?

**Trả lời / Answer:** Recursive CTE cho phép thực hiện truy vấn đệ quy, thường dùng cho hierarchical data.
**Answer:** Recursive CTE allows performing recursive queries, commonly used for hierarchical data.

```sql
-- Ví dụ: Tìm cấu trúc phân cấp nhân viên
-- Example: Find employee hierarchy
WITH EmployeeHierarchy AS (
    -- Anchor member / Thành viên neo
    SELECT EmployeeID, FirstName, LastName, ManagerID, 0 as Level
    FROM Employee
    WHERE ManagerID IS NULL

    UNION ALL

    -- Recursive member / Thành viên đệ quy
    SELECT e.EmployeeID, e.FirstName, e.LastName, e.ManagerID, eh.Level + 1
    FROM Employee e
    INNER JOIN EmployeeHierarchy eh ON e.ManagerID = eh.EmployeeID
)
SELECT * FROM EmployeeHierarchy;
```

### 4. Advanced CTE Scenarios / Kịch bản CTE nâng cao

**Câu hỏi / Question:** Cho ví dụ sử dụng Multiple CTE trong một query? / Give examples of using Multiple CTEs in a query?

**Trả lời / Answer:** Có thể sử dụng nhiều CTE trong cùng một câu lệnh bằng cách phân tách bằng dấu phẩy.
**Answer:** You can use multiple CTEs in the same statement by separating them with commas.

```sql
-- Ví dụ: Phân tích doanh số và xếp hạng nhân viên
-- Example: Sales analysis and employee ranking
WITH SalesData AS (
    SELECT
        EmployeeID,
        SUM(Amount) as TotalSales,
        COUNT(*) as OrderCount
    FROM Orders
    WHERE OrderDate >= '2024-01-01'
    GROUP BY EmployeeID
),
EmployeeInfo AS (
    SELECT
        EmployeeID,
        FirstName,
        LastName,
        DepartmentID
    FROM Employee
)
SELECT
    ei.FirstName,
    ei.LastName,
    sd.TotalSales,
    sd.OrderCount,
    RANK() OVER (ORDER BY sd.TotalSales DESC) as SalesRank
FROM EmployeeInfo ei
INNER JOIN SalesData sd ON ei.EmployeeID = sd.EmployeeID;
```

### 5. CTE vs Table Variables vs Temp Tables / So sánh CTE với Table Variables và Temp Tables

**Câu hỏi / Question:** So sánh CTE với Table Variables và Temp Tables? Khi nào sử dụng từng loại? / Compare CTE with Table Variables and Temp Tables? When to use each type?

**Trả lời / Answer:**

#### CTE (Common Table Expressions):

- **Ưu điểm / Advantages:**

  - Chỉ tồn tại trong phạm vi một câu lệnh / Only exists within one statement scope
  - Có thể đệ quy / Can be recursive
  - Dễ đọc và maintain / Easy to read and maintain
  - Không cần cleanup / No cleanup needed

- **Nhược điểm / Disadvantages:**
  - Không thể reuse trong multiple statements / Cannot reuse in multiple statements
  - Không có indexes / No indexes
  - Performance có thể không tối ưu với large datasets / Performance may not be optimal with large datasets

#### Table Variables (@table):

- **Ưu điểm / Advantages:**

  - Stored in memory (nếu nhỏ) / Stored in memory (if small)
  - Local scope / Phạm vi cục bộ
  - Không ghi log transactions / Not logged in transactions

- **Nhược điểm / Disadvantages:**
  - Không có statistics / No statistics
  - Không thể create indexes (except constraints) / Cannot create indexes (except constraints)
  - Kém performance với large data / Poor performance with large data

**Dataset Size Guidelines / Hướng dẫn kích thước dữ liệu:**

| Dataset Size / Kích thước | Row Count / Số dòng   | Memory Usage / Sử dụng bộ nhớ | Performance Impact / Ảnh hưởng hiệu suất |
| ------------------------- | --------------------- | ----------------------------- | ---------------------------------------- |
| **Very Small / Rất nhỏ**  | < 100 rows            | < 1 MB                        | Excellent / Xuất sắc                     |
| **Small / Nhỏ**           | 100 - 1,000 rows      | 1 - 10 MB                     | Good / Tốt                               |
| **Medium / Trung bình**   | 1,000 - 10,000 rows   | 10 - 100 MB                   | Moderate / Trung bình                    |
| **Large / Lớn**           | 10,000 - 100,000 rows | 100 MB - 1 GB                 | Poor / Kém                               |
| **Very Large / Rất lớn**  | > 100,000 rows        | > 1 GB                        | Very Poor / Rất kém                      |

**Factors Affecting Dataset Size / Các yếu tố ảnh hưởng đến kích thước dữ liệu:**

1. **Number of Columns / Số lượng cột:**
   - Nhiều cột = Dataset lớn hơn / More columns = Larger dataset
2. **Data Types / Kiểu dữ liệu:**

   - VARCHAR(MAX), NVARCHAR(MAX), VARBINARY(MAX) = Rất lớn / Very large
   - TEXT, NTEXT, IMAGE = Rất lớn / Very large
   - INT, DECIMAL, DATETIME = Trung bình / Medium
   - BIT, TINYINT = Nhỏ / Small

3. **Data Content / Nội dung dữ liệu:**
   - String length / Độ dài chuỗi
   - Binary data size / Kích thước dữ liệu nhị phân

```sql
-- Ví dụ Dataset Size Calculation / Example Dataset Size Calculation

-- Small Dataset Example (< 1,000 rows)
-- Suitable for Table Variable / Phù hợp cho Table Variable
DECLARE @SmallCustomers TABLE (
    CustomerID INT,
    CustomerName NVARCHAR(100),
    City NVARCHAR(50)
);
-- Estimated size: 1,000 rows × ~200 bytes = ~200 KB

-- Medium Dataset Example (1,000 - 10,000 rows)
-- Consider Temp Table / Cân nhắc Temp Table
CREATE TABLE #MediumOrders (
    OrderID INT,
    CustomerID INT,
    OrderDate DATETIME,
    Amount DECIMAL(10,2),
    Status NVARCHAR(20)
);
-- Estimated size: 10,000 rows × ~50 bytes = ~500 KB

-- Large Dataset Example (> 10,000 rows)
-- Use Temp Table with Indexes / Sử dụng Temp Table với Indexes
CREATE TABLE #LargeTransactions (
    TransactionID BIGINT,
    AccountID INT,
    TransactionDate DATETIME,
    Amount DECIMAL(15,2),
    Description NVARCHAR(500),
    CategoryID INT
);

CREATE INDEX IX_LargeTransactions_Date ON #LargeTransactions(TransactionDate);
CREATE INDEX IX_LargeTransactions_Account ON #LargeTransactions(AccountID);
-- Estimated size: 100,000 rows × ~600 bytes = ~60 MB
```

**Memory Thresholds / Ngưỡng bộ nhớ:**

- **Table Variables** tốt nhất khi < 8 MB (SQL Server memory page limit)
- **Table Variables** are best when < 8 MB (SQL Server memory page limit)

- **Temp Tables** nên sử dụng khi > 10 MB và cần indexes/statistics
- **Temp Tables** should be used when > 10 MB and need indexes/statistics

```sql
-- Test Dataset Size / Kiểm tra kích thước Dataset
SELECT
    OBJECT_NAME(object_id) as TableName,
    SUM(reserved_page_count) * 8.0 / 1024 as SizeMB,
    SUM(row_count) as RowCount
FROM sys.dm_db_partition_stats
WHERE object_id = OBJECT_ID('YourTableName')
GROUP BY object_id;

-- Estimate Table Variable Memory Usage / Ước tính sử dụng bộ nhớ Table Variable
-- Rule of thumb: RowCount × AvgRowSize (bytes)
-- Quy tắc: Số dòng × Kích thước dòng trung bình (bytes)
```

#### Temp Tables (#table):

- **Ưu điểm / Advantages:**

  - Có statistics / Has statistics
  - Có thể create indexes / Can create indexes
  - Tối ưu cho large datasets / Optimized for large datasets

- **Nhược điểm / Disadvantages:**
  - Cần cleanup / Needs cleanup
  - Overhead của TempDB / TempDB overhead
  - Ghi log transactions / Logged in transactions

**Dataset Size Guidelines / Hướng dẫn kích thước dữ liệu:**

| Dataset Size / Kích thước | Row Count / Số dòng   | Memory Usage / Sử dụng bộ nhớ | Performance Impact / Ảnh hưởng hiệu suất |
| ------------------------- | --------------------- | ----------------------------- | ---------------------------------------- |
| **Very Small / Rất nhỏ**  | < 100 rows            | < 1 MB                        | Excellent / Xuất sắc                     |
| **Small / Nhỏ**           | 100 - 1,000 rows      | 1 - 10 MB                     | Good / Tốt                               |
| **Medium / Trung bình**   | 1,000 - 10,000 rows   | 10 - 100 MB                   | Moderate / Trung bình                    |
| **Large / Lớn**           | 10,000 - 100,000 rows | 100 MB - 1 GB                 | Poor / Kém                               |
| **Very Large / Rất lớn**  | > 100,000 rows        | > 1 GB                        | Very Poor / Rất kém                      |

**Factors Affecting Dataset Size / Các yếu tố ảnh hưởng đến kích thước dữ liệu:**

1. **Number of Columns / Số lượng cột:**
   - Nhiều cột = Dataset lớn hơn / More columns = Larger dataset
2. **Data Types / Kiểu dữ liệu:**

   - VARCHAR(MAX), NVARCHAR(MAX), VARBINARY(MAX) = Rất lớn / Very large
   - TEXT, NTEXT, IMAGE = Rất lớn / Very large
   - INT, DECIMAL, DATETIME = Trung bình / Medium
   - BIT, TINYINT = Nhỏ / Small

3. **Data Content / Nội dung dữ liệu:**
   - String length / Độ dài chuỗi
   - Binary data size / Kích thước dữ liệu nhị phân

```sql
-- Ví dụ Dataset Size Calculation / Example Dataset Size Calculation

-- Small Dataset Example (< 1,000 rows)
-- Suitable for Temp Table / Phù hợp cho Temp Table
CREATE TABLE #SmallCustomers (
    CustomerID INT,
    CustomerName NVARCHAR(100),
    City NVARCHAR(50)
);
-- Estimated size: 1,000 rows × ~200 bytes = ~200 KB

-- Medium Dataset Example (1,000 - 10,000 rows)
-- Consider Temp Table / Cân nhắc Temp Table
CREATE TABLE #MediumOrders (
    OrderID INT,
    CustomerID INT,
    OrderDate DATETIME,
    Amount DECIMAL(10,2),
    Status NVARCHAR(20)
);
-- Estimated size: 10,000 rows × ~50 bytes = ~500 KB

-- Large Dataset Example (> 10,000 rows)
-- Use Temp Table with Indexes / Sử dụng Temp Table với Indexes
CREATE TABLE #LargeTransactions (
    TransactionID BIGINT,
    AccountID INT,
    TransactionDate DATETIME,
    Amount DECIMAL(15,2),
    Description NVARCHAR(500),
    CategoryID INT
);

CREATE INDEX IX_LargeTransactions_Date ON #LargeTransactions(TransactionDate);
CREATE INDEX IX_LargeTransactions_Account ON #LargeTransactions(AccountID);
-- Estimated size: 100,000 rows × ~600 bytes = ~60 MB
```

**Memory Thresholds / Ngưỡng bộ nhớ:**

- **Temp Tables** nên sử dụng khi > 10 MB và cần indexes/statistics
- **Temp Tables** should be used when > 10 MB and need indexes/statistics

```sql
-- Test Dataset Size / Kiểm tra kích thước Dataset
SELECT
    OBJECT_NAME(object_id) as TableName,
    SUM(reserved_page_count) * 8.0 / 1024 as SizeMB,
    SUM(row_count) as RowCount
FROM sys.dm_db_partition_stats
WHERE object_id = OBJECT_ID('YourTableName')
GROUP BY object_id;

-- Estimate Temp Table Memory Usage / Ước tính sử dụng bộ nhớ Temp Table
-- Rule of thumb: RowCount × AvgRowSize (bytes)
-- Quy tắc: Số dòng × Kích thước dòng trung bình (bytes)
```

### 6. CTE Performance Considerations / Những điều cần xem xét về hiệu suất CTE

**Câu hỏi / Question:** Những lưu ý về performance khi sử dụng CTE? / Performance considerations when using CTE?

**Trả lời / Answer:**

#### CTE Performance Tips / Mẹo tối ưu hiệu suất CTE:

1. **Materialization:** CTE không được materialize tự động / CTE is not automatically materialized
2. **Multiple References:** Nếu CTE được reference nhiều lần, nó sẽ được evaluate nhiều lần / If CTE is referenced multiple times, it will be evaluated multiple times
3. **Recursive CTE Limits:** Cần cẩn thận với MAXRECURSION option / Be careful with MAXRECURSION option

```sql
-- BAD: CTE được evaluate 2 lần / CTE evaluated twice
WITH ExpensiveCalculation AS (
    SELECT
        CustomerID,
        SUM(Amount) as TotalAmount,
        COUNT(*) as OrderCount,
        -- Complex calculations here / Tính toán phức tạp ở đây
        (SELECT AVG(Amount) FROM Orders o2 WHERE o2.CustomerID = o1.CustomerID) as AvgAmount
    FROM Orders o1
    GROUP BY CustomerID
)
SELECT ec1.CustomerID, ec1.TotalAmount
FROM ExpensiveCalculation ec1
INNER JOIN ExpensiveCalculation ec2 ON ec1.CustomerID = ec2.CustomerID -- BAD: Evaluated again!
WHERE ec2.OrderCount > 10;

-- GOOD: Sử dụng Temp Table cho multiple references / Use Temp Table for multiple references
CREATE TABLE #CustomerCalc AS
SELECT
    CustomerID,
    SUM(Amount) as TotalAmount,
    COUNT(*) as OrderCount,
    (SELECT AVG(Amount) FROM Orders o2 WHERE o2.CustomerID = o1.CustomerID) as AvgAmount
FROM Orders o1
GROUP BY CustomerID;

SELECT cc1.CustomerID, cc1.TotalAmount
FROM #CustomerCalc cc1
INNER JOIN #CustomerCalc cc2 ON cc1.CustomerID = cc2.CustomerID
WHERE cc2.OrderCount > 10;

DROP TABLE #CustomerCalc;

-- Recursive CTE với MAXRECURSION / Recursive CTE with MAXRECURSION
WITH EmployeeHierarchy AS (
    SELECT EmployeeID, ManagerID, FirstName, 0 as Level
    FROM Employee
    WHERE ManagerID IS NULL

    UNION ALL

    SELECT e.EmployeeID, e.ManagerID, e.FirstName, eh.Level + 1
    FROM Employee e
    INNER JOIN EmployeeHierarchy eh ON e.ManagerID = eh.EmployeeID
    WHERE eh.Level < 10 -- Prevent infinite recursion / Ngăn đệ quy vô hạn
)
SELECT * FROM EmployeeHierarchy
OPTION (MAXRECURSION 100); -- Set maximum recursion level / Đặt mức đệ quy tối đa
```

### 7. When to Choose Each Approach / Khi nào chọn phương pháp nào

**Câu hỏi / Question:** Làm thế nào để quyết định sử dụng CTE, Table Variable hay Temp Table? / How to decide between using CTE, Table Variable, or Temp Table?

**Trả lời / Answer:**

| Scenario / Kịch bản                                                  | Data Size / Kích thước        | Recommended / Khuyến nghị    | Reason / Lý do                                              |
| -------------------------------------------------------------------- | ----------------------------- | ---------------------------- | ----------------------------------------------------------- |
| Simple data transformation / Chuyển đổi dữ liệu đơn giản             | Any / Bất kỳ                  | CTE                          | Dễ đọc, không cần storage / Easy to read, no storage needed |
| Recursive operations / Thao tác đệ quy                               | Any / Bất kỳ                  | CTE                          | Only option for recursion / Chỉ có CTE hỗ trợ đệ quy        |
| Small result set / Tập kết quả nhỏ                                   | < 1,000 rows (< 1 MB)         | Table Variable               | Memory-based, fast / Dựa trên bộ nhớ, nhanh                 |
| Medium result set / Tập kết quả trung bình                           | 1,000 - 10,000 rows (1-10 MB) | Table Variable or Temp Table | Depends on complexity / Tùy thuộc độ phức tạp               |
| Large result set / Tập kết quả lớn                                   | > 10,000 rows (> 10 MB)       | Temp Table                   | Statistics and indexes available / Có statistics và indexes |
| Very large result set / Tập kết quả rất lớn                          | > 100,000 rows (> 100 MB)     | Temp Table + Indexes         | Must have indexes for performance / Bắt buộc có indexes     |
| Multiple references to same data / Nhiều tham chiếu đến cùng dữ liệu | > 1,000 rows                  | Temp Table                   | Avoid re-evaluation / Tránh tính toán lại                   |
| Cross-procedure data sharing / Chia sẻ dữ liệu giữa procedures       | Any / Bất kỳ                  | Temp Table                   | Scope persists / Phạm vi tồn tại lâu hơn                    |
| Transaction rollback concerns / Quan tâm về rollback transaction     | < 10,000 rows                 | Table Variable               | Not affected by rollback / Không bị ảnh hưởng bởi rollback  |
| Complex joins and aggregations / JOIN và tổng hợp phức tạp           | > 5,000 rows                  | Temp Table                   | Need statistics for optimization / Cần statistics để tối ưu |

```sql
-- Decision Matrix Example / Ví dụ ma trận quyết định

-- Scenario 1: Simple lookup transformation / Chuyển đổi lookup đơn giản
-- Choose: CTE
WITH StatusLookup AS (
    SELECT StatusID, StatusName FROM Status
)
SELECT o.OrderID, o.CustomerID, sl.StatusName
FROM Orders o
INNER JOIN StatusLookup sl ON o.StatusID = sl.StatusID;

-- Scenario 2: Multiple complex calculations / Nhiều tính toán phức tạp
-- Choose: Temp Table
CREATE TABLE #CustomerCalc AS
SELECT
    CustomerID,
    SUM(Amount) as TotalAmount,
    COUNT(*) as OrderCount,
    (SELECT AVG(Amount) FROM Orders o2 WHERE o2.CustomerID = o1.CustomerID) as AvgAmount
FROM Orders o1
GROUP BY CustomerID;

SELECT cc1.CustomerID, cc1.TotalAmount
FROM #CustomerCalc cc1
INNER JOIN #CustomerCalc cc2 ON cc1.CustomerID = cc2.CustomerID
WHERE cc2.OrderCount > 10;

DROP TABLE #CustomerCalc;
```

## Performance và Optimization / Hiệu suất và tối ưu hóa

### 4. Query Execution Plan / Kế hoạch thực thi truy vấn

**Câu hỏi / Question:** Làm thế nào để phân tích performance của một query? / How to analyze query performance?

**Mục đích / Purpose:** Execution Plan cho phép hiểu cách SQL Server thực thi queries, identify performance bottlenecks, và optimize query performance.
**Purpose:** Execution Plan allows understanding how SQL Server executes queries, identifies performance bottlenecks, and optimizes query performance.

**Tại sao quan trọng / Why important:**

- Identify expensive operations / Xác định operations tốn kém
- Find missing indexes / Tìm indexes thiếu
- Understand data access patterns / Hiểu patterns truy cập dữ liệu
- Optimize JOIN strategies / Tối ưu chiến lược JOIN
- Detect parameter sniffing issues / Phát hiện vấn đề parameter sniffing

#### 4.1. Các loại Execution Plans / Types of Execution Plans

**Estimated vs Actual Plans:**

- **Estimated Plan:** Dự đoán execution mà không chạy query
- **Actual Plan:** Plan thực tế sau khi query đã execute
- **Live Query Statistics:** Real-time execution progress

```sql
-- Enable execution plan analysis / Bật phân tích execution plan
SET STATISTICS IO ON;        -- Hiển thị I/O statistics
SET STATISTICS TIME ON;      -- Hiển thị time statistics
SET STATISTICS PROFILE ON;   -- Hiển thị detailed execution info

-- Example query để analyze / Ví dụ query để phân tích
SELECT e.FirstName, e.LastName, d.DepartmentName, e.Salary
FROM Employee e
INNER JOIN Department d ON e.DepartmentID = d.DepartmentID
WHERE e.Salary > 50000
ORDER BY e.Salary DESC;

SET STATISTICS IO OFF;
SET STATISTICS TIME OFF;
SET STATISTICS PROFILE OFF;
```

#### 4.2. Reading Execution Plans / Đọc Execution Plans

```sql
-- Ví dụ thực tế: E-commerce Order Analysis / Real-world example: E-commerce Order Analysis
CREATE TABLE Customers (
    CustomerID INT IDENTITY(1,1) PRIMARY KEY,
    CustomerName NVARCHAR(100) NOT NULL,
    Email NVARCHAR(100),
    City NVARCHAR(50),
    Country NVARCHAR(50),
    RegistrationDate DATETIME DEFAULT GETDATE()
);

CREATE TABLE Orders (
    OrderID INT IDENTITY(1,1) PRIMARY KEY,
    CustomerID INT NOT NULL,
    OrderDate DATETIME NOT NULL,
    TotalAmount DECIMAL(15,2) NOT NULL,
    Status VARCHAR(20) NOT NULL DEFAULT 'Pending',
    FOREIGN KEY (CustomerID) REFERENCES Customers(CustomerID)
);

CREATE TABLE OrderItems (
    OrderItemID INT IDENTITY(1,1) PRIMARY KEY,
    OrderID INT NOT NULL,
    ProductName NVARCHAR(255),
    Quantity INT NOT NULL,
    UnitPrice DECIMAL(10,2) NOT NULL,
    FOREIGN KEY (OrderID) REFERENCES Orders(OrderID)
);

-- Insert sample data / Chèn dữ liệu mẫu
INSERT INTO Customers (CustomerName, Email, City, Country, RegistrationDate)
SELECT
    'Customer ' + CAST(ROW_NUMBER() OVER (ORDER BY (SELECT NULL)) AS VARCHAR(10)),
    'customer' + CAST(ROW_NUMBER() OVER (ORDER BY (SELECT NULL)) AS VARCHAR(10)) + '@email.com',
    CASE ABS(CHECKSUM(NEWID())) % 5
        WHEN 0 THEN 'New York'
        WHEN 1 THEN 'London'
        WHEN 2 THEN 'Paris'
        WHEN 3 THEN 'Tokyo'
        ELSE 'Sydney'
    END,
    CASE ABS(CHECKSUM(NEWID())) % 3
        WHEN 0 THEN 'USA'
        WHEN 1 THEN 'UK'
        ELSE 'France'
    END,
    DATEADD(day, -ABS(CHECKSUM(NEWID())) % 365, GETDATE())
FROM master.dbo.spt_values
WHERE type = 'P' AND number < 1000;  -- 1000 customers

-- Complex query for analysis / Query phức tạp để phân tích
-- Business scenario: Find high-value customers with recent large orders
-- Kịch bản: Tìm khách hàng có giá trị cao với đơn hàng lớn gần đây
```

#### 4.3. Common Execution Plan Operators / Các toán tử phổ biến trong Execution Plan

```sql
-- Scenario 1: Table Scan vs Index Seek / Quét bảng vs Tìm kiếm index
-- BAD QUERY: Causes table scan / Query tệ: gây ra table scan
SET STATISTICS IO ON;
SELECT CustomerName, Email FROM Customers WHERE UPPER(CustomerName) LIKE '%JOHN%';
-- Result: Table Scan (expensive) / Kết quả: Table Scan (tốn kém)
-- Logical reads: High / Logical reads: Cao

-- GOOD QUERY: Uses index seek / Query tốt: sử dụng index seek
SELECT CustomerName, Email FROM Customers WHERE CustomerName LIKE 'John%';
-- Result: Index Seek (efficient) / Kết quả: Index Seek (hiệu quả)
-- Logical reads: Low / Logical reads: Thấp

-- Create supporting index / Tạo index hỗ trợ
CREATE NONCLUSTERED INDEX IX_Customers_Name_Email
ON Customers (CustomerName) INCLUDE (Email);

-- Retest the query / Test lại query
SELECT CustomerName, Email FROM Customers WHERE CustomerName LIKE 'Customer 1%';
SET STATISTICS IO OFF;
```

#### 4.4. JOIN Execution Strategies / Chiến lược thực thi JOIN

```sql
-- Different JOIN algorithms and when they occur / Các thuật toán JOIN khác nhau và khi nào xảy ra

-- 1. NESTED LOOP JOIN
-- Best for: Small datasets, good indexes on join columns
-- Tốt nhất cho: Datasets nhỏ, có indexes tốt trên join columns
SELECT TOP 10 c.CustomerName, o.OrderDate, o.TotalAmount
FROM Customers c
INNER JOIN Orders o ON c.CustomerID = o.CustomerID
WHERE c.CustomerID BETWEEN 1 AND 10;
-- Expected: Nested Loop (small result set) / Dự kiến: Nested Loop (tập kết quả nhỏ)

-- 2. HASH JOIN
-- Best for: Large datasets, no good indexes
-- Tốt nhất cho: Datasets lớn, không có indexes tốt
SELECT c.Country, COUNT(*) as CustomerCount, SUM(o.TotalAmount) as TotalRevenue
FROM Customers c
INNER JOIN Orders o ON c.CustomerID = o.CustomerID
GROUP BY c.Country;
-- Expected: Hash Join (large datasets) / Dự kiến: Hash Join (datasets lớn)

-- 3. MERGE JOIN
-- Best for: Both tables sorted on join columns
-- Tốt nhất cho: Cả hai bảng đã sort trên join columns
SELECT c.CustomerName, o.OrderDate, o.TotalAmount
FROM Customers c
INNER JOIN Orders o ON c.CustomerID = o.CustomerID
ORDER BY c.CustomerID, o.OrderDate;
-- Expected: Merge Join (if proper indexes exist) / Dự kiến: Merge Join (nếu có indexes phù hợp)

-- 4. DETAILED JOIN STRATEGY ANALYSIS / PHÂN TÍCH CHI TIẾT CHIẾN LƯỢC JOIN

-- 4.1 NESTED LOOP JOIN - Chi tiết hoạt động
/*
Cách hoạt động:
1. Quét từng row của bảng ngoài (outer table)
2. Với mỗi row, tìm kiếm trong bảng trong (inner table)
3. Sử dụng index nếu có để tìm kiếm nhanh

Ưu điểm:
- Hiệu quả khi outer table nhỏ
- Tận dụng index tốt
- Memory usage thấp

Nhược điểm:
- Chậm khi outer table lớn
- Không hiệu quả khi không có index
*/

-- Ví dụ Nested Loop Join hiệu quả:
SELECT c.CustomerName, o.OrderDate
FROM Customers c  -- Outer table (nhỏ)
INNER JOIN Orders o ON c.CustomerID = o.CustomerID  -- Inner table (lớn)
WHERE c.CustomerID IN (1, 2, 3, 4, 5);  -- Chỉ 5 customers

-- 4.2 HASH JOIN - Chi tiết hoạt động
/*
Cách hoạt động:
1. Tạo hash table từ bảng nhỏ hơn
2. Quét bảng lớn và tìm kiếm trong hash table
3. Không cần index, nhưng cần memory để lưu hash table

Ưu điểm:
- Hiệu quả với datasets lớn
- Không cần index trên join columns
- Tốt cho equi-joins

Nhược điểm:
- Cần memory để lưu hash table
- Không hiệu quả cho range queries
- Overhead tạo hash table
*/

-- Ví dụ Hash Join hiệu quả:
SELECT c.Country, COUNT(*) as CustomerCount
FROM Customers c  -- Bảng lớn
INNER JOIN Orders o ON c.CustomerID = o.CustomerID  -- Bảng lớn
WHERE o.OrderDate >= '2024-01-01'
GROUP BY c.Country;

-- 4.3 MERGE JOIN - Chi tiết hoạt động
/*
Cách hoạt động:
1. Cả hai bảng phải được sort theo join columns
2. Merge hai sorted lists
3. Chỉ cần scan một lần qua mỗi bảng

Ưu điểm:
- Rất hiệu quả khi data đã sort
- Memory usage thấp
- Tốt cho large datasets

Nhược điểm:
- Cần sort data trước (tốn kém)
- Chỉ hiệu quả khi có proper indexes
*/

-- Ví dụ Merge Join hiệu quả:
SELECT c.CustomerName, o.OrderDate
FROM Customers c
INNER JOIN Orders o ON c.CustomerID = o.CustomerID
WHERE c.CustomerID BETWEEN 1000 AND 2000
ORDER BY c.CustomerID, o.OrderDate;

-- 4.4 FACTORS AFFECTING JOIN CHOICE / CÁC YẾU TỐ ẢNH HƯỞNG ĐẾN LỰA CHỌN JOIN

-- A. Data Size / Kích thước dữ liệu
-- Small datasets (< 1000 rows): Nested Loop
-- Medium datasets (1000-10000 rows): Hash Join
-- Large datasets (> 10000 rows): Hash Join hoặc Merge Join

-- B. Index Availability / Sự có sẵn của Index
-- Có index trên join columns: Nested Loop hoặc Merge Join
-- Không có index: Hash Join

-- C. Memory Availability / Sự có sẵn của Memory
-- Memory cao: Hash Join
-- Memory thấp: Nested Loop

-- D. Query Type / Loại Query
-- Equi-joins (=): Tất cả 3 loại
-- Range joins (>, <, BETWEEN): Nested Loop hoặc Merge Join

-- 4.5 FORCING JOIN STRATEGIES / ÉP BUỘC CHIẾN LƯỢC JOIN

-- Force Nested Loop Join:
SELECT c.CustomerName, o.OrderDate
FROM Customers c
INNER LOOP JOIN Orders o ON c.CustomerID = o.CustomerID;

-- Force Hash Join:
SELECT c.CustomerName, o.OrderDate
FROM Customers c
INNER HASH JOIN Orders o ON c.CustomerID = o.CustomerID;

-- Force Merge Join:
SELECT c.CustomerName, o.OrderDate
FROM Customers c
INNER MERGE JOIN Orders o ON c.CustomerID = o.CustomerID;

-- 4.6 MONITORING JOIN PERFORMANCE / THEO DÕI HIỆU SUẤT JOIN

-- Kiểm tra execution plan:
SET STATISTICS IO ON;
SET STATISTICS TIME ON;

-- Query để test:
SELECT c.Country, COUNT(*) as CustomerCount
FROM Customers c
INNER JOIN Orders o ON c.CustomerID = o.CustomerID
GROUP BY c.Country;

-- Các metrics quan trọng:
-- Logical reads: Số lần đọc từ buffer cache
-- Physical reads: Số lần đọc từ disk
-- CPU time: Thời gian xử lý CPU
-- Elapsed time: Tổng thời gian thực thi

-- 4.7 COMMON JOIN OPTIMIZATION TECHNIQUES / KỸ THUẬT TỐI ƯU JOIN PHỔ BIẾN

-- A. Index Optimization / Tối ưu Index
CREATE NONCLUSTERED INDEX IX_Customers_CustomerID
ON Customers (CustomerID) INCLUDE (CustomerName, Country);

CREATE NONCLUSTERED INDEX IX_Orders_CustomerID_Date
ON Orders (CustomerID, OrderDate) INCLUDE (TotalAmount);

-- B. Query Rewriting / Viết lại Query
-- Thay vì:
SELECT c.CustomerName,
       (SELECT COUNT(*) FROM Orders o WHERE o.CustomerID = c.CustomerID) as OrderCount
FROM Customers c;

-- Sử dụng:
SELECT c.CustomerName, COUNT(o.OrderID) as OrderCount
FROM Customers c
LEFT JOIN Orders o ON c.CustomerID = o.CustomerID
GROUP BY c.CustomerID, c.CustomerName;

-- C. Partitioning / Phân vùng
-- Chia nhỏ bảng lớn thành các partition
-- Giúp Nested Loop Join hiệu quả hơn

-- D. Statistics Maintenance / Bảo trì Statistics
-- Cập nhật statistics để optimizer có thông tin chính xác
UPDATE STATISTICS Customers;
UPDATE STATISTICS Orders;
```

#### 4.5. Performance Bottleneck Identification / Xác định nút thắt hiệu suất

```sql
-- Complex query with multiple potential issues / Query phức tạp với nhiều vấn đề tiềm ẩn
-- Business scenario: Monthly sales report with customer details and ranking
-- Kịch bản: Báo cáo doanh số tháng với chi tiết khách hàng và xếp hạng

-- PROBLEMATIC QUERY / QUERY CÓ VẤN ĐỀ:
SELECT
    c.CustomerName,
    c.Email,
    c.City,
    c.Country,
    COUNT(o.OrderID) as OrderCount,
    SUM(o.TotalAmount) as TotalSpent,
    AVG(o.TotalAmount) as AvgOrderValue,
    RANK() OVER (PARTITION BY c.Country ORDER BY SUM(o.TotalAmount) DESC) as CountryRank,
    -- Expensive subquery / Subquery tốn kém
    (SELECT COUNT(*) FROM Orders o2 WHERE o2.CustomerID = c.CustomerID AND o2.Status = 'Completed') as CompletedOrders,
    -- Another expensive subquery / Subquery tốn kém khác
    (SELECT MAX(TotalAmount) FROM Orders o3 WHERE o3.CustomerID = c.CustomerID) as LargestOrder
FROM Customers c
LEFT JOIN Orders o ON c.CustomerID = o.CustomerID
WHERE o.OrderDate >= '2024-01-01'
  AND c.Country IN ('USA', 'UK', 'France')
  AND EXISTS (SELECT 1 FROM Orders o4 WHERE o4.CustomerID = c.CustomerID AND o4.TotalAmount > 1000)
GROUP BY c.CustomerID, c.CustomerName, c.Email, c.City, c.Country
HAVING SUM(o.TotalAmount) > 5000
ORDER BY TotalSpent DESC;

-- Issues trong execution plan sẽ shows:
-- 1. Table Scans due to missing indexes
-- 2. Multiple executions of correlated subqueries
-- 3. Expensive sorts due to lack of proper indexes
-- 4. High logical reads và CPU time

-- OPTIMIZED VERSION / PHIÊN BẢN TỐI ƯU:
WITH CustomerOrderStats AS (
    SELECT
        o.CustomerID,
        COUNT(*) as OrderCount,
        SUM(o.TotalAmount) as TotalSpent,
        AVG(o.TotalAmount) as AvgOrderValue,
        COUNT(CASE WHEN o.Status = 'Completed' THEN 1 END) as CompletedOrders,
        MAX(o.TotalAmount) as LargestOrder
    FROM Orders o
    WHERE o.OrderDate >= '2024-01-01'
    GROUP BY o.CustomerID
    HAVING SUM(o.TotalAmount) > 5000
),
CustomerRanking AS (
    SELECT
        c.CustomerID,
        c.CustomerName,
        c.Email,
        c.City,
        c.Country,
        cos.OrderCount,
        cos.TotalSpent,
        cos.AvgOrderValue,
        cos.CompletedOrders,
        cos.LargestOrder,
        RANK() OVER (PARTITION BY c.Country ORDER BY cos.TotalSpent DESC) as CountryRank
    FROM Customers c
    INNER JOIN CustomerOrderStats cos ON c.CustomerID = cos.CustomerID
    WHERE c.Country IN ('USA', 'UK', 'France')
      AND cos.TotalSpent > 1000  -- Replaced EXISTS with simple filter
)
SELECT * FROM CustomerRanking
ORDER BY TotalSpent DESC;

-- Supporting indexes for optimization / Indexes hỗ trợ để tối ưu:
CREATE NONCLUSTERED INDEX IX_Orders_Customer_Date_Amount
ON Orders (CustomerID, OrderDate, Status) INCLUDE (TotalAmount);

CREATE NONCLUSTERED INDEX IX_Customers_Country
ON Customers (Country) INCLUDE (CustomerName, Email, City);
```

#### 4.6. Execution Plan Analysis Tools / Công cụ phân tích Execution Plan

```sql
-- 1. Using SHOWPLAN options / Sử dụng tùy chọn SHOWPLAN
SET SHOWPLAN_ALL ON;
SELECT c.CustomerName, COUNT(o.OrderID)
FROM Customers c
LEFT JOIN Orders o ON c.CustomerID = o.CustomerID
GROUP BY c.CustomerName;
SET SHOWPLAN_ALL OFF;

-- 2. XML execution plans / Execution plans dạng XML
SET SHOWPLAN_XML ON;
SELECT c.CustomerName, SUM(o.TotalAmount)
FROM Customers c
INNER JOIN Orders o ON c.CustomerID = o.CustomerID
WHERE o.OrderDate >= '2024-01-01'
GROUP BY c.CustomerName;
SET SHOWPLAN_XML OFF;

-- 3. Live Query Statistics / Thống kê query trực tiếp
-- Enable in SSMS: Query menu > Include Live Query Statistics
-- Shows real-time progress of query execution
-- Hiển thị tiến trình thực thi query theo thời gian thực

-- 4. Query Store execution plans / Execution plans từ Query Store
SELECT
    qp.plan_id,
    qp.query_id,
    qp.avg_duration,
    qp.last_execution_time,
    qp.query_plan  -- XML execution plan
FROM sys.query_store_plan qp
INNER JOIN sys.query_store_query qq ON qp.query_id = qq.query_id
WHERE qq.object_id = OBJECT_ID('Customers')
ORDER BY qp.avg_duration DESC;
```

#### 4.7. Common Performance Issues và Solutions / Vấn đề hiệu suất phổ biến và giải pháp

```sql
-- Issue 1: Parameter Sniffing / Vấn đề Parameter Sniffing
-- Problem: Cached plan optimized for specific parameter value
-- Vấn đề: Cached plan được tối ưu cho giá trị parameter cụ thể

CREATE PROCEDURE GetCustomerOrders
    @Country NVARCHAR(50)
AS
BEGIN
    -- BAD: Subject to parameter sniffing / Có vấn đề parameter sniffing
    SELECT c.CustomerName, COUNT(o.OrderID) as OrderCount
    FROM Customers c
    LEFT JOIN Orders o ON c.CustomerID = o.CustomerID
    WHERE c.Country = @Country
    GROUP BY c.CustomerName;
END;

-- SOLUTION 1: Use OPTION (RECOMPILE) / Giải pháp 1: Sử dụng OPTION (RECOMPILE)
CREATE PROCEDURE GetCustomerOrders_Fixed1
    @Country NVARCHAR(50)
AS
BEGIN
    SELECT c.CustomerName, COUNT(o.OrderID) as OrderCount
    FROM Customers c
    LEFT JOIN Orders o ON c.CustomerID = o.CustomerID
    WHERE c.Country = @Country
    GROUP BY c.CustomerName
    OPTION (RECOMPILE);  -- Forces recompilation each time / Buộc recompile mỗi lần
END;

-- SOLUTION 2: Use local variables / Giải pháp 2: Sử dụng local variables
CREATE PROCEDURE GetCustomerOrders_Fixed2
    @Country NVARCHAR(50)
AS
BEGIN
    DECLARE @LocalCountry NVARCHAR(50) = @Country;

    SELECT c.CustomerName, COUNT(o.OrderID) as OrderCount
    FROM Customers c
    LEFT JOIN Orders o ON c.CustomerID = o.CustomerID
    WHERE c.Country = @LocalCountry  -- Uses average statistics / Sử dụng statistics trung bình
    GROUP BY c.CustomerName;
END;

-- SOLUTION 3: Use OPTIMIZE FOR hint / Giải pháp 3: Sử dụng OPTIMIZE FOR hint
CREATE PROCEDURE GetCustomerOrders_Fixed3
    @Country NVARCHAR(50)
AS
BEGIN
    SELECT c.CustomerName, COUNT(o.OrderID) as OrderCount
    FROM Customers c
    LEFT JOIN Orders o ON c.CustomerID = o.CustomerID
    WHERE c.Country = @Country
    GROUP BY c.CustomerName
    OPTION (OPTIMIZE FOR (@Country = 'USA'));  -- Optimize for specific value
END;
```

#### 4.8. Advanced Execution Plan Analysis / Phân tích Execution Plan nâng cao

```sql
-- Issue 2: Implicit Conversions / Chuyển đổi ngầm
-- Problem: Data type mismatches cause expensive conversions
-- Vấn đề: Không khớp kiểu dữ liệu gây chuyển đổi tốn kém

-- BAD: Implicit conversion from INT to VARCHAR / Chuyển đổi ngầm từ INT sang VARCHAR
SELECT CustomerName FROM Customers WHERE CustomerID = '123';  -- '123' is VARCHAR
-- Plan shows: CONVERT_IMPLICIT(int,[Customers].[CustomerID],0)='123'

-- GOOD: Proper data type / Kiểu dữ liệu đúng
SELECT CustomerName FROM Customers WHERE CustomerID = 123;  -- 123 is INT

-- Issue 3: Missing Statistics / Thiếu thống kê
-- Check statistics freshness / Kiểm tra độ tươi của statistics
SELECT
    s.name AS StatisticsName,
    sp.last_updated,
    sp.rows,
    sp.rows_sampled,
    sp.modification_counter,
    CASE
        WHEN sp.modification_counter > sp.rows * 0.2 THEN 'STALE - UPDATE NEEDED'
        WHEN sp.modification_counter > sp.rows * 0.1 THEN 'AGING - MONITOR'
        ELSE 'FRESH'
    END AS StatisticsStatus
FROM sys.stats s
CROSS APPLY sys.dm_db_stats_properties(s.object_id, s.stats_id) sp
WHERE s.object_id = OBJECT_ID('Orders')
ORDER BY sp.modification_counter DESC;

-- Update stale statistics / Cập nhật statistics cũ
UPDATE STATISTICS Orders WITH FULLSCAN;

-- Issue 4: Cardinality Estimation Problems / Vấn đề ước tính cardinality
-- Force legacy cardinality estimator if needed / Buộc sử dụng cardinality estimator cũ nếu cần
SELECT c.CustomerName, COUNT(o.OrderID)
FROM Customers c
LEFT JOIN Orders o ON c.CustomerID = o.CustomerID
WHERE c.RegistrationDate >= '2024-01-01'
GROUP BY c.CustomerName
OPTION (USE HINT('FORCE_LEGACY_CARDINALITY_ESTIMATION'));
```

#### 4.9. Execution Plan Best Practices / Thực hành tốt nhất cho Execution Plan

```sql
-- 1. Always analyze actual vs estimated plans / Luôn phân tích actual vs estimated plans
-- 2. Look for warnings in execution plans / Tìm warnings trong execution plans
-- 3. Focus on most expensive operations first / Tập trung vào operations tốn kém nhất trước
-- 4. Check for missing index recommendations / Kiểm tra khuyến nghị missing indexes

-- Example: Comprehensive performance analysis / Ví dụ: Phân tích performance toàn diện
-- Business scenario: Customer lifetime value analysis
-- Kịch bản: Phân tích giá trị khách hàng suốt đời

-- Enable detailed analysis / Bật phân tích chi tiết
SET STATISTICS IO ON;
SET STATISTICS TIME ON;

-- Complex analytical query / Query phân tích phức tạp
WITH CustomerLifetimeValue AS (
    SELECT
        c.CustomerID,
        c.CustomerName,
        c.RegistrationDate,
        COUNT(o.OrderID) as TotalOrders,
        SUM(o.TotalAmount) as LifetimeValue,
        AVG(o.TotalAmount) as AvgOrderValue,
        DATEDIFF(day, MIN(o.OrderDate), MAX(o.OrderDate)) as CustomerLifespanDays,
        FIRST_VALUE(o.OrderDate) OVER (PARTITION BY c.CustomerID ORDER BY o.OrderDate) as FirstOrderDate,
        LAST_VALUE(o.OrderDate) OVER (PARTITION BY c.CustomerID ORDER BY o.OrderDate
            ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING) as LastOrderDate
    FROM Customers c
    INNER JOIN Orders o ON c.CustomerID = o.CustomerID
    WHERE o.Status = 'Completed'
    GROUP BY c.CustomerID, c.CustomerName, c.RegistrationDate
),
CustomerSegmentation AS (
    SELECT *,
        CASE
            WHEN LifetimeValue >= 10000 AND TotalOrders >= 20 THEN 'VIP'
            WHEN LifetimeValue >= 5000 AND TotalOrders >= 10 THEN 'Premium'
            WHEN LifetimeValue >= 1000 AND TotalOrders >= 5 THEN 'Regular'
            ELSE 'Basic'
        END as CustomerSegment,
        NTILE(10) OVER (ORDER BY LifetimeValue DESC) as ValueDecile
    FROM CustomerLifetimeValue
)
SELECT
    CustomerSegment,
    COUNT(*) as CustomerCount,
    AVG(LifetimeValue) as AvgLifetimeValue,
    AVG(TotalOrders) as AvgTotalOrders,
    AVG(AvgOrderValue) as AvgOrderValue,
    AVG(CustomerLifespanDays) as AvgLifespanDays
FROM CustomerSegmentation
GROUP BY CustomerSegment
ORDER BY AvgLifetimeValue DESC;

-- Analyze the results:
-- - Check logical reads for each table
-- - Look for table scans vs index seeks
-- - Identify expensive operations (sorts, hash joins)
-- - Review warnings and missing index suggestions
-- - Compare estimated vs actual row counts

SET STATISTICS IO OFF;
SET STATISTICS TIME OFF;

-- Key metrics to monitor in execution plan / Metrics chính cần giám sát:
-- 1. Logical Reads: Lower is better / Thấp hơn thì tốt hơn
-- 2. CPU Time: Should be reasonable for query complexity / Nên hợp lý với độ phức tạp query
-- 3. Elapsed Time: Total time including waits / Tổng thời gian bao gồm waits
-- 4. Estimated vs Actual Rows: Should be close / Nên gần nhau
-- 5. Warnings: Address missing indexes, data type issues / Giải quyết missing indexes, vấn đề kiểu dữ liệu
```

#### 4.10. Execution Plan Troubleshooting Checklist / Checklist xử lý sự cố Execution Plan

**Performance Issues Checklist / Checklist vấn đề hiệu suất:**

1. ✅ **High Logical Reads:**

   - Check for table scans → Add indexes
   - Look for key lookups → Create covering indexes
   - Review JOIN strategies → Ensure proper indexes on JOIN columns

2. ✅ **Long Execution Time:**

   - Identify expensive operators (>50% cost)
   - Check for blocking/deadlocks
   - Review SORT operations → Add ORDER BY indexes

3. ✅ **Inaccurate Estimates:**

   - Update statistics: `UPDATE STATISTICS table_name WITH FULLSCAN`
   - Check for parameter sniffing → Use RECOMPILE or local variables
   - Review multi-column statistics → Create filtered statistics

4. ✅ **Warnings in Plan:**

   - Missing indexes → Implement suggested indexes
   - Implicit conversions → Fix data type mismatches
   - Unmatched indexes → Review WHERE clause conditions

5. ✅ **Memory Issues:**
   - Hash spills to tempdb → Increase memory or rewrite query
   - Sort spills → Add supporting indexes for ORDER BY
   - Large memory grants → Optimize query or add indexes

**Summary / Tóm tắt:**
Execution Plan analysis là essential skill để optimize SQL Server performance. Bằng cách hiểu plans, developers có thể identify bottlenecks, create appropriate indexes, và write efficient queries cho production systems.

### 5. Index Optimization / Tối ưu hóa chỉ mục

**Câu hỏi / Question:** Làm thế nào để tối ưu index cho query? / How to optimize indexes for queries?

**Mục đích / Purpose:** Index Optimization giúp cải thiện performance của queries bằng cách tạo ra các cấu trúc dữ liệu hiệu quả để SQL Server có thể truy cập data nhanh hơn.
**Purpose:** Index Optimization helps improve query performance by creating efficient data structures for SQL Server to access data faster.

**Tại sao quan trọng / Why important:**

- Giảm I/O operations / Reduce I/O operations
- Tăng tốc độ SELECT, WHERE, JOIN, ORDER BY / Speed up SELECT, WHERE, JOIN, ORDER BY
- Cải thiện concurrency / Improve concurrency
- Giảm CPU usage / Reduce CPU usage

#### 5.1. Các loại Index và khi nào sử dụng / Index types and when to use

**Clustered Index / Chỉ mục cụm:**

- **Mục đích:** Sắp xếp physical data pages theo thứ tự của index key
- **Khi dùng:** Primary key, frequently searched ranges
- **Giới hạn:** Chỉ 1 clustered index per table

**Nonclustered Index / Chỉ mục không cụm:**

- **Mục đích:** Tạo separate structure pointing to data rows
- **Khi dùng:** Foreign keys, frequently searched columns, WHERE clauses
- **Giới hạn:** Up to 999 nonclustered indexes per table

```sql
-- Ví dụ thực tế: E-commerce Database / Real-world example: E-commerce Database

-- 1. Bảng Products với different index strategies
CREATE TABLE Products (
    ProductID INT IDENTITY(1,1) PRIMARY KEY, -- Automatically creates clustered index
    ProductName NVARCHAR(255) NOT NULL,
    CategoryID INT NOT NULL,
    Price DECIMAL(10,2) NOT NULL,
    StockQuantity INT NOT NULL,
    CreatedDate DATETIME DEFAULT GETDATE(),
    LastModified DATETIME DEFAULT GETDATE(),
    IsActive BIT DEFAULT 1
);

-- 2. Nonclustered index cho category searches (common in e-commerce)
CREATE NONCLUSTERED INDEX IX_Products_CategoryID
ON Products (CategoryID)
INCLUDE (ProductName, Price, StockQuantity);
-- Lý do: Customers often browse by category / Customers often browse by category

-- 3. Covering index cho price range searches
CREATE NONCLUSTERED INDEX IX_Products_Price_Active
ON Products (IsActive, Price)
INCLUDE (ProductID, ProductName, CategoryID, StockQuantity);
-- Lý do: Filter by price ranges and active products / Filter by price ranges and active products

-- 4. Composite index cho multi-column searches
CREATE NONCLUSTERED INDEX IX_Products_Category_Price
ON Products (CategoryID, Price DESC, IsActive);
-- Lý do: Category + price sorting (expensive to cheap) / Category + price sorting (expensive to cheap)
```

#### 5.2. Index Analysis và Missing Index Detection / Index Analysis and Missing Index Detection

```sql
-- Phân tích missing indexes / Analyze missing indexes
SELECT
    dm_migs.avg_user_impact,
    dm_migs.last_user_seek,
    dm_migs.last_user_scan,
    dm_mid.statement AS TableName,
    dm_migs.unique_compiles,
    dm_migs.user_seeks,
    dm_migs.user_scans,
    dm_migs.avg_total_user_cost,
    dm_mid.equality_columns,
    dm_mid.inequality_columns,
    dm_mid.included_columns,
    'CREATE NONCLUSTERED INDEX IX_' +
    REPLACE(REPLACE(REPLACE(dm_mid.statement, '[', ''), ']', ''), '.', '_') + '_' +
    ISNULL(REPLACE(dm_mid.equality_columns, ', ', '_'), '') +
    CASE WHEN dm_mid.inequality_columns IS NOT NULL
         THEN '_' + REPLACE(dm_mid.inequality_columns, ', ', '_')
         ELSE '' END +
    ' ON ' + dm_mid.statement +
    ' (' + ISNULL(dm_mid.equality_columns, '') +
    CASE WHEN dm_mid.equality_columns IS NOT NULL AND dm_mid.inequality_columns IS NOT NULL
         THEN ', ' ELSE '' END +
    ISNULL(dm_mid.inequality_columns, '') + ')' +
    CASE WHEN dm_mid.included_columns IS NOT NULL
         THEN ' INCLUDE (' + dm_mid.included_columns + ')'
         ELSE '' END AS CreateIndexStatement
FROM sys.dm_db_missing_index_group_stats dm_migs
INNER JOIN sys.dm_db_missing_index_groups dm_mig
    ON dm_migs.group_handle = dm_mig.index_group_handle
INNER JOIN sys.dm_db_missing_index_details dm_mid
    ON dm_mig.index_handle = dm_mid.index_handle
WHERE dm_migs.avg_user_impact > 50  -- High impact indexes only
  AND dm_migs.user_seeks + dm_migs.user_scans > 1000  -- Frequently accessed
ORDER BY dm_migs.avg_user_impact DESC, dm_migs.avg_total_user_cost DESC;
```

#### 5.3. Index Fragmentation và Maintenance / Index Fragmentation and Maintenance

```sql
-- Kiểm tra fragmentation / Check fragmentation
SELECT
    OBJECT_NAME(ips.object_id) AS TableName,
    i.name AS IndexName,
    ips.index_type_desc,
    ips.avg_fragmentation_in_percent,
    ips.page_count,
    ips.avg_page_space_used_in_percent,
    ips.record_count,
    CASE
        WHEN ips.avg_fragmentation_in_percent >= 30 THEN 'REBUILD'
        WHEN ips.avg_fragmentation_in_percent >= 10 THEN 'REORGANIZE'
        ELSE 'NO ACTION NEEDED'
    END AS RecommendedAction
FROM sys.dm_db_index_physical_stats(DB_ID(), NULL, NULL, NULL, 'DETAILED') ips
INNER JOIN sys.indexes i ON ips.object_id = i.object_id
    AND ips.index_id = i.index_id
WHERE ips.avg_fragmentation_in_percent > 5
  AND ips.page_count > 100  -- Skip small indexes
ORDER BY ips.avg_fragmentation_in_percent DESC;

-- Automated index maintenance script / Script bảo trì index tự động
DECLARE @TableName NVARCHAR(255);
DECLARE @IndexName NVARCHAR(255);
DECLARE @Fragmentation FLOAT;
DECLARE @SQL NVARCHAR(MAX);

DECLARE index_cursor CURSOR FOR
SELECT
    OBJECT_NAME(ips.object_id),
    i.name,
    ips.avg_fragmentation_in_percent
FROM sys.dm_db_index_physical_stats(DB_ID(), NULL, NULL, NULL, 'LIMITED') ips
INNER JOIN sys.indexes i ON ips.object_id = i.object_id AND ips.index_id = i.index_id
WHERE ips.avg_fragmentation_in_percent > 10
  AND ips.page_count > 100
  AND i.name IS NOT NULL;

OPEN index_cursor;
FETCH NEXT FROM index_cursor INTO @TableName, @IndexName, @Fragmentation;

WHILE @@FETCH_STATUS = 0
BEGIN
    IF @Fragmentation >= 30
    BEGIN
        SET @SQL = 'ALTER INDEX [' + @IndexName + '] ON [' + @TableName + '] REBUILD WITH (ONLINE = ON, SORT_IN_TEMPDB = ON)';
        PRINT 'Rebuilding: ' + @SQL;
        -- EXEC sp_executesql @SQL;  -- Uncomment to actually execute
    END
    ELSE IF @Fragmentation >= 10
    BEGIN
        SET @SQL = 'ALTER INDEX [' + @IndexName + '] ON [' + @TableName + '] REORGANIZE';
        PRINT 'Reorganizing: ' + @SQL;
        -- EXEC sp_executesql @SQL;  -- Uncomment to actually execute
    END

    FETCH NEXT FROM index_cursor INTO @TableName, @IndexName, @Fragmentation;
END

CLOSE index_cursor;
DEALLOCATE index_cursor;
```

#### 5.4. Covering Indexes và Include Columns / Covering Indexes and Include Columns

```sql
-- Ví dụ thực tế: Order Management System / Real-world example: Order Management System
CREATE TABLE Orders (
    OrderID INT IDENTITY(1,1) PRIMARY KEY,
    CustomerID INT NOT NULL,
    OrderDate DATETIME NOT NULL,
    TotalAmount DECIMAL(15,2) NOT NULL,
    Status VARCHAR(20) NOT NULL,
    ShippingAddress NVARCHAR(500),
    CreatedBy INT,
    CreatedDate DATETIME DEFAULT GETDATE()
);

-- Common query: Get recent orders for a customer with details
-- Query thường gặp: Lấy đơn hàng gần đây của khách hàng với chi tiết
/*
SELECT OrderID, OrderDate, TotalAmount, Status, ShippingAddress
FROM Orders
WHERE CustomerID = @CustomerID
  AND OrderDate >= '2024-01-01'
ORDER BY OrderDate DESC;
*/

-- BAD Index: Only covers WHERE clause / Chỉ cover WHERE clause
CREATE NONCLUSTERED INDEX IX_Orders_CustomerID_Bad
ON Orders (CustomerID, OrderDate);
-- Problem: Still needs key lookups for TotalAmount, Status, ShippingAddress
-- Vấn đề: Vẫn cần key lookups cho TotalAmount, Status, ShippingAddress

-- GOOD Index: Covering index với INCLUDE / Covering index with INCLUDE
CREATE NONCLUSTERED INDEX IX_Orders_CustomerID_Date_Covering
ON Orders (CustomerID, OrderDate DESC)
INCLUDE (OrderID, TotalAmount, Status, ShippingAddress);
-- Benefit: No key lookups needed, faster query execution
-- Lợi ích: Không cần key lookups, thực thi query nhanh hơn

-- Example query sử dụng covering index / Example query using covering index
SELECT OrderID, OrderDate, TotalAmount, Status, ShippingAddress
FROM Orders WITH (INDEX(IX_Orders_CustomerID_Date_Covering))
WHERE CustomerID = 12345
  AND OrderDate >= '2024-01-01'
ORDER BY OrderDate DESC;
```

#### 5.5. Index Usage Analysis / Phân tích sử dụng Index

```sql
-- Monitor index usage / Giám sát sử dụng index
SELECT
    OBJECT_NAME(i.object_id) AS TableName,
    i.name AS IndexName,
    i.type_desc AS IndexType,
    ius.user_seeks,
    ius.user_scans,
    ius.user_lookups,
    ius.user_updates,
    ius.last_user_seek,
    ius.last_user_scan,
    ius.last_user_lookup,
    ius.last_user_update,
    CASE
        WHEN ius.user_seeks + ius.user_scans + ius.user_lookups = 0 THEN 'UNUSED'
        WHEN ius.user_updates > (ius.user_seeks + ius.user_scans + ius.user_lookups) * 10 THEN 'HIGH_MAINTENANCE'
        WHEN ius.user_seeks + ius.user_scans + ius.user_lookups > 1000 THEN 'FREQUENTLY_USED'
        ELSE 'MODERATE_USAGE'
    END AS UsageCategory,
    (ius.user_seeks + ius.user_scans + ius.user_lookups) AS TotalReads,
    ius.user_updates AS TotalWrites,
    CASE
        WHEN ius.user_updates > 0
        THEN (ius.user_seeks + ius.user_scans + ius.user_lookups) / CAST(ius.user_updates AS FLOAT)
        ELSE NULL
    END AS ReadWriteRatio
FROM sys.indexes i
LEFT JOIN sys.dm_db_index_usage_stats ius
    ON i.object_id = ius.object_id AND i.index_id = ius.index_id
WHERE i.object_id = OBJECT_ID('Orders')  -- Specify table name
  AND i.index_id > 0  -- Exclude heap
ORDER BY TotalReads DESC;

-- Find unused indexes (candidates for removal) / Tìm indexes không sử dụng (có thể xóa)
SELECT
    OBJECT_NAME(i.object_id) AS TableName,
    i.name AS IndexName,
    i.type_desc,
    'DROP INDEX [' + i.name + '] ON [' + OBJECT_NAME(i.object_id) + ']' AS DropStatement
FROM sys.indexes i
LEFT JOIN sys.dm_db_index_usage_stats ius
    ON i.object_id = ius.object_id AND i.index_id = ius.index_id
WHERE i.index_id > 1  -- Exclude clustered index
  AND i.is_primary_key = 0  -- Exclude primary key
  AND i.is_unique_constraint = 0  -- Exclude unique constraints
  AND (ius.user_seeks + ius.user_scans + ius.user_lookups = 0 OR ius.user_seeks IS NULL)
  AND DATEDIFF(day, ISNULL(ius.last_user_seek, '1900-01-01'), GETDATE()) > 90  -- Not used in 90 days
ORDER BY OBJECT_NAME(i.object_id), i.name;
```

#### 5.6. Index Design Best Practices / Thực hành tốt nhất thiết kế Index

```sql
-- Ví dụ thực tế: Social Media Platform / Real-world example: Social Media Platform
CREATE TABLE Posts (
    PostID BIGINT IDENTITY(1,1) PRIMARY KEY,  -- Clustered index
    UserID INT NOT NULL,
    Content NVARCHAR(MAX),
    PostDate DATETIME NOT NULL,
    LikesCount INT DEFAULT 0,
    CommentsCount INT DEFAULT 0,
    IsPublic BIT DEFAULT 1,
    Tags NVARCHAR(500),
    LocationID INT NULL
);

-- Index Strategy Design / Thiết kế chiến lược Index:

-- 1. User's posts timeline (most common query) / Timeline bài viết của user (query phổ biến nhất)
CREATE NONCLUSTERED INDEX IX_Posts_User_Date_Timeline
ON Posts (UserID, PostDate DESC, IsPublic)
INCLUDE (PostID, LikesCount, CommentsCount, LocationID);
-- Query: SELECT posts for user timeline

-- 2. Public posts feed (discover page) / Feed bài viết công khai (trang khám phá)
CREATE NONCLUSTERED INDEX IX_Posts_Public_Date_Feed
ON Posts (IsPublic, PostDate DESC)
INCLUDE (PostID, UserID, LikesCount, CommentsCount)
WHERE IsPublic = 1;  -- Filtered index for public posts only
-- Query: SELECT recent public posts

-- 3. Popular posts (trending) / Bài viết phổ biến (xu hướng)
CREATE NONCLUSTERED INDEX IX_Posts_Popular_Trending
ON Posts (LikesCount DESC, PostDate DESC, IsPublic)
INCLUDE (PostID, UserID, CommentsCount)
WHERE IsPublic = 1 AND PostDate >= DATEADD(day, -7, GETDATE());  -- Recent popular posts
-- Query: SELECT trending posts

-- 4. Location-based posts / Bài viết theo vị trí
CREATE NONCLUSTERED INDEX IX_Posts_Location_Date
ON Posts (LocationID, PostDate DESC, IsPublic)
INCLUDE (PostID, UserID, LikesCount, CommentsCount)
WHERE LocationID IS NOT NULL AND IsPublic = 1;
-- Query: SELECT posts by location

-- Example queries optimized by above indexes / Ví dụ queries được tối ưu bởi các indexes trên:

-- Query 1: User timeline / Timeline người dùng
SELECT PostID, Content, PostDate, LikesCount, CommentsCount
FROM Posts
WHERE UserID = 12345 AND IsPublic = 1
ORDER BY PostDate DESC;
-- Uses: IX_Posts_User_Date_Timeline (covering index)

-- Query 2: Trending posts / Bài viết xu hướng
SELECT TOP 50 PostID, UserID, LikesCount, CommentsCount, PostDate
FROM Posts
WHERE IsPublic = 1 AND PostDate >= DATEADD(day, -7, GETDATE())
ORDER BY LikesCount DESC, PostDate DESC;
-- Uses: IX_Posts_Popular_Trending (filtered covering index)

-- Query 3: Location feed / Feed theo vị trí
SELECT PostID, UserID, PostDate, LikesCount
FROM Posts
WHERE LocationID = 100 AND IsPublic = 1
ORDER BY PostDate DESC;
-- Uses: IX_Posts_Location_Date (filtered covering index)
```

#### 5.7. Index Performance Testing / Kiểm tra hiệu suất Index

```sql
-- Performance comparison script / Script so sánh hiệu suất
-- Test query performance before và after index creation

-- Setup test data / Thiết lập dữ liệu test
INSERT INTO Posts (UserID, Content, PostDate, LikesCount, IsPublic, LocationID)
SELECT
    ABS(CHECKSUM(NEWID())) % 10000 + 1,  -- Random UserID 1-10000
    'Sample post content ' + CAST(ROW_NUMBER() OVER (ORDER BY (SELECT NULL)) AS VARCHAR(10)),
    DATEADD(day, -ABS(CHECKSUM(NEWID())) % 365, GETDATE()),  -- Random date within last year
    ABS(CHECKSUM(NEWID())) % 1000,  -- Random likes 0-999
    CASE WHEN ABS(CHECKSUM(NEWID())) % 10 < 8 THEN 1 ELSE 0 END,  -- 80% public
    CASE WHEN ABS(CHECKSUM(NEWID())) % 5 = 0 THEN ABS(CHECKSUM(NEWID())) % 500 + 1 ELSE NULL END  -- 20% have location
FROM master.dbo.spt_values v1
CROSS JOIN master.dbo.spt_values v2
WHERE v1.type = 'P' AND v2.type = 'P'
  AND v1.number < 100 AND v2.number < 100;  -- 10,000 rows

-- Test query performance / Kiểm tra hiệu suất query
SET STATISTICS IO ON;
SET STATISTICS TIME ON;

-- Before index: Table scan expected / Trước khi có index: Dự kiến table scan
SELECT PostID, PostDate, LikesCount
FROM Posts
WHERE UserID = 5000 AND IsPublic = 1
ORDER BY PostDate DESC;

-- Create the optimized index / Tạo index tối ưu
CREATE NONCLUSTERED INDEX IX_Posts_User_Date_Test
ON Posts (UserID, PostDate DESC, IsPublic)
INCLUDE (PostID, LikesCount);

-- After index: Index seek expected / Sau khi có index: Dự kiến index seek
SELECT PostID, PostDate, LikesCount
FROM Posts
WHERE UserID = 5000 AND IsPublic = 1
ORDER BY PostDate DESC;

SET STATISTICS IO OFF;
SET STATISTICS TIME OFF;

-- Analyze execution plans để compare performance
-- Compare logical reads, CPU time, elapsed time
```

#### 5.8. Common Index Anti-patterns / Các anti-pattern phổ biến của Index

```sql
-- ❌ BAD PRACTICES / THỰC HÀNH SAI:

-- 1. Wrong column order in composite index / Thứ tự cột sai trong composite index
CREATE INDEX IX_Orders_Bad_Order ON Orders (OrderDate, CustomerID);
-- Problem: Inefficient for WHERE CustomerID = X queries
-- Vấn đề: Không hiệu quả cho WHERE CustomerID = X queries

-- 2. Over-indexing / Tạo quá nhiều index
CREATE INDEX IX_Orders_CustomerID ON Orders (CustomerID);
CREATE INDEX IX_Orders_CustomerID_Status ON Orders (CustomerID, Status);
CREATE INDEX IX_Orders_CustomerID_Date ON Orders (CustomerID, OrderDate);
-- Problem: Redundant indexes, high maintenance cost
-- Vấn đề: Indexes dư thừa, chi phí bảo trì cao

-- 3. Wide indexes with too many included columns / Index rộng với quá nhiều included columns
CREATE INDEX IX_Orders_Everything
ON Orders (CustomerID)
INCLUDE (OrderDate, TotalAmount, Status, ShippingAddress, CreatedBy, CreatedDate, Notes, PaymentMethod);
-- Problem: Large index size, slower updates
-- Vấn đề: Index size lớn, updates chậm hơn

-- ✅ GOOD PRACTICES / THỰC HÀNH ĐÚNG:

-- 1. Correct column order: Most selective first / Thứ tự cột đúng: Selective nhất trước
CREATE INDEX IX_Orders_Good_Order ON Orders (CustomerID, OrderDate);
-- Better for both WHERE CustomerID = X AND WHERE CustomerID = X AND OrderDate = Y

-- 2. Consolidated covering index / Covering index hợp nhất
CREATE INDEX IX_Orders_Customer_Comprehensive
ON Orders (CustomerID, OrderDate DESC, Status)
INCLUDE (OrderID, TotalAmount);
-- Covers multiple query patterns efficiently / Cover nhiều pattern query hiệu quả

-- 3. Filtered indexes for specific scenarios / Filtered indexes cho scenarios cụ thể
CREATE INDEX IX_Orders_Recent_Active
ON Orders (CustomerID, OrderDate DESC)
INCLUDE (OrderID, TotalAmount, Status)
WHERE Status IN ('Pending', 'Processing') AND OrderDate >= '2024-01-01';
-- Smaller, more focused index / Index nhỏ hơn, tập trung hơn
```

#### 5.9. Index Maintenance Strategy / Chiến lược bảo trì Index

```sql
-- Automated index maintenance job / Job bảo trì index tự động
CREATE PROCEDURE sp_IndexMaintenance
    @FragmentationThreshold_Reorganize INT = 10,
    @FragmentationThreshold_Rebuild INT = 30,
    @MinPageCount INT = 100,
    @OnlineRebuild BIT = 1
AS
BEGIN
    SET NOCOUNT ON;

    DECLARE @SQL NVARCHAR(MAX);
    DECLARE @Message NVARCHAR(500);

    -- Create temp table để store maintenance actions
    CREATE TABLE #IndexMaintenance (
        TableName NVARCHAR(255),
        IndexName NVARCHAR(255),
        Fragmentation FLOAT,
        PageCount BIGINT,
        Action VARCHAR(20),
        SQL_Command NVARCHAR(MAX)
    );

    -- Identify indexes needing maintenance / Xác định indexes cần bảo trì
    INSERT INTO #IndexMaintenance (TableName, IndexName, Fragmentation, PageCount, Action, SQL_Command)
    SELECT
        OBJECT_NAME(ips.object_id) AS TableName,
        i.name AS IndexName,
        ips.avg_fragmentation_in_percent AS Fragmentation,
        ips.page_count AS PageCount,
        CASE
            WHEN ips.avg_fragmentation_in_percent >= @FragmentationThreshold_Rebuild THEN 'REBUILD'
            WHEN ips.avg_fragmentation_in_percent >= @FragmentationThreshold_Reorganize THEN 'REORGANIZE'
            ELSE 'NO_ACTION'
        END AS Action,
        CASE
            WHEN ips.avg_fragmentation_in_percent >= @FragmentationThreshold_Rebuild THEN
                'ALTER INDEX [' + i.name + '] ON [' + OBJECT_NAME(ips.object_id) + '] REBUILD' +
                CASE WHEN @OnlineRebuild = 1 THEN ' WITH (ONLINE = ON, SORT_IN_TEMPDB = ON)' ELSE '' END
            WHEN ips.avg_fragmentation_in_percent >= @FragmentationThreshold_Reorganize THEN
                'ALTER INDEX [' + i.name + '] ON [' + OBJECT_NAME(ips.object_id) + '] REORGANIZE'
            ELSE NULL
        END AS SQL_Command
    FROM sys.dm_db_index_physical_stats(DB_ID(), NULL, NULL, NULL, 'LIMITED') ips
    INNER JOIN sys.indexes i ON ips.object_id = i.object_id AND ips.index_id = i.index_id
    WHERE ips.avg_fragmentation_in_percent >= @FragmentationThreshold_Reorganize
      AND ips.page_count >= @MinPageCount
      AND i.name IS NOT NULL
      AND i.is_disabled = 0;

    -- Execute maintenance actions / Thực thi các actions bảo trì
    DECLARE maintenance_cursor CURSOR FOR
    SELECT SQL_Command, TableName, IndexName, Action, Fragmentation
    FROM #IndexMaintenance
    WHERE SQL_Command IS NOT NULL
    ORDER BY PageCount DESC;  -- Largest indexes first / Indexes lớn nhất trước

    DECLARE @SQLCmd NVARCHAR(MAX), @Table NVARCHAR(255), @Index NVARCHAR(255), @Action VARCHAR(20), @Frag FLOAT;

    OPEN maintenance_cursor;
    FETCH NEXT FROM maintenance_cursor INTO @SQLCmd, @Table, @Index, @Action, @Frag;

    WHILE @@FETCH_STATUS = 0
    BEGIN
        SET @Message = 'Executing ' + @Action + ' on ' + @Table + '.' + @Index +
                      ' (Fragmentation: ' + CAST(@Frag AS VARCHAR(10)) + '%)';
        PRINT @Message;

        BEGIN TRY
            EXEC sp_executesql @SQLCmd;
            PRINT 'SUCCESS: ' + @Message;
        END TRY
        BEGIN CATCH
            PRINT 'ERROR: ' + @Message + ' - ' + ERROR_MESSAGE();
        END CATCH

        FETCH NEXT FROM maintenance_cursor INTO @SQLCmd, @Table, @Index, @Action, @Frag;
    END

    CLOSE maintenance_cursor;
    DEALLOCATE maintenance_cursor;

    -- Summary report / Báo cáo tổng kết
    SELECT
        Action,
        COUNT(*) AS IndexCount,
        AVG(Fragmentation) AS AvgFragmentation,
        SUM(PageCount) AS TotalPages
    FROM #IndexMaintenance
    WHERE Action != 'NO_ACTION'
    GROUP BY Action
    ORDER BY Action;

    DROP TABLE #IndexMaintenance;
END;

-- Schedule to run during maintenance window / Lên lịch chạy trong maintenance window
-- EXEC sp_IndexMaintenance @FragmentationThreshold_Reorganize = 10, @FragmentationThreshold_Rebuild = 30;
```

**Tổng kết Index Optimization / Index Optimization Summary:**

1. **Analyze before creating / Phân tích trước khi tạo:** Use missing index DMVs, query plans
2. **Design for your queries / Thiết kế cho queries của bạn:** Cover common WHERE, ORDER BY, JOIN patterns
3. **Monitor usage / Giám sát sử dụng:** Remove unused indexes, optimize heavily used ones
4. **Maintain regularly / Bảo trì định kỳ:** Handle fragmentation, update statistics
5. **Test performance impact / Kiểm tra impact hiệu suất:** Measure before/after results
6. **Balance reads vs writes / Cân bằng reads vs writes:** More indexes = faster reads but slower writes

### 6. Query Hints / Gợi ý truy vấn

**Câu hỏi / Question:** Khi nào sử dụng query hints? / When to use query hints?

**Trả lời / Answer:** Query hints chỉ định cách SQL Server thực thi query, nhưng nên cẩn thận vì có thể ảnh hưởng performance.
**Answer:** Query hints specify how SQL Server executes queries, but should be used carefully as they can affect performance.

```sql
-- Ví dụ sử dụng hints / Example using hints
SELECT FirstName, LastName
FROM Employee WITH (INDEX(IX_Employee_LastName))
WHERE LastName LIKE 'S%';

-- Force index scan / Buộc quét chỉ mục
SELECT * FROM Employee WITH (INDEX(0))
WHERE Salary > 50000;
```

## Advanced Joins và Subqueries / JOIN và truy vấn con nâng cao

### 7. Self Join / Tự JOIN

**Câu hỏi / Question:** Self Join được sử dụng khi nào? / When is Self Join used?

**Trả lời / Answer:** Self Join dùng khi cần join một bảng với chính nó, thường cho hierarchical data.
**Answer:** Self Join is used when you need to join a table with itself, usually for hierarchical data.

```sql
-- Ví dụ: Tìm manager của mỗi nhân viên
-- Example: Find manager of each employee
SELECT
    e1.FirstName + ' ' + e1.LastName as EmployeeName,
    e2.FirstName + ' ' + e2.LastName as ManagerName
FROM Employee e1
LEFT JOIN Employee e2 ON e1.ManagerID = e2.EmployeeID;
```

### 8. Cross Apply và Outer Apply

**Câu hỏi / Question:** Sự khác biệt giữa CROSS APPLY và OUTER APPLY? / Difference between CROSS APPLY and OUTER APPLY?

**Trả lời / Answer:**

- **CROSS APPLY:** Tương tự INNER JOIN, chỉ trả về kết quả khi function trả về dữ liệu
- **OUTER APPLY:** Tương tự LEFT JOIN, trả về tất cả dữ liệu từ bảng bên trái

**Answer:**

- **CROSS APPLY:** Similar to INNER JOIN, only returns results when function returns data
- **OUTER APPLY:** Similar to LEFT JOIN, returns all data from left table

```sql
-- Ví dụ sử dụng APPLY / Example using APPLY
SELECT e.FirstName, e.LastName, f.FunctionResult
FROM Employee e
CROSS APPLY (SELECT dbo.GetEmployeeInfo(e.EmployeeID)) f(FunctionResult);
```

### 9. PIVOT và UNPIVOT

**Câu hỏi / Question:** PIVOT và UNPIVOT được sử dụng để làm gì? / What are PIVOT and UNPIVOT used for?

**Trả lời / Answer:**

- **PIVOT:** Chuyển đổi dữ liệu từ dạng dòng sang cột
- **UNPIVOT:** Chuyển đổi dữ liệu từ dạng cột sang dòng

**Answer:**

- **PIVOT:** Transforms data from row format to column format
- **UNPIVOT:** Transforms data from column format to row format

```sql
-- Ví dụ PIVOT / PIVOT example
SELECT DepartmentID, [IT], [HR], [Finance]
FROM (
    SELECT DepartmentID, DepartmentName, EmployeeCount
    FROM DepartmentStats
) AS SourceTable
PIVOT (
    SUM(EmployeeCount)
    FOR DepartmentName IN ([IT], [HR], [Finance])
) AS PivotTable;
```

## Stored Procedures và Functions / Thủ tục và hàm lưu trữ

### 10. Advanced Stored Procedures / Thủ tục lưu trữ nâng cao

**Câu hỏi / Question:** Viết stored procedure với error handling và transaction? / Write stored procedure with error handling and transaction?

```sql
CREATE PROCEDURE TransferSalary
    @FromEmployeeID INT,
    @ToEmployeeID INT,
    @Amount DECIMAL(10,2)
AS
BEGIN
    SET NOCOUNT ON;

    BEGIN TRY
        BEGIN TRANSACTION;

        -- Kiểm tra số dư / Check balance
        DECLARE @CurrentSalary DECIMAL(10,2);
        SELECT @CurrentSalary = Salary FROM Employee WHERE EmployeeID = @FromEmployeeID;

        IF @CurrentSalary < @Amount
            THROW 50001, 'Insufficient salary', 1;

        -- Thực hiện transfer / Perform transfer
        UPDATE Employee SET Salary = Salary - @Amount WHERE EmployeeID = @FromEmployeeID;
        UPDATE Employee SET Salary = Salary + @Amount WHERE EmployeeID = @ToEmployeeID;

        COMMIT TRANSACTION;

        SELECT 'Transfer completed successfully' as Result;
    END TRY
    BEGIN CATCH
        IF @@TRANCOUNT > 0
            ROLLBACK TRANSACTION;

        SELECT
            ERROR_MESSAGE() as ErrorMessage,
            ERROR_LINE() as ErrorLine;
    END CATCH
END
```

### 11. Table-Valued Functions / Hàm trả về bảng

**Câu hỏi / Question:** Viết table-valued function? / Write table-valued function?

```sql
CREATE FUNCTION GetEmployeesByDepartment
(
    @DepartmentID INT
)
RETURNS TABLE
AS
RETURN
(
    SELECT EmployeeID, FirstName, LastName, Salary
    FROM Employee
    WHERE DepartmentID = @DepartmentID
);

-- Sử dụng / Usage
SELECT * FROM GetEmployeesByDepartment(1);
```

## Advanced Data Types và Functions / Kiểu dữ liệu và hàm nâng cao

### 12. JSON Functions / Hàm JSON

**Câu hỏi / Question:** Làm thế nào để làm việc với JSON trong SQL Server? / How to work with JSON in SQL Server?

```sql
-- Tạo JSON từ query / Create JSON from query
SELECT EmployeeID, FirstName, LastName, Salary
FROM Employee
FOR JSON PATH;

-- Parse JSON / Phân tích JSON
DECLARE @json NVARCHAR(MAX) = '{"name": "John", "age": 30}';
SELECT JSON_VALUE(@json, '$.name') as Name,
       JSON_VALUE(@json, '$.age') as Age;
```

### 13. Temporal Tables / Bảng thời gian

**Câu hỏi / Question:** Temporal Tables là gì và khi nào sử dụng? / What are Temporal Tables and when to use them?

**Trả lời / Answer:** Temporal Tables cho phép lưu trữ lịch sử thay đổi dữ liệu theo thời gian.
**Answer:** Temporal Tables allow storing history of data changes over time.

```sql
-- Tạo temporal table / Create temporal table
CREATE TABLE EmployeeHistory
(
    EmployeeID INT PRIMARY KEY,
    FirstName VARCHAR(50),
    LastName VARCHAR(50),
    Salary DECIMAL(10,2),
    ValidFrom DATETIME2 GENERATED ALWAYS AS ROW START,
    ValidTo DATETIME2 GENERATED ALWAYS AS ROW END,
    PERIOD FOR SYSTEM_TIME (ValidFrom, ValidTo)
)
WITH (SYSTEM_VERSIONING = ON (HISTORY_TABLE = dbo.EmployeeHistoryArchive));
```

## Concurrency và Locking / Đồng thời và khóa

### 14. Isolation Levels / Mức cô lập

**Câu hỏi / Question:** Các isolation levels trong SQL Server? / Isolation levels in SQL Server?

**Trả lời / Answer:**

- **READ UNCOMMITTED:** Đọc dữ liệu chưa commit (dirty read)
- **READ COMMITTED:** Chỉ đọc dữ liệu đã commit (default)
- **REPEATABLE READ:** Đảm bảo đọc lại cùng kết quả
- **SERIALIZABLE:** Đảm bảo serializable execution
- **SNAPSHOT:** Đọc từ snapshot

**Answer:**

- **READ UNCOMMITTED:** Read uncommitted data (dirty read)
- **READ COMMITTED:** Only read committed data (default)
- **REPEATABLE READ:** Ensure reading same result again
- **SERIALIZABLE:** Ensure serializable execution
- **SNAPSHOT:** Read from snapshot

```sql
-- Ví dụ set isolation level / Example setting isolation level
SET TRANSACTION ISOLATION LEVEL READ COMMITTED;
BEGIN TRANSACTION;
    SELECT * FROM Employee WHERE EmployeeID = 1;
    -- Có thể đọc được thay đổi từ transaction khác đã commit
    -- Can read changes from other committed transactions
COMMIT;
```

### 15. Deadlock Prevention / Ngăn chặn deadlock

**Câu hỏi / Question:** Làm thế nào để tránh deadlock? / How to avoid deadlocks?

**Trả lời / Answer:**

- Sử dụng cùng thứ tự truy cập bảng / Use same order to access tables
- Giảm thời gian transaction / Reduce transaction time
- Sử dụng NOLOCK hint khi cần thiết / Use NOLOCK hint when necessary
- Monitoring deadlock events / Giám sát sự kiện deadlock

#### 15.1. Deadlock là gì và tại sao xảy ra? / What is deadlock and why does it occur?

**Deadlock** xảy ra khi hai hoặc nhiều transactions chờ đợi lẫn nhau để giải phóng locks.

#### 15.2. Ví dụ thực tế về Deadlock / Real-world deadlock examples

##### **Ví dụ 1: Hệ thống ngân hàng - Chuyển tiền giữa hai tài khoản**

```sql
-- Tạo bảng tài khoản
CREATE TABLE BankAccount (
    AccountID INT PRIMARY KEY,
    AccountNumber VARCHAR(20),
    Balance DECIMAL(10,2),
    LastUpdated DATETIME DEFAULT GETDATE()
);

-- Insert dữ liệu
INSERT INTO BankAccount VALUES
(1, 'ACC001', 10000.00, GETDATE()),
(2, 'ACC002', 5000.00, GETDATE());

-- ❌ Tình huống deadlock thực tế:
-- User A chuyển từ ACC001 sang ACC002
-- User B chuyển từ ACC002 sang ACC001 (cùng lúc)

-- Transaction A (User A)
BEGIN TRANSACTION;
    UPDATE BankAccount SET Balance = Balance - 1000 WHERE AccountID = 1; -- Lock ACC001
    -- Trong thực tế: Có thể có network delay, processing time, validation logic...
    UPDATE BankAccount SET Balance = Balance + 1000 WHERE AccountID = 2; -- Wait for ACC002
COMMIT;

-- Transaction B (User B) - chạy song song
BEGIN TRANSACTION;
    UPDATE BankAccount SET Balance = Balance - 500 WHERE AccountID = 2; -- Lock ACC002
    -- Trong thực tế: Có thể có network delay, processing time, validation logic...
    UPDATE BankAccount SET Balance = Balance + 500 WHERE AccountID = 1; -- Wait for ACC001
COMMIT;

-- Kết quả: DEADLOCK! Cả hai đều chờ nhau
```

##### **Ví dụ 2: Hệ thống đặt hàng - Cập nhật tồn kho và đơn hàng**

```sql
-- Tạo bảng sản phẩm và đơn hàng
CREATE TABLE Product (
    ProductID INT PRIMARY KEY,
    ProductName VARCHAR(100),
    StockQuantity INT,
    ReservedQuantity INT DEFAULT 0
);

CREATE TABLE OrderItem (
    OrderID INT,
    ProductID INT,
    Quantity INT,
    Status VARCHAR(20) DEFAULT 'Pending'
);

-- Insert dữ liệu
INSERT INTO Product VALUES (1, 'Laptop Dell', 10, 0);
INSERT INTO OrderItem VALUES (1, 1, 2, 'Pending');

-- ❌ Tình huống deadlock thực tế:
-- User A đặt hàng sản phẩm 1
-- User B hủy đơn hàng sản phẩm 1 (cùng lúc)

-- Transaction A (User A đặt hàng)
BEGIN TRANSACTION;
    -- Kiểm tra và reserve stock
    UPDATE Product
    SET ReservedQuantity = ReservedQuantity + 2,
        StockQuantity = StockQuantity - 2
    WHERE ProductID = 1 AND StockQuantity >= 2;

    -- Tạo order item
    INSERT INTO OrderItem (OrderID, ProductID, Quantity, Status)
    VALUES (2, 1, 2, 'Confirmed');

    -- Cập nhật status của order cũ
    UPDATE OrderItem SET Status = 'Confirmed' WHERE OrderID = 1;
COMMIT;

-- Transaction B (User B hủy đơn hàng) - chạy song song
BEGIN TRANSACTION;
    -- Cập nhật status trước
    UPDATE OrderItem SET Status = 'Cancelled' WHERE OrderID = 1;

    -- Trả lại stock
    UPDATE Product
    SET ReservedQuantity = ReservedQuantity - 2,
        StockQuantity = StockQuantity + 2
    WHERE ProductID = 1;
COMMIT;

-- Kết quả: DEADLOCK! Transaction A lock Product trước, Transaction B lock OrderItem trước
```

##### **Ví dụ 3: Hệ thống booking - Đặt phòng khách sạn**

```sql
-- Tạo bảng phòng và booking
CREATE TABLE HotelRoom (
    RoomID INT PRIMARY KEY,
    RoomNumber VARCHAR(10),
    IsAvailable BIT DEFAULT 1,
    CurrentBookingID INT NULL
);

CREATE TABLE Booking (
    BookingID INT PRIMARY KEY,
    RoomID INT,
    CustomerID INT,
    CheckInDate DATE,
    CheckOutDate DATE,
    Status VARCHAR(20) DEFAULT 'Confirmed'
);

-- Insert dữ liệu
INSERT INTO HotelRoom VALUES (1, '101', 1, NULL);
INSERT INTO Booking VALUES (1, 1, 100, '2024-01-01', '2024-01-03', 'Confirmed');

-- ❌ Tình huống deadlock thực tế:
-- User A muốn extend booking
-- User B muốn cancel booking (cùng lúc)

-- Transaction A (User A extend booking)
BEGIN TRANSACTION;
    -- Kiểm tra phòng có available không
    SELECT * FROM HotelRoom WHERE RoomID = 1 AND IsAvailable = 1;

    -- Cập nhật booking
    UPDATE Booking
    SET CheckOutDate = '2024-01-05'
    WHERE BookingID = 1;

    -- Cập nhật trạng thái phòng
    UPDATE HotelRoom
    SET CurrentBookingID = 1
    WHERE RoomID = 1;
COMMIT;

-- Transaction B (User B hủy đơn hàng) - chạy song song
BEGIN TRANSACTION;
    -- Cập nhật trạng thái phòng trước
    UPDATE HotelRoom
    SET IsAvailable = 1, CurrentBookingID = NULL
    WHERE RoomID = 1;

    -- Cập nhật booking
    UPDATE Booking
    SET Status = 'Cancelled'
    WHERE BookingID = 1;
COMMIT;

-- Kết quả: DEADLOCK! Khác thứ tự truy cập
```

#### 15.3. Cách phát hiện Deadlock trong thực tế / Real-world deadlock detection

```sql
-- Monitor deadlocks trong production
CREATE EVENT SESSION [Production_Deadlock_Monitoring] ON SERVER
ADD EVENT sqlserver.xml_deadlock_report
ADD TARGET package0.event_file(SET filename=N'C:\XEvents\production_deadlock.xel')
WITH (MAX_MEMORY=4096 KB, EVENT_RETENTION_MODE=ALLOW_SINGLE_EVENT_LOSS);

-- Query để analyze deadlocks
SELECT
    event_data.value('(event/@timestamp)[1]', 'DATETIME2') AS DeadlockTime,
    event_data.value('(event/data[@name="xml_report"]/value)[1]', 'NVARCHAR(MAX)') AS DeadlockXML,
    event_data.value('(event/action[@name="database_name"]/value)[1]', 'NVARCHAR(128)') AS DatabaseName
FROM (
    SELECT CAST(event_data AS XML) AS event_data
    FROM sys.fn_xe_file_target_read_file('C:\XEvents\production_deadlock*.xel', NULL, NULL, NULL)
) AS events
WHERE event_data.value('(event/@timestamp)[1]', 'DATETIME2') >= DATEADD(HOUR, -24, GETDATE());
```

#### 15.4. Giải pháp thực tế cho từng ví dụ / Real solutions for each example

##### **Giải pháp cho Ví dụ 1 (Ngân hàng):**

```sql
-- ✅ Giải pháp: Luôn lock theo thứ tự AccountID tăng dần
CREATE PROCEDURE TransferMoney
    @FromAccountID INT,
    @ToAccountID INT,
    @Amount DECIMAL(10,2)
AS
BEGIN
    SET NOCOUNT ON;

    -- Luôn lock account có ID nhỏ hơn trước
    DECLARE @FirstAccountID INT = CASE WHEN @FromAccountID < @ToAccountID
                                       THEN @FromAccountID ELSE @ToAccountID END;
    DECLARE @SecondAccountID INT = CASE WHEN @FromAccountID < @ToAccountID
                                        THEN @ToAccountID ELSE @FromAccountID END;

    BEGIN TRANSACTION;
        -- Lock theo thứ tự nhất quán
        UPDATE BankAccount
        SET Balance = CASE
            WHEN AccountID = @FromAccountID THEN Balance - @Amount
            WHEN AccountID = @ToAccountID THEN Balance + @Amount
            ELSE Balance
        END,
        LastUpdated = GETDATE()
        WHERE AccountID IN (@FirstAccountID, @SecondAccountID);

    COMMIT TRANSACTION;
END;
```

##### **Giải pháp cho Ví dụ 2 (Đặt hàng):**

```sql
-- ✅ Giải pháp: Sử dụng optimistic locking
CREATE PROCEDURE PlaceOrder
    @ProductID INT,
    @Quantity INT,
    @OrderID INT
AS
BEGIN
    SET NOCOUNT ON;

    DECLARE @RetryCount INT = 0;
    DECLARE @MaxRetries INT = 3;
    DECLARE @Success BIT = 0;

    WHILE @RetryCount < @MaxRetries AND @Success = 0
    BEGIN
        BEGIN TRY
            BEGIN TRANSACTION;
                -- Kiểm tra stock trước
                IF (SELECT StockQuantity FROM Product WHERE ProductID = @ProductID) >= @Quantity
                BEGIN
                    -- Cập nhật stock
                    UPDATE Product
                    SET StockQuantity = StockQuantity - @Quantity,
                        ReservedQuantity = ReservedQuantity + @Quantity
                    WHERE ProductID = @ProductID;

                    -- Tạo order
                    INSERT INTO OrderItem (OrderID, ProductID, Quantity, Status)
                    VALUES (@OrderID, @ProductID, @Quantity, 'Confirmed');

                    SET @Success = 1;
                END
                ELSE
                BEGIN
                    THROW 50001, 'Insufficient stock', 1;
                END
            COMMIT TRANSACTION;
        END TRY
        BEGIN CATCH
            IF @@TRANCOUNT > 0
                ROLLBACK TRANSACTION;

            IF ERROR_NUMBER() = 1205 -- Deadlock
            BEGIN
                SET @RetryCount = @RetryCount + 1;
                WAITFOR DELAY '00:00:00.' + CAST((@RetryCount * 100) AS VARCHAR(3));
            END
            ELSE
                THROW;
        END CATCH
    END

    IF @Success = 0
        THROW 50002, 'Failed after maximum retries', 1;
END;
```

##### **Giải pháp cho Ví dụ 3 (Booking):**

```sql
-- ✅ Giải pháp: Sử dụng application-level locking
CREATE PROCEDURE UpdateBooking
    @BookingID INT,
    @NewCheckOutDate DATE = NULL,
    @NewStatus VARCHAR(20) = NULL
AS
BEGIN
    SET NOCOUNT ON;

    DECLARE @RoomID INT;
    SELECT @RoomID = RoomID FROM Booking WHERE BookingID = @BookingID;

    -- Luôn lock Room trước, Booking sau
    BEGIN TRANSACTION;
        -- Lock room trước
        UPDATE HotelRoom
        SET IsAvailable = CASE WHEN @NewStatus = 'Cancelled' THEN 1 ELSE 0 END,
            CurrentBookingID = CASE WHEN @NewStatus = 'Cancelled' THEN NULL ELSE @BookingID END
        WHERE RoomID = @RoomID;

        -- Sau đó lock booking
        UPDATE Booking
        SET CheckOutDate = ISNULL(@NewCheckOutDate, CheckOutDate),
            Status = ISNULL(@NewStatus, Status)
        WHERE BookingID = @BookingID;

    COMMIT TRANSACTION;
END;
```

#### 15.5. Monitoring Deadlocks trong Production / Production deadlock monitoring

```sql
-- Tạo bảng log deadlocks
CREATE TABLE DeadlockLog (
    DeadlockID INT IDENTITY(1,1) PRIMARY KEY,
    DeadlockTime DATETIME2 DEFAULT GETDATE(),
    DatabaseName NVARCHAR(128),
    AffectedTables NVARCHAR(500),
    VictimProcess NVARCHAR(50),
    BlockingProcess NVARCHAR(50),
    DeadlockXML NVARCHAR(MAX),
    ResolutionTime DATETIME2,
    IsResolved BIT DEFAULT 0
);

-- Stored procedure để log và alert deadlocks
CREATE PROCEDURE LogAndAlertDeadlock
    @DeadlockXML NVARCHAR(MAX),
    @DatabaseName NVARCHAR(128) = NULL
AS
BEGIN
    DECLARE @AffectedTables NVARCHAR(500) = '';
    DECLARE @VictimProcess NVARCHAR(50) = '';
    DECLARE @BlockingProcess NVARCHAR(50) = '';

    -- Parse deadlock XML để extract thông tin
    -- (Đây là logic đơn giản, trong thực tế cần parse XML phức tạp hơn)

    INSERT INTO DeadlockLog (
        DatabaseName, AffectedTables, VictimProcess,
        BlockingProcess, DeadlockXML
    )
    VALUES (
        ISNULL(@DatabaseName, DB_NAME()),
        @AffectedTables, @VictimProcess, @BlockingProcess, @DeadlockXML
    );

    -- Send alert (implement your alerting mechanism)
    -- Có thể gửi email, Slack notification, etc.
    PRINT 'DEADLOCK DETECTED! Check DeadlockLog table for details.';

    -- Log to application log
    EXEC sp_executesql N'
        INSERT INTO ApplicationLog (LogLevel, Message, Details)
        VALUES (''ERROR'', ''Deadlock detected'', @DeadlockXML)
    ', N'@DeadlockXML NVARCHAR(MAX)', @DeadlockXML;
END;
```

#### 15.6. Best Practices thực tế / Real-world best practices

```sql
-- 1. Luôn sử dụng cùng thứ tự truy cập bảng
-- 2. Implement retry logic cho critical operations
-- 3. Sử dụng appropriate isolation levels
-- 4. Monitor và alert deadlocks
-- 5. Có contingency plan khi deadlock xảy ra

-- Ví dụ: Comprehensive deadlock handling
CREATE PROCEDURE SafeBankTransfer
    @FromAccountID INT,
    @ToAccountID INT,
    @Amount DECIMAL(10,2),
    @MaxRetries INT = 3
AS
BEGIN
    SET NOCOUNT ON;

    DECLARE @RetryCount INT = 0;
    DECLARE @Success BIT = 0;
    DECLARE @ErrorMessage NVARCHAR(4000);

    WHILE @RetryCount < @MaxRetries AND @Success = 0
    BEGIN
        BEGIN TRY
            BEGIN TRANSACTION;
                -- Validate accounts
                IF NOT EXISTS (SELECT 1 FROM BankAccount WHERE AccountID = @FromAccountID)
                    THROW 50001, 'From account not found', 1;

                IF NOT EXISTS (SELECT 1 FROM BankAccount WHERE AccountID = @ToAccountID)
                    THROW 50002, 'To account not found', 1;

                -- Check balance
                DECLARE @CurrentBalance DECIMAL(10,2);
                SELECT @CurrentBalance = Balance FROM BankAccount WHERE AccountID = @FromAccountID;

                IF @CurrentBalance < @Amount
                    THROW 50003, 'Insufficient balance', 1;

                -- Transfer money (luôn theo thứ tự AccountID)
                DECLARE @FirstAccountID INT = CASE WHEN @FromAccountID < @ToAccountID
                                                   THEN @FromAccountID ELSE @ToAccountID END;
                DECLARE @SecondAccountID INT = CASE WHEN @FromAccountID < @ToAccountID
                                                    THEN @ToAccountID ELSE @FromAccountID END;

                UPDATE BankAccount
                SET Balance = CASE
                    WHEN AccountID = @FromAccountID THEN Balance - @Amount
                    WHEN AccountID = @ToAccountID THEN Balance + @Amount
                    ELSE Balance
                END,
                LastUpdated = GETDATE()
                WHERE AccountID IN (@FirstAccountID, @SecondAccountID);

                -- Log transaction
                INSERT INTO TransactionLog (
                    FromAccountID, ToAccountID, Amount, TransactionDate
                )
                VALUES (@FromAccountID, @ToAccountID, @Amount, GETDATE());

            COMMIT TRANSACTION;
            SET @Success = 1;

        END TRY
        BEGIN CATCH
            IF @@TRANCOUNT > 0
                ROLLBACK TRANSACTION;

            SET @ErrorMessage = ERROR_MESSAGE();

            IF ERROR_NUMBER() = 1205 -- Deadlock
            BEGIN
                SET @RetryCount = @RetryCount + 1;

                -- Exponential backoff
                WAITFOR DELAY '00:00:00.' + CAST((@RetryCount * 200) AS VARCHAR(3));

                -- Log retry attempt
                INSERT INTO RetryLog (Operation, RetryCount, ErrorMessage, RetryTime)
                VALUES ('BankTransfer', @RetryCount, @ErrorMessage, GETDATE());
            END
            ELSE
            BEGIN
                -- Non-deadlock error, don't retry
                THROW;
            END
        END CATCH
    END

    IF @Success = 0
    BEGIN
        -- Log final failure
        INSERT INTO FailureLog (Operation, ErrorMessage, FinalRetryCount)
        VALUES ('BankTransfer', @ErrorMessage, @RetryCount);

        THROW 50004, 'Transfer failed after maximum retries', 1;
    END
END;
```

#### 15.7. Tổng kết / Summary

**Deadlock trong thực tế thường xảy ra khi:**

- Nhiều users cùng thao tác trên cùng dữ liệu
- Network delays, processing time tạo ra timing issues
- Khác thứ tự truy cập bảng giữa các transactions
- Long-running transactions

**Cách phòng tránh hiệu quả:**

- ✅ Luôn truy cập bảng theo thứ tự nhất quán
- ✅ Implement retry logic với exponential backoff
- ✅ Giữ transactions ngắn và focused
- ✅ Sử dụng appropriate isolation levels
- ✅ Monitor và alert deadlocks trong production
- ✅ Có contingency plan khi deadlock xảy ra

**Ghi nhớ:** Deadlock là vấn đề phổ biến trong production systems có nhiều concurrent users. Việc hiểu rõ nguyên nhân và áp dụng các best practices sẽ giúp giảm thiểu đáng kể tần suất xảy ra deadlock.

## Advanced Queries / Truy vấn nâng cao

### 16. Dynamic SQL / SQL động

**Câu hỏi / Question:** Khi nào sử dụng Dynamic SQL và cách implement? / When to use Dynamic SQL and how to implement?

```sql
-- Ví dụ Dynamic SQL / Dynamic SQL example
CREATE PROCEDURE SearchEmployees
    @SearchColumn VARCHAR(50),
    @SearchValue VARCHAR(100)
AS
BEGIN
    DECLARE @SQL NVARCHAR(MAX);
    SET @SQL = 'SELECT * FROM Employee WHERE ' + @SearchColumn + ' = @Value';

    EXEC sp_executesql @SQL, N'@Value VARCHAR(100)', @SearchValue;
END
```

### 17. Complex Aggregations / Tổng hợp phức tạp

**Câu hỏi / Question:** Viết query tính toán phức tạp? / Write complex calculation query?

```sql
-- Ví dụ: Tính tỷ lệ lương trung bình của mỗi phòng ban so với toàn công ty
-- Example: Calculate average salary ratio of each department compared to company
WITH DeptStats AS (
    SELECT
        DepartmentID,
        AVG(Salary) as AvgDeptSalary,
        COUNT(*) as EmployeeCount
    FROM Employee
    GROUP BY DepartmentID
),
CompanyStats AS (
    SELECT AVG(Salary) as AvgCompanySalary
    FROM Employee
)
SELECT
    d.DepartmentName,
    ds.AvgDeptSalary,
    cs.AvgCompanySalary,
    (ds.AvgDeptSalary / cs.AvgCompanySalary * 100) as SalaryRatio
FROM DeptStats ds
CROSS JOIN CompanyStats cs
INNER JOIN Department d ON ds.DepartmentID = d.DepartmentID;
```

## Advanced Performance Tuning / Tối ưu hóa hiệu suất nâng cao

### 18. Query Store và Performance Monitoring / Query Store and Performance Monitoring

**Câu hỏi / Question:** Làm thế nào để monitor và optimize query performance sử dụng Query Store? / How to monitor and optimize query performance using Query Store?

```sql
-- Enable Query Store / Bật Query Store
ALTER DATABASE YourDatabase
SET QUERY_STORE = ON
(
    OPERATION_MODE = READ_WRITE,
    CLEANUP_POLICY = (STALE_QUERY_THRESHOLD_DAYS = 30),
    DATA_FLUSH_INTERVAL_SECONDS = 3000
);

-- Analyze query performance / Phân tích hiệu suất truy vấn
SELECT
    qs.query_id,
    qt.query_sql_text,
    qs.avg_duration,
    qs.execution_count,
    qs.avg_cpu_time,
    qs.avg_logical_io_reads
FROM sys.query_store_query qs
INNER JOIN sys.query_store_query_text qt ON qs.query_text_id = qt.query_text_id
WHERE qs.avg_duration > 1000000
ORDER BY qs.avg_duration DESC;
```

### 19. Index Optimization Strategies / Chiến lược tối ưu hóa chỉ mục

**Câu hỏi / Question:** Làm thế nào để tối ưu hóa indexes cho performance? / How to optimize indexes for performance?

```sql
-- Index fragmentation analysis / Phân tích phân mảnh chỉ mục
SELECT
    OBJECT_NAME(ind.OBJECT_ID) AS TableName,
    ind.name AS IndexName,
    indexstats.avg_fragmentation_in_percent,
    indexstats.page_count,
    indexstats.avg_page_space_used_in_percent
FROM sys.dm_db_index_physical_stats(DB_ID(), NULL, NULL, NULL, NULL) indexstats
INNER JOIN sys.indexes ind ON ind.object_id = indexstats.object_id
    AND ind.index_id = indexstats.index_id
WHERE indexstats.avg_fragmentation_in_percent > 10
ORDER BY indexstats.avg_fragmentation_in_percent DESC;

-- Missing index suggestions / Gợi ý chỉ mục thiếu
SELECT
    dm_migs.avg_user_impact,
    dm_migs.last_user_seek,
    dm_mid.statement AS TableName,
    dm_migs.unique_compiles,
    dm_migs.user_seeks,
    dm_migs.avg_total_user_cost,
    dm_migs.avg_user_impact,
    dm_mid.equality_columns,
    dm_mid.inequality_columns,
    dm_mid.included_columns
FROM sys.dm_db_missing_index_group_stats dm_migs
INNER JOIN sys.dm_db_missing_index_groups dm_mig ON dm_migs.group_handle = dm_mig.index_group_handle
INNER JOIN sys.dm_db_missing_index_details dm_mid ON dm_mig.index_handle = dm_mid.index_handle
ORDER BY dm_migs.avg_user_impact DESC;
```

### 20. Statistics Management / Quản lý thống kê

**Câu hỏi / Question:** Tại sao statistics quan trọng và làm thế nào để quản lý? / Why are statistics important and how to manage them?

```sql
-- Check statistics / Kiểm tra thống kê
SELECT
    s.name AS StatisticsName,
    s.auto_created,
    s.user_created,
    s.no_recompute,
    sp.last_updated,
    sp.rows,
    sp.rows_sampled,
    sp.steps
FROM sys.stats s
INNER JOIN sys.stats_columns sc ON s.object_id = sc.object_id AND s.stats_id = sc.stats_id
INNER JOIN sys.columns c ON sc.object_id = c.object_id AND sc.column_id = c.column_id
CROSS APPLY sys.dm_db_stats_properties(s.object_id, s.stats_id) sp
WHERE s.object_id = OBJECT_ID('Employee');

-- Update statistics / Cập nhật thống kê
UPDATE STATISTICS Employee WITH FULLSCAN;
UPDATE STATISTICS Employee(DepartmentID) WITH SAMPLE 50 PERCENT;
```

## Advanced Data Types and Functions / Kiểu dữ liệu và hàm nâng cao

### 21. JSON Functions / Hàm JSON

**Câu hỏi / Question:** Làm thế nào để làm việc với JSON trong SQL Server? / How to work with JSON in SQL Server?

```sql
-- Tạo JSON từ query / Create JSON from query
SELECT
    EmployeeID,
    FirstName + ' ' + LastName AS FullName,
    Salary,
    DepartmentID
FROM Employee
FOR JSON PATH, ROOT('Employees');

-- Parse JSON / Phân tích JSON
DECLARE @json NVARCHAR(MAX) = '{"name": "John", "age": 30, "department": "IT"}';
SELECT
    JSON_VALUE(@json, '$.name') AS Name,
    JSON_VALUE(@json, '$.age') AS Age,
    JSON_VALUE(@json, '$.department') AS Department;

-- JSON với nested objects / JSON with nested objects
SELECT
    d.DepartmentName,
    (SELECT
        e.FirstName + ' ' + e.LastName AS FullName,
        e.Salary
     FROM Employee e
     WHERE e.DepartmentID = d.DepartmentID
     FOR JSON PATH) AS Employees
FROM Department d
FOR JSON PATH;
```

### 22. Temporal Tables / Bảng thời gian

**Câu hỏi / Question:** Temporal Tables là gì và khi nào sử dụng? / What are Temporal Tables and when to use them?

```sql
-- Tạo temporal table / Create temporal table
CREATE TABLE EmployeeHistory
(
    EmployeeID INT PRIMARY KEY,
    FirstName VARCHAR(50),
    LastName VARCHAR(50),
    Salary DECIMAL(10,2),
    DepartmentID INT,
    ValidFrom DATETIME2 GENERATED ALWAYS AS ROW START,
    ValidTo DATETIME2 GENERATED ALWAYS AS ROW END,
    PERIOD FOR SYSTEM_TIME (ValidFrom, ValidTo)
)
WITH (SYSTEM_VERSIONING = ON (HISTORY_TABLE = dbo.EmployeeHistoryArchive));

-- Query temporal data / Truy vấn dữ liệu thời gian
SELECT * FROM EmployeeHistory
FOR SYSTEM_TIME BETWEEN '2023-01-01' AND '2023-12-31'
WHERE EmployeeID = 1;

-- Query point in time / Truy vấn tại thời điểm cụ thể
SELECT * FROM EmployeeHistory
FOR SYSTEM_TIME AS OF '2023-06-15 10:00:00'
WHERE EmployeeID = 1;
```

### 23. Window Functions nâng cao / Advanced Window Functions

**Câu hỏi / Question:** Làm thế nào để sử dụng window functions cho phân tích phức tạp? / How to use window functions for complex analysis?

```sql
-- Running totals và moving averages / Running totals and moving averages
SELECT
    EmployeeID,
    FirstName + ' ' + LastName AS FullName,
    Salary,
    DepartmentID,
    SUM(Salary) OVER (PARTITION BY DepartmentID ORDER BY Salary) AS RunningTotal,
    AVG(Salary) OVER (PARTITION BY DepartmentID ORDER BY Salary ROWS BETWEEN 2 PRECEDING AND CURRENT ROW) AS MovingAvg,
    ROW_NUMBER() OVER (PARTITION BY DepartmentID ORDER BY Salary DESC) AS SalaryRank,
    LAG(Salary, 1) OVER (PARTITION BY DepartmentID ORDER BY Salary) AS PreviousSalary,
    LEAD(Salary, 1) OVER (PARTITION BY DepartmentID ORDER BY Salary) AS NextSalary
FROM Employee;

-- Percentile calculations / Tính toán percentile
SELECT
    DepartmentID,
    Salary,
    PERCENT_RANK() OVER (PARTITION BY DepartmentID ORDER BY Salary) AS PercentRank,
    NTILE(4) OVER (PARTITION BY DepartmentID ORDER BY Salary) AS Quartile
FROM Employee;
```

## Advanced Stored Procedures and Functions / Thủ tục và hàm lưu trữ nâng cao

### 24. Table-Valued Functions / Hàm trả về bảng

**Câu hỏi / Question:** Viết table-valued function phức tạp? / Write complex table-valued function?

```sql
-- Inline table-valued function / Hàm trả về bảng inline
CREATE FUNCTION GetEmployeeHierarchy
(
    @ManagerID INT
)
RETURNS TABLE
AS
RETURN
(
    WITH EmployeeCTE AS (
        SELECT EmployeeID, FirstName, LastName, ManagerID, 0 AS Level
        FROM Employee
        WHERE ManagerID = @ManagerID

        UNION ALL

        SELECT e.EmployeeID, e.FirstName, e.LastName, e.ManagerID, cte.Level + 1
        FROM Employee e
        INNER JOIN EmployeeCTE cte ON e.ManagerID = cte.EmployeeID
    )
    SELECT
        EmployeeID,
        FirstName + ' ' + LastName AS FullName,
        ManagerID,
        Level,
        REPLICATE('  ', Level) + FirstName + ' ' + LastName AS HierarchyDisplay
    FROM EmployeeCTE
);

-- Multi-statement table-valued function / Hàm trả về bảng multi-statement
CREATE FUNCTION GetDepartmentEmployees
(
    @DepartmentID INT,
    @MinSalary DECIMAL(10,2) = 0
)
RETURNS @Result TABLE
(
    EmployeeID INT,
    FullName NVARCHAR(100),
    Salary DECIMAL(10,2),
    YearsOfService INT
)
AS
BEGIN
    INSERT INTO @Result
    SELECT
        e.EmployeeID,
        e.FirstName + ' ' + e.LastName,
        e.Salary,
        DATEDIFF(YEAR, e.HireDate, GETDATE())
    FROM Employee e
    WHERE e.DepartmentID = @DepartmentID
        AND e.Salary >= @MinSalary;

    RETURN;
END;
```

### 25. Advanced Stored Procedures với Error Handling / Advanced Stored Procedures with Error Handling

**Câu hỏi / Question:** Viết stored procedure với comprehensive error handling? / Write stored procedure with comprehensive error handling?

```sql
CREATE PROCEDURE TransferEmployee
    @EmployeeID INT,
    @NewDepartmentID INT,
    @NewSalary DECIMAL(10,2) = NULL,
    @EffectiveDate DATE = NULL
AS
BEGIN
    SET NOCOUNT ON;

    DECLARE @ErrorMessage NVARCHAR(4000);
    DECLARE @ErrorSeverity INT;
    DECLARE @ErrorState INT;
    DECLARE @OldDepartmentID INT;
    DECLARE @OldSalary DECIMAL(10,2);

    BEGIN TRY
        -- Validate input parameters / Kiểm tra tham số đầu vào
        IF @EmployeeID IS NULL
            THROW 50001, 'EmployeeID cannot be NULL', 1;

        IF @NewDepartmentID IS NULL
            THROW 50002, 'NewDepartmentID cannot be NULL', 1;

        IF @EffectiveDate IS NULL
            SET @EffectiveDate = GETDATE();

        -- Check if employee exists / Kiểm tra nhân viên có tồn tại
        IF NOT EXISTS (SELECT 1 FROM Employee WHERE EmployeeID = @EmployeeID)
            THROW 50003, 'Employee does not exist', 1;

        -- Check if department exists / Kiểm tra phòng ban có tồn tại
        IF NOT EXISTS (SELECT 1 FROM Department WHERE DepartmentID = @NewDepartmentID)
            THROW 50004, 'Department does not exist', 1;

        -- Get current values / Lấy giá trị hiện tại
        SELECT @OldDepartmentID = DepartmentID, @OldSalary = Salary
        FROM Employee WHERE EmployeeID = @EmployeeID;

        -- Begin transaction / Bắt đầu transaction
        BEGIN TRANSACTION;

        -- Update employee / Cập nhật nhân viên
        UPDATE Employee
        SET
            DepartmentID = @NewDepartmentID,
            Salary = ISNULL(@NewSalary, Salary),
            UpdatedAt = GETDATE()
        WHERE EmployeeID = @EmployeeID;

        -- Log the transfer / Ghi log chuyển đổi
        INSERT INTO EmployeeTransferLog (
            EmployeeID,
            OldDepartmentID,
            NewDepartmentID,
            OldSalary,
            NewSalary,
            TransferDate
        )
        VALUES (
            @EmployeeID,
            @OldDepartmentID,
            @NewDepartmentID,
            @OldSalary,
            ISNULL(@NewSalary, @OldSalary),
            @EffectiveDate
        );

        -- Update department budgets / Cập nhật ngân sách phòng ban
        UPDATE Department
        SET Budget = Budget - @OldSalary
        WHERE DepartmentID = @OldDepartmentID;

        UPDATE Department
        SET Budget = Budget + ISNULL(@NewSalary, @OldSalary)
        WHERE DepartmentID = @NewDepartmentID;

        COMMIT TRANSACTION;

        -- Return success message / Trả về thông báo thành công
        SELECT
            'Employee transferred successfully' AS Result,
            @EmployeeID AS EmployeeID,
            @OldDepartmentID AS OldDepartmentID,
            @NewDepartmentID AS NewDepartmentID;

    END TRY
    BEGIN CATCH
        IF @@TRANCOUNT > 0
            ROLLBACK TRANSACTION;

        SELECT
            ERROR_MESSAGE() AS ErrorMessage,
            ERROR_LINE() AS ErrorLine,
            ERROR_NUMBER() AS ErrorNumber,
            ERROR_PROCEDURE() AS ErrorProcedure;
    END CATCH
END;
```

## Advanced Query Optimization / Tối ưu hóa truy vấn nâng cao

### 26. Query Hints và Plan Guides / Query Hints and Plan Guides

**Câu hỏi / Question:** Khi nào sử dụng query hints và plan guides? / When to use query hints and plan guides?

```sql
-- Query hints / Gợi ý truy vấn
SELECT e.FirstName, e.LastName, d.DepartmentName
FROM Employee e
INNER JOIN Department d ON e.DepartmentID = d.DepartmentID
WHERE e.Salary > 50000
OPTION (RECOMPILE, MAXDOP 4);

-- Plan guide / Hướng dẫn kế hoạch
EXEC sp_create_plan_guide
    @name = N'EmployeeQuery_PlanGuide',
    @stmt = N'SELECT e.FirstName, e.LastName, d.DepartmentName
               FROM Employee e
               INNER JOIN Department d ON e.DepartmentID = d.DepartmentID
               WHERE e.Salary > @Salary',
    @type = N'SQL',
    @module_or_batch = NULL,
    @params = N'@Salary DECIMAL(10,2)',
    @hints = N'OPTION (OPTIMIZE FOR (@Salary = 50000))';
```

### 27. Partitioned Tables / Bảng phân vùng

**Câu hỏi / Question:** Làm thế nào để implement partitioned tables? / How to implement partitioned tables?

```sql
-- Create partition function / Tạo hàm phân vùng
CREATE PARTITION FUNCTION PF_DateRange (DATETIME)
AS RANGE RIGHT FOR VALUES
('2023-01-01', '2023-04-01', '2023-07-01', '2023-10-01', '2024-01-01');

-- Create partition scheme / Tạo scheme phân vùng
CREATE PARTITION SCHEME PS_DateRange
AS PARTITION PF_DateRange
TO (FG1, FG2, FG3, FG4, FG5, FG6);

-- Create partitioned table / Tạo bảng phân vùng
CREATE TABLE EmployeeHistory_Partitioned
(
    EmployeeID INT,
    FirstName VARCHAR(50),
    LastName VARCHAR(50),
    Salary DECIMAL(10,2),
    ChangeDate DATETIME,
    ChangeType VARCHAR(20)
)
ON PS_DateRange(ChangeDate);
```

## Advanced Monitoring và Troubleshooting / Giám sát và xử lý sự cố nâng cao

### 28. Extended Events / Sự kiện mở rộng

**Câu hỏi / Question:** Setup monitoring với Extended Events? / Setup monitoring with Extended Events?

```sql
-- Create event session / Tạo phiên sự kiện
CREATE EVENT SESSION [Performance_Monitoring] ON SERVER
ADD EVENT sqlserver.sql_statement_completed
(
    ACTION (
        sqlserver.sql_text,
        sqlserver.database_name,
        sqlserver.username,
        sqlserver.client_hostname
    )
    WHERE duration > 1000000 -- Queries taking more than 1 second
),
ADD EVENT sqlserver.deadlock_graph
ADD TARGET package0.event_file(SET filename=N'C:\XEvents\performance.xel')
WITH (MAX_MEMORY=4096 KB, EVENT_RETENTION_MODE=ALLOW_SINGLE_EVENT_LOSS);

-- Start session / Bắt đầu phiên
ALTER EVENT SESSION [Performance_Monitoring] ON SERVER STATE = START;

-- Query event data / Truy vấn dữ liệu sự kiện
SELECT
    event_data.value('(event/@timestamp)[1]', 'DATETIME2') AS EventTime,
    event_data.value('(event/action[@name="sql_text"]/value)[1]', 'NVARCHAR(MAX)') AS SQLText,
    event_data.value('(event/data[@name="duration"]/value)[1]', 'BIGINT') AS Duration
FROM (
    SELECT CAST(event_data AS XML) AS event_data
    FROM sys.fn_xe_file_target_read_file('C:\XEvents\performance*.xel', NULL, NULL, NULL)
) AS events;
```

### 29. DMVs cho Performance Analysis / DMVs for Performance Analysis

**Câu hỏi / Question:** Sử dụng DMVs để analyze performance issues? / Use DMVs to analyze performance issues?

```sql
-- Top queries by CPU / Top truy vấn theo CPU
SELECT TOP 10
    qs.sql_handle,
    qs.execution_count,
    qs.total_worker_time,
    qs.total_elapsed_time,
    qs.total_logical_reads,
    qs.total_physical_reads,
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
    max_wait_time_ms,
    signal_wait_time_ms,
    wait_time_ms - signal_wait_time_ms AS resource_wait_time_ms
FROM sys.dm_os_wait_stats
WHERE wait_time_ms > 0
ORDER BY wait_time_ms DESC;

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
    t.text AS sql_text,
    p.query_plan
FROM sys.dm_exec_sessions s
INNER JOIN sys.dm_exec_requests r ON s.session_id = r.session_id
CROSS APPLY sys.dm_exec_sql_text(r.sql_handle) t
CROSS APPLY sys.dm_exec_query_plan(r.plan_handle) p
WHERE r.blocking_session_id > 0;
```

### 30. Memory và Buffer Pool Analysis / Phân tích bộ nhớ và buffer pool

**Câu hỏi / Question:** Làm thế nào để analyze memory pressure? / How to analyze memory pressure?

```sql
-- Buffer pool usage / Sử dụng buffer pool
SELECT
    DB_NAME(database_id) AS DatabaseName,
    COUNT(*) AS PageCount,
    COUNT(*) * 8 AS SizeKB
FROM sys.dm_os_buffer_descriptors
GROUP BY database_id
ORDER BY PageCount DESC;

-- Memory grants / Cấp phát bộ nhớ
SELECT
    request_id,
    session_id,
    requested_memory_kb,
    granted_memory_kb,
    used_memory_kb,
    max_used_memory_kb,
    ideal_memory_kb
FROM sys.dm_exec_query_memory_grants
WHERE granted_memory_kb > 0
ORDER BY granted_memory_kb DESC;

-- Memory clerks / Memory clerks
SELECT
    type,
    name,
    memory_node_id,
    pages_kb,
    virtual_memory_committed_kb,
    awe_allocated_kb
FROM sys.dm_os_memory_clerks
WHERE pages_kb > 0
ORDER BY pages_kb DESC;
```

## Advanced Data Analysis / Phân tích dữ liệu nâng cao

### 31. Pivot và Unpivot Operations / Thao tác Pivot và Unpivot

**Câu hỏi / Question:** Làm thế nào để sử dụng PIVOT và UNPIVOT cho data analysis? / How to use PIVOT and UNPIVOT for data analysis?

```sql
-- PIVOT example / Ví dụ PIVOT
SELECT DepartmentName, [IT], [HR], [Finance], [Marketing]
FROM (
    SELECT
        d.DepartmentName,
        e.Salary,
        d.DepartmentName AS Dept
    FROM Employee e
    INNER JOIN Department d ON e.DepartmentID = d.DepartmentID
) AS SourceTable
PIVOT (
    AVG(Salary)
    FOR Dept IN ([IT], [HR], [Finance], [Marketing])
) AS PivotTable;

-- UNPIVOT example / Ví dụ UNPIVOT
WITH SalaryData AS (
    SELECT
        DepartmentName,
        AVG(Salary) AS AvgSalary,
        MIN(Salary) AS MinSalary,
        MAX(Salary) AS MaxSalary
    FROM Employee e
    INNER JOIN Department d ON e.DepartmentID = d.DepartmentID
    GROUP BY DepartmentName
)
SELECT DepartmentName, SalaryType, SalaryValue
FROM SalaryData
UNPIVOT (
    SalaryValue FOR SalaryType IN (AvgSalary, MinSalary, MaxSalary)
) AS UnpivotTable;
```

### 32. Advanced CTEs và Recursive Queries / CTEs nâng cao và truy vấn đệ quy

**Câu hỏi / Question:** Viết complex recursive CTEs? / Write complex recursive CTEs?

```sql
-- Employee hierarchy với salary analysis / Employee hierarchy with salary analysis
WITH EmployeeHierarchy AS (
    -- Anchor member / Thành viên neo
    SELECT
        EmployeeID,
        FirstName + ' ' + LastName AS FullName,
        ManagerID,
        Salary,
        DepartmentID,
        0 AS Level,
        CAST(FirstName + ' ' + LastName AS NVARCHAR(MAX)) AS HierarchyPath,
        Salary AS CumulativeSalary
    FROM Employee
    WHERE ManagerID IS NULL

    UNION ALL

    -- Recursive member / Thành viên đệ quy
    SELECT
        e.EmployeeID,
        e.FirstName + ' ' + e.LastName,
        e.ManagerID,
        e.Salary,
        e.DepartmentID,
        eh.Level + 1,
        eh.HierarchyPath + ' -> ' + e.FirstName + ' ' + e.LastName,
        eh.CumulativeSalary + e.Salary
    FROM Employee e
    INNER JOIN EmployeeHierarchy eh ON e.ManagerID = eh.EmployeeID
)
SELECT
    FullName,
    Level,
    HierarchyPath,
    Salary,
    CumulativeSalary,
    AVG(Salary) OVER (PARTITION BY Level) AS AvgSalaryAtLevel
FROM EmployeeHierarchy
ORDER BY HierarchyPath;
```

### 33. Advanced Subqueries và Correlated Queries / Truy vấn con nâng cao và correlated queries

**Câu hỏi / Question:** Viết complex correlated subqueries? / Write complex correlated subqueries?

```sql
-- Employees with salary higher than department average / Nhân viên có lương cao hơn trung bình phòng ban
SELECT
    e1.FirstName + ' ' + e1.LastName AS FullName,
    e1.Salary,
    d.DepartmentName,
    (SELECT AVG(e2.Salary)
     FROM Employee e2
     WHERE e2.DepartmentID = e1.DepartmentID) AS DeptAvgSalary,
    e1.Salary - (SELECT AVG(e2.Salary)
                 FROM Employee e2
                 WHERE e2.DepartmentID = e1.DepartmentID) AS SalaryDifference
FROM Employee e1
INNER JOIN Department d ON e1.DepartmentID = d.DepartmentID
WHERE e1.Salary > (SELECT AVG(e2.Salary)
                   FROM Employee e2
                   WHERE e2.DepartmentID = e1.DepartmentID);

-- Department với highest paid employee / Phòng ban với nhân viên lương cao nhất
SELECT
    d.DepartmentName,
    (SELECT e.FirstName + ' ' + e.LastName
     FROM Employee e
     WHERE e.DepartmentID = d.DepartmentID
       AND e.Salary = (SELECT MAX(Salary)
                       FROM Employee e2
                       WHERE e2.DepartmentID = d.DepartmentID)) AS HighestPaidEmployee,
    (SELECT MAX(Salary)
     FROM Employee e
     WHERE e.DepartmentID = d.DepartmentID) AS MaxSalary
FROM Department d
WHERE EXISTS (SELECT 1 FROM Employee e WHERE e.DepartmentID = d.DepartmentID);
```

## Advanced Error Handling và Logging / Xử lý lỗi và ghi log nâng cao

### 34. Comprehensive Error Handling / Xử lý lỗi toàn diện

**Câu hỏi / Question:** Implement comprehensive error handling strategy? / Implement comprehensive error handling strategy?

```sql
-- Error logging table / Bảng ghi log lỗi
CREATE TABLE ErrorLog (
    ErrorLogID INT IDENTITY(1,1) PRIMARY KEY,
    ErrorTime DATETIME2 DEFAULT GETDATE(),
    ErrorNumber INT,
    ErrorSeverity INT,
    ErrorState INT,
    ErrorProcedure NVARCHAR(128),
    ErrorLine INT,
    ErrorMessage NVARCHAR(4000),
    UserName NVARCHAR(128),
    HostName NVARCHAR(128),
    ApplicationName NVARCHAR(128)
);

-- Error handling procedure / Thủ tục xử lý lỗi
CREATE PROCEDURE LogError
    @ErrorMessage NVARCHAR(4000) = NULL
AS
BEGIN
    INSERT INTO ErrorLog (
        ErrorNumber, ErrorSeverity, ErrorState, ErrorProcedure,
        ErrorLine, ErrorMessage, UserName, HostName, ApplicationName
    )
    VALUES (
        ERROR_NUMBER(), ERROR_SEVERITY(), ERROR_STATE(), ERROR_PROCEDURE(),
        ERROR_LINE(), ISNULL(@ErrorMessage, ERROR_MESSAGE()),
        SYSTEM_USER, HOST_NAME(), APP_NAME()
    );
END;

-- Comprehensive error handling example / Ví dụ xử lý lỗi toàn diện
CREATE PROCEDURE SafeEmployeeUpdate
    @EmployeeID INT,
    @NewSalary DECIMAL(10,2),
    @NewDepartmentID INT
AS
BEGIN
    SET NOCOUNT ON;

    DECLARE @ErrorMessage NVARCHAR(4000);
    DECLARE @ErrorSeverity INT;
    DECLARE @ErrorState INT;
    DECLARE @OldDepartmentID INT;
    DECLARE @OldSalary DECIMAL(10,2);

    BEGIN TRY
        -- Validate inputs / Kiểm tra đầu vào
        IF @EmployeeID IS NULL
            THROW 50001, 'EmployeeID cannot be NULL', 1;

        IF @NewSalary <= 0
            THROW 50002, 'Salary must be greater than 0', 1;

        -- Check if employee exists / Kiểm tra nhân viên tồn tại
        IF NOT EXISTS (SELECT 1 FROM Employee WHERE EmployeeID = @EmployeeID)
            THROW 50003, 'Employee does not exist', 1;

        -- Get current values / Lấy giá trị hiện tại
        SELECT @OldSalary = Salary, @OldDepartmentID = DepartmentID
        FROM Employee WHERE EmployeeID = @EmployeeID;

        -- Begin transaction / Bắt đầu transaction
        BEGIN TRANSACTION;

        -- Update employee / Cập nhật nhân viên
        UPDATE Employee
        SET
            Salary = @NewSalary,
            DepartmentID = @NewDepartmentID,
            UpdatedAt = GETDATE()
        WHERE EmployeeID = @EmployeeID;

        -- Log the change / Ghi log thay đổi
        INSERT INTO EmployeeChangeLog (
            EmployeeID, OldSalary, NewSalary,
            OldDepartmentID, NewDepartmentID, ChangeDate
        )
        VALUES (
            @EmployeeID,
            @OldSalary,
            @NewSalary,
            @OldDepartmentID,
            @NewDepartmentID,
            GETDATE()
        );

        -- Update department budgets / Cập nhật ngân sách phòng ban
        UPDATE Department
        SET Budget = Budget - @OldSalary
        WHERE DepartmentID = @OldDepartmentID;

        UPDATE Department
        SET Budget = Budget + ISNULL(@NewSalary, @OldSalary)
        WHERE DepartmentID = @NewDepartmentID;

        COMMIT TRANSACTION;

        SELECT 'Employee updated successfully' AS Result;

    END TRY
    BEGIN CATCH
        IF @@TRANCOUNT > 0
            ROLLBACK TRANSACTION;

        SET @ErrorMessage = ERROR_MESSAGE();
        SET @ErrorSeverity = ERROR_SEVERITY();
        SET @ErrorState = ERROR_STATE();

        -- Log the error / Ghi log lỗi
        EXEC LogError @ErrorMessage;

        -- Re-throw the error / Ném lại lỗi
        THROW;
    END CATCH
END;
```

## 5. IDENTIFYING EXPENSIVE PARTS IN QUERIES / XÁC ĐỊNH CÁC PHẦN TỐN KÉM TRONG QUERY

### 5.1 EXECUTION PLAN ANALYSIS / PHÂN TÍCH EXECUTION PLAN

**Câu hỏi / Question:** Làm sao để xác định các phần tốn kém trong một query? / How to identify expensive parts in a query?

**Trả lời / Answer:** Sử dụng Execution Plan để phân tích chi tiết từng bước thực thi và xác định bottleneck.
**Answer:** Use Execution Plan to analyze each execution step in detail and identify bottlenecks.

#### 5.1.1 Cách xem Execution Plan / How to view Execution Plan

```sql
-- Hiển thị Execution Plan / Display Execution Plan
SET STATISTICS IO ON;
SET STATISTICS TIME ON;

-- Query cần phân tích / Query to analyze
SELECT
    c.CustomerName,
    c.Email,
    c.City,
    c.Country,
    COUNT(o.OrderID) as OrderCount,
    SUM(o.TotalAmount) as TotalSpent,
    AVG(o.TotalAmount) as AvgOrderValue,
    RANK() OVER (PARTITION BY c.Country ORDER BY SUM(o.TotalAmount) DESC) as CountryRank,
    -- Expensive subquery / Subquery tốn kém
    (SELECT COUNT(*) FROM Orders o2 WHERE o2.CustomerID = c.CustomerID AND o2.Status = 'Completed') as CompletedOrders,
    -- Another expensive subquery / Subquery tốn kém khác
    (SELECT MAX(TotalAmount) FROM Orders o3 WHERE o3.CustomerID = c.CustomerID) as LargestOrder
FROM Customers c
LEFT JOIN Orders o ON c.CustomerID = o.CustomerID
WHERE o.OrderDate >= '2024-01-01'
  AND c.Country IN ('USA', 'UK', 'France')
  AND EXISTS (SELECT 1 FROM Orders o4 WHERE o4.CustomerID = c.CustomerID AND o4.TotalAmount > 1000)
GROUP BY c.CustomerID, c.CustomerName, c.Email, c.City, c.Country
HAVING SUM(o.TotalAmount) > 5000
ORDER BY TotalSpent DESC;
```

#### 5.1.2 Các chỉ số quan trọng trong Execution Plan / Important metrics in Execution Plan

| Chỉ số / Metric    | Ý nghĩa / Meaning                                            | Cách đọc / How to read                                |
| ------------------ | ------------------------------------------------------------ | ----------------------------------------------------- |
| **Actual Rows**    | Số dòng thực tế được xử lý / Actual number of rows processed | Cao = Nhiều dữ liệu / High = Lots of data             |
| **Estimated Rows** | Số dòng ước tính / Estimated number of rows                  | So sánh với Actual Rows / Compare with Actual Rows    |
| **Cost %**         | Phần trăm chi phí của toán tử / Percentage cost of operator  | Cao = Tốn kém / High = Expensive                      |
| **CPU Cost**       | Chi phí CPU / CPU cost                                       | Cao = Tính toán phức tạp / High = Complex computation |
| **I/O Cost**       | Chi phí I/O / I/O cost                                       | Cao = Đọc/ghi nhiều / High = Lots of read/write       |

#### 5.1.3 Các toán tử tốn kém thường gặp / Common expensive operators

```sql
-- 1. TABLE SCAN - Quét toàn bộ bảng / Full table scan
-- Tốn kém khi: Bảng lớn, không có index / Expensive when: Large table, no index
SELECT * FROM LargeTable WHERE Column1 = 'Value';

-- 2. KEY LOOKUP - Tra cứu khóa / Key lookup
-- Tốn kém khi: Nhiều lần tra cứu / Expensive when: Many lookups
SELECT c.CustomerName, o.OrderID
FROM Customers c
INNER JOIN Orders o ON c.CustomerID = o.CustomerID;

-- 3. SORT - Sắp xếp / Sorting
-- Tốn kém khi: Nhiều dữ liệu cần sắp xếp / Expensive when: Lots of data to sort
SELECT * FROM Orders ORDER BY OrderDate DESC;

-- 4. HASH MATCH - Khớp hash / Hash matching
-- Tốn kém khi: Bộ nhớ không đủ / Expensive when: Insufficient memory
SELECT * FROM TableA a
INNER JOIN TableB b ON a.ID = b.ID;

-- 5. NESTED LOOPS - Vòng lặp lồng nhau / Nested loops
-- Tốn kém khi: Bảng bên trong lớn / Expensive when: Inner table is large
SELECT * FROM SmallTable s
INNER JOIN LargeTable l ON s.ID = l.ID;
```

### 5.2 PHÂN TÍCH CHI TIẾT CÁC PHẦN TỐN KÉM / DETAILED ANALYSIS OF EXPENSIVE PARTS

#### 5.2.1 Correlated Subqueries / Subquery tương quan

**Vấn đề / Problem:**

```sql
-- TỐN KÉM / EXPENSIVE
SELECT
    c.CustomerName,
    (SELECT COUNT(*) FROM Orders o WHERE o.CustomerID = c.CustomerID) as OrderCount,
    (SELECT MAX(TotalAmount) FROM Orders o WHERE o.CustomerID = c.CustomerID) as MaxOrder
FROM Customers c;
```

**Tại sao tốn kém / Why expensive:**

- Thực thi cho mỗi dòng của Customers / Executes for each row of Customers
- Không có index trên CustomerID / No index on CustomerID
- Lặp lại nhiều lần / Repeated execution

**Giải pháp / Solution:**

```sql
-- TỐI ƯU / OPTIMIZED
SELECT
    c.CustomerName,
    COUNT(o.OrderID) as OrderCount,
    MAX(o.TotalAmount) as MaxOrder
FROM Customers c
LEFT JOIN Orders o ON c.CustomerID = o.CustomerID
GROUP BY c.CustomerID, c.CustomerName;
```

#### 5.2.2 Window Functions với PARTITION BY lớn / Window Functions with large PARTITION BY

**Vấn đề / Problem:**

```sql
-- TỐN KÉM / EXPENSIVE
SELECT
    CustomerID,
    OrderID,
    TotalAmount,
    ROW_NUMBER() OVER (PARTITION BY CustomerID ORDER BY OrderDate) as RowNum,
    RANK() OVER (PARTITION BY CustomerID ORDER BY TotalAmount DESC) as AmountRank,
    LAG(TotalAmount) OVER (PARTITION BY CustomerID ORDER BY OrderDate) as PrevAmount
FROM Orders;
```

**Tại sao tốn kém / Why expensive:**

- Phân vùng theo CustomerID (có thể có nhiều giá trị) / Partition by CustomerID (can have many values)
- Nhiều window functions cùng lúc / Multiple window functions at once
- Sắp xếp phức tạp / Complex sorting

**Giải pháp / Solution:**

```sql
-- TỐI ƯU / OPTIMIZED
WITH RankedOrders AS (
    SELECT
        CustomerID,
        OrderID,
        TotalAmount,
        OrderDate,
        ROW_NUMBER() OVER (PARTITION BY CustomerID ORDER BY OrderDate) as RowNum,
        RANK() OVER (PARTITION BY CustomerID ORDER BY TotalAmount DESC) as AmountRank
    FROM Orders
)
SELECT
    CustomerID,
    OrderID,
    TotalAmount,
    RowNum,
    AmountRank,
    LAG(TotalAmount) OVER (PARTITION BY CustomerID ORDER BY OrderDate) as PrevAmount
FROM RankedOrders;
```

#### 5.2.3 EXISTS với subquery phức tạp / EXISTS with complex subquery

**Vấn đề / Problem:**

```sql
-- TỐN KÉM / EXPENSIVE
SELECT c.CustomerName
FROM Customers c
WHERE EXISTS (
    SELECT 1
    FROM Orders o
    INNER JOIN OrderDetails od ON o.OrderID = od.OrderID
    INNER JOIN Products p ON od.ProductID = p.ProductID
    WHERE o.CustomerID = c.CustomerID
      AND p.Category = 'Electronics'
      AND o.OrderDate >= '2024-01-01'
);
```

**Tại sao tốn kém / Why expensive:**

- JOIN nhiều bảng trong subquery / Multiple JOINs in subquery
- Thực thi cho mỗi Customer / Executes for each Customer
- Không có index tối ưu / No optimal indexes

**Giải pháp / Solution:**

```sql
-- TỐI ƯU / OPTIMIZED
SELECT DISTINCT c.CustomerName
FROM Customers c
INNER JOIN Orders o ON c.CustomerID = o.CustomerID
INNER JOIN OrderDetails od ON o.OrderID = od.OrderID
INNER JOIN Products p ON od.ProductID = p.ProductID
WHERE p.Category = 'Electronics'
  AND o.OrderDate >= '2024-01-01';
```

### 5.3 CÔNG CỤ PHÂN TÍCH HIỆU SUẤT / PERFORMANCE ANALYSIS TOOLS

#### 5.3.1 SET STATISTICS Commands / Lệnh SET STATISTICS

```sql
-- Bật thống kê I/O / Enable I/O statistics
SET STATISTICS IO ON;

-- Bật thống kê thời gian / Enable time statistics
SET STATISTICS TIME ON;

-- Bật thống kê profile / Enable profile statistics
SET STATISTICS PROFILE ON;

-- Query cần phân tích / Query to analyze
SELECT * FROM LargeTable WHERE Column1 = 'Value';

-- Tắt thống kê / Disable statistics
SET STATISTICS IO OFF;
SET STATISTICS TIME OFF;
SET STATISTICS PROFILE OFF;
```

#### 5.3.2 Dynamic Management Views (DMV) / View quản lý động

```sql
-- Xem các query đang chạy / View running queries
SELECT
    r.session_id,
    r.status,
    r.command,
    r.cpu_time,
    r.total_elapsed_time,
    r.reads,
    r.writes,
    r.logical_reads,
    st.text as sql_text
FROM sys.dm_exec_requests r
CROSS APPLY sys.dm_exec_sql_text(r.sql_handle) st
WHERE r.session_id > 50;

-- Xem cache plan / View plan cache
SELECT
    qs.execution_count,
    qs.total_elapsed_time,
    qs.total_logical_reads,
    qs.total_physical_reads,
    qs.total_worker_time,
    st.text as sql_text
FROM sys.dm_exec_query_stats qs
CROSS APPLY sys.dm_exec_sql_text(qs.sql_handle) st
ORDER BY qs.total_elapsed_time DESC;
```

#### 5.3.3 Query Store (SQL Server 2016+) / Query Store

```sql
-- Bật Query Store / Enable Query Store
ALTER DATABASE YourDatabase
SET QUERY_STORE = ON;

-- Xem thống kê query / View query statistics
SELECT
    qsq.query_id,
    qsq.query_hash,
    qsq.query_plan_hash,
    qsqt.query_sql_text,
    qsrs.avg_duration,
    qsrs.avg_logical_io_reads,
    qsrs.avg_physical_io_reads,
    qsrs.execution_count
FROM sys.query_store_query qsq
INNER JOIN sys.query_store_query_text qsqt ON qsq.query_text_id = qsqt.query_text_id
INNER JOIN sys.query_store_plan qsp ON qsq.query_id = qsp.query_id
INNER JOIN sys.query_store_runtime_stats qsrs ON qsp.plan_id = qsrs.plan_id
ORDER BY qsrs.avg_duration DESC;
```

### 5.4 CHIẾN LƯỢC TỐI ƯU HÓA / OPTIMIZATION STRATEGIES

#### 5.4.1 Index Optimization / Tối ưu hóa Index

```sql
-- Tạo index phù hợp / Create appropriate indexes
CREATE INDEX IX_Orders_CustomerID_OrderDate
ON Orders(CustomerID, OrderDate);

CREATE INDEX IX_Orders_Status_CustomerID
ON Orders(Status, CustomerID);

-- Index covering / Index bao phủ
CREATE INDEX IX_Orders_Covering
ON Orders(CustomerID, OrderDate, TotalAmount, Status)
INCLUDE (OrderID);
```

#### 5.4.2 Query Rewriting / Viết lại Query

```sql
-- Trước khi tối ưu / Before optimization
SELECT
    c.CustomerName,
    (SELECT COUNT(*) FROM Orders o WHERE o.CustomerID = c.CustomerID) as OrderCount
FROM Customers c
WHERE c.Country = 'USA';

-- Sau khi tối ưu / After optimization
SELECT
    c.CustomerName,
    COUNT(o.OrderID) as OrderCount
FROM Customers c
LEFT JOIN Orders o ON c.CustomerID = o.CustomerID
WHERE c.Country = 'USA'
GROUP BY c.CustomerID, c.CustomerName;
```

#### 5.4.3 Partitioning / Phân vùng

```sql
-- Tạo partitioned table / Create partitioned table
CREATE PARTITION FUNCTION PF_OrderDate (DATETIME)
AS RANGE RIGHT FOR VALUES (
    '2024-01-01', '2024-02-01', '2024-03-01', '2024-04-01'
);

CREATE PARTITION SCHEME PS_OrderDate
AS PARTITION PF_OrderDate
TO (FG1, FG2, FG3, FG4, FG5);

CREATE TABLE Orders_Partitioned (
    OrderID INT,
    CustomerID INT,
    OrderDate DATETIME,
    TotalAmount DECIMAL(10,2)
) ON PS_OrderDate(OrderDate);
```

### 5.5 CHECKLIST KIỂM TRA HIỆU SUẤT / PERFORMANCE CHECKLIST

#### 5.5.1 Trước khi tối ưu / Before optimization

- [ ] Có Execution Plan không? / Is there an Execution Plan?
- [ ] Có thống kê I/O và thời gian không? / Are there I/O and time statistics?
- [ ] Có index phù hợp không? / Are there appropriate indexes?
- [ ] Có subquery không cần thiết không? / Are there unnecessary subqueries?
- [ ] Có JOIN không hiệu quả không? / Are there inefficient JOINs?

#### 5.5.2 Sau khi tối ưu / After optimization

- [ ] Thời gian thực thi giảm? / Has execution time decreased?
- [ ] Số logical reads giảm? / Has number of logical reads decreased?
- [ ] CPU time giảm? / Has CPU time decreased?
- [ ] Memory usage giảm? / Has memory usage decreased?
- [ ] Query plan đơn giản hơn? / Is query plan simpler?

### 5.6 VÍ DỤ THỰC TẾ / PRACTICAL EXAMPLES

#### 5.6.1 Phân tích query phức tạp / Complex query analysis

```sql
-- QUERY GỐC / ORIGINAL QUERY
SELECT
    c.CustomerName,
    c.Email,
    COUNT(o.OrderID) as OrderCount,
    SUM(o.TotalAmount) as TotalSpent,
    -- Expensive: Correlated subquery
    (SELECT COUNT(*) FROM Orders o2 WHERE o2.CustomerID = c.CustomerID AND o2.Status = 'Completed') as CompletedOrders,
    -- Expensive: Another correlated subquery
    (SELECT MAX(TotalAmount) FROM Orders o3 WHERE o3.CustomerID = c.CustomerID) as LargestOrder,
    -- Expensive: Window function with large partition
    RANK() OVER (PARTITION BY c.Country ORDER BY SUM(o.TotalAmount) DESC) as CountryRank
FROM Customers c
LEFT JOIN Orders o ON c.CustomerID = o.CustomerID
WHERE c.Country IN ('USA', 'UK', 'France')
GROUP BY c.CustomerID, c.CustomerName, c.Email, c.Country;

-- QUERY TỐI ƯU / OPTIMIZED QUERY
WITH CustomerStats AS (
    SELECT
        CustomerID,
        COUNT(*) as OrderCount,
        SUM(TotalAmount) as TotalSpent,
        COUNT(CASE WHEN Status = 'Completed' THEN 1 END) as CompletedOrders,
        MAX(TotalAmount) as LargestOrder
    FROM Orders
    GROUP BY CustomerID
),
RankedCustomers AS (
    SELECT
        c.CustomerID,
        c.CustomerName,
        c.Email,
        c.Country,
        cs.OrderCount,
        cs.TotalSpent,
        cs.CompletedOrders,
        cs.LargestOrder,
        RANK() OVER (PARTITION BY c.Country ORDER BY cs.TotalSpent DESC) as CountryRank
    FROM Customers c
    INNER JOIN CustomerStats cs ON c.CustomerID = cs.CustomerID
    WHERE c.Country IN ('USA', 'UK', 'France')
)
SELECT * FROM RankedCustomers;
```

#### 5.6.2 So sánh hiệu suất / Performance comparison

```sql
-- Đo hiệu suất query gốc / Measure original query performance
SET STATISTICS IO ON;
SET STATISTICS TIME ON;

-- Query gốc / Original query
-- (Insert original query here)

-- Đo hiệu suất query tối ưu / Measure optimized query performance
SET STATISTICS IO ON;
SET STATISTICS TIME ON;

-- Query tối ưu / Optimized query
-- (Insert optimized query here)

-- So sánh kết quả / Compare results
-- - Logical reads: Giảm từ 1000 xuống 200
-- - CPU time: Giảm từ 500ms xuống 100ms
-- - Elapsed time: Giảm từ 800ms xuống 150ms
```

**Kết luận / Conclusion:**
Việc xác định các phần tốn kém trong query đòi hỏi hiểu biết sâu về Execution Plan, các chỉ số hiệu suất, và các kỹ thuật tối ưu hóa. Bằng cách sử dụng các công cụ phân tích và checklist có hệ thống, bạn có thể cải thiện đáng kể hiệu suất của các query phức tạp.

**Conclusion:** Identifying expensive parts in queries requires deep understanding of Execution Plans, performance metrics, and optimization techniques. By using analytical tools and systematic checklists, you can significantly improve the performance of complex queries.
