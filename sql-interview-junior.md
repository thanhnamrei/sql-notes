# SQL Interview Questions - Junior Level

# Câu hỏi phỏng vấn SQL - Cấp độ Junior

## Cơ bản về SQL / SQL Fundamentals

### 1. SQL là gì? / What is SQL?

**Trả lời / Answer:** SQL (Structured Query Language) là ngôn ngữ lập trình được sử dụng để quản lý và thao tác với cơ sở dữ liệu quan hệ.
**Answer:** SQL (Structured Query Language) is a programming language used to manage and manipulate relational databases.

### 2. Các loại câu lệnh SQL chính? / Main SQL Statement Types?

**Trả lời / Answer:**

- **DDL (Data Definition Language):** CREATE, ALTER, DROP, TRUNCATE
- **DML (Data Manipulation Language):** SELECT, INSERT, UPDATE, DELETE
- **DCL (Data Control Language):** GRANT, REVOKE
- **TCL (Transaction Control Language):** COMMIT, ROLLBACK, SAVEPOINT

### 3. Sự khác biệt giữa DELETE và TRUNCATE? / Difference between DELETE and TRUNCATE?

**Trả lời / Answer:**

- **DELETE:** Xóa từng dòng một, có thể ROLLBACK, chậm hơn
- **TRUNCATE:** Xóa toàn bộ bảng, không thể ROLLBACK, nhanh hơn

**Answer:**

- **DELETE:** Removes rows one by one, can be ROLLBACK, slower
- **TRUNCATE:** Removes all data from table, cannot ROLLBACK, faster

### 4. Các loại JOIN trong SQL? / Types of JOINs in SQL?

**Trả lời / Answer:**

- **INNER JOIN:** Chỉ lấy dữ liệu khớp giữa 2 bảng
- **LEFT JOIN:** Lấy tất cả dữ liệu từ bảng bên trái
- **RIGHT JOIN:** Lấy tất cả dữ liệu từ bảng bên phải
- **FULL JOIN:** Lấy tất cả dữ liệu từ cả 2 bảng

**Answer:**

- **INNER JOIN:** Returns only matching data between 2 tables
- **LEFT JOIN:** Returns all data from left table
- **RIGHT JOIN:** Returns all data from right table
- **FULL JOIN:** Returns all data from both tables

### 5. Sự khác biệt giữa WHERE và HAVING? / Difference between WHERE and HAVING?

**Trả lời / Answer:**

- **WHERE:** Lọc dữ liệu trước khi GROUP BY
- **HAVING:** Lọc dữ liệu sau khi GROUP BY

**Answer:**

- **WHERE:** Filters data before GROUP BY
- **HAVING:** Filters data after GROUP BY

### 6. Các loại dữ liệu cơ bản trong SQL Server? / Basic data types in SQL Server?

**Trả lời / Answer:**

- **INT:** Số nguyên / Integer
- **VARCHAR(n):** Chuỗi có độ dài thay đổi / Variable-length string
- **CHAR(n):** Chuỗi có độ dài cố định / Fixed-length string
- **DECIMAL(p,s):** Số thập phân / Decimal number
- **DATE:** Ngày tháng / Date
- **DATETIME:** Ngày giờ / Date and time
- **BIT:** Boolean (0/1) / Boolean value

## Thực hành cơ bản / Basic Practice

### 7. Viết câu lệnh tạo bảng Employee / Write CREATE TABLE statement for Employee

```sql
CREATE TABLE Employee (
    EmployeeID INT PRIMARY KEY,
    FirstName VARCHAR(50),
    LastName VARCHAR(50),
    Email VARCHAR(100),
    HireDate DATE,
    Salary DECIMAL(10,2)
);
```

### 8. Viết câu lệnh INSERT dữ liệu / Write INSERT statement

```sql
INSERT INTO Employee (EmployeeID, FirstName, LastName, Email, HireDate, Salary)
VALUES (1, 'John', 'Doe', 'john.doe@email.com', '2023-01-15', 50000.00);
```

### 9. Viết câu lệnh SELECT với điều kiện / Write SELECT with conditions

```sql
SELECT FirstName, LastName, Salary
FROM Employee
WHERE Salary > 45000
ORDER BY Salary DESC;
```

### 10. Viết câu lệnh UPDATE / Write UPDATE statement

```sql
UPDATE Employee
SET Salary = Salary * 1.1
WHERE EmployeeID = 1;
```

### 11. Viết câu lệnh DELETE / Write DELETE statement

```sql
DELETE FROM Employee
WHERE EmployeeID = 1;
```

### 12. Tạo bảng Department / Create Department table

```sql
CREATE TABLE Department (
    DepartmentID INT PRIMARY KEY,
    DepartmentName VARCHAR(100),
    Location VARCHAR(100),
    Budget DECIMAL(12,2)
);
```

### 13. Thêm khóa ngoại cho Employee / Add foreign key for Employee

```sql
ALTER TABLE Employee
ADD DepartmentID INT,
CONSTRAINT FK_Employee_Department
FOREIGN KEY (DepartmentID) REFERENCES Department(DepartmentID);
```

## Các hàm cơ bản / Basic Functions

### 14. Các hàm tổng hợp (Aggregate Functions)

**Trả lời / Answer:**

- COUNT(): Đếm số dòng / Count rows
- SUM(): Tổng / Sum
- AVG(): Trung bình / Average
- MAX(): Giá trị lớn nhất / Maximum value
- MIN(): Giá trị nhỏ nhất / Minimum value

### 15. Ví dụ sử dụng GROUP BY / Example using GROUP BY

```sql
SELECT DepartmentID, AVG(Salary) as AvgSalary
FROM Employee
GROUP BY DepartmentID
HAVING AVG(Salary) > 50000;
```

### 16. Các hàm xử lý chuỗi / String functions

**Trả lời / Answer:**

- **LEN():** Độ dài chuỗi / String length
- **LEFT():** Lấy ký tự bên trái / Get left characters
- **RIGHT():** Lấy ký tự bên phải / Get right characters
- **SUBSTRING():** Cắt chuỗi / Extract substring
- **UPPER():** Chuyển thành chữ hoa / Convert to uppercase
- **LOWER():** Chuyển thành chữ thường / Convert to lowercase
- **CONCAT():** Nối chuỗi / Concatenate strings

### 17. Các hàm xử lý ngày tháng / Date functions

**Trả lời / Answer:**

- **GETDATE():** Lấy ngày giờ hiện tại / Get current date and time
- **DATEADD():** Thêm/bớt ngày / Add/subtract date
- **DATEDIFF():** Tính khoảng cách giữa 2 ngày / Calculate difference between dates
- **YEAR():** Lấy năm / Extract year
- **MONTH():** Lấy tháng / Extract month
- **DAY():** Lấy ngày / Extract day

### 18. Ví dụ sử dụng hàm ngày tháng / Example using date functions

```sql
SELECT
    FirstName,
    LastName,
    HireDate,
    DATEDIFF(YEAR, HireDate, GETDATE()) as YearsOfService
FROM Employee
WHERE DATEDIFF(YEAR, HireDate, GETDATE()) > 5;
```

## Index và Performance / Index and Performance

### 19. Index là gì? Tại sao cần sử dụng? / What is Index? Why use it?

**Trả lời / Answer:** Index là cấu trúc dữ liệu giúp tăng tốc độ truy vấn dữ liệu. Nó giống như mục lục của sách, giúp tìm kiếm nhanh hơn.
**Answer:** Index is a data structure that helps speed up data retrieval. It's like a book index, helping search faster.

### 20. Các loại Index cơ bản / Basic Index Types

**Trả lời / Answer:**

- **Clustered Index:** Sắp xếp dữ liệu vật lý theo thứ tự
- **Non-Clustered Index:** Tạo bản sao dữ liệu để tìm kiếm

**Answer:**

- **Clustered Index:** Physically sorts data in order
- **Non-Clustered Index:** Creates data copy for searching

### 21. Tạo Index / Create Index

```sql
-- Tạo clustered index / Create clustered index
CREATE CLUSTERED INDEX IX_Employee_EmployeeID
ON Employee(EmployeeID);

-- Tạo non-clustered index / Create non-clustered index
CREATE NONCLUSTERED INDEX IX_Employee_LastName
ON Employee(LastName);
```

### 22. Khi nào nên tạo Index? / When to create Index?

**Trả lời / Answer:**

- Cột thường xuyên được sử dụng trong WHERE / Columns frequently used in WHERE
- Cột được sử dụng trong JOIN / Columns used in JOIN
- Cột được sử dụng trong ORDER BY / Columns used in ORDER BY
- Cột có tính duy nhất cao / Columns with high uniqueness

## Constraints

### 23. Các loại Constraints / Types of Constraints

**Trả lời / Answer:**

- **PRIMARY KEY:** Khóa chính, không được NULL và phải unique
- **FOREIGN KEY:** Khóa ngoại, tham chiếu đến bảng khác
- **UNIQUE:** Đảm bảo giá trị duy nhất
- **NOT NULL:** Không được để trống
- **CHECK:** Kiểm tra điều kiện
- **DEFAULT:** Giá trị mặc định

**Answer:**

- **PRIMARY KEY:** Primary key, cannot be NULL and must be unique
- **FOREIGN KEY:** Foreign key, references another table
- **UNIQUE:** Ensures unique values
- **NOT NULL:** Cannot be empty
- **CHECK:** Validates conditions
- **DEFAULT:** Default value

### 24. Ví dụ tạo bảng với Constraints / Example creating table with constraints

```sql
CREATE TABLE Orders (
    OrderID INT PRIMARY KEY,
    CustomerID INT NOT NULL,
    OrderDate DATE DEFAULT GETDATE(),
    TotalAmount DECIMAL(10,2) CHECK (TotalAmount > 0),
    Status VARCHAR(20) DEFAULT 'Pending',
    FOREIGN KEY (CustomerID) REFERENCES Customers(CustomerID)
);
```

### 25. Thêm/sửa/xóa Constraints / Add/modify/delete constraints

```sql
-- Thêm constraint / Add constraint
ALTER TABLE Employee
ADD CONSTRAINT CK_Salary CHECK (Salary > 0);

-- Xóa constraint / Drop constraint
ALTER TABLE Employee
DROP CONSTRAINT CK_Salary;
```

## Stored Procedures cơ bản / Basic Stored Procedures

### 26. Stored Procedure là gì? / What is Stored Procedure?

**Trả lời / Answer:** Là tập hợp các câu lệnh SQL được lưu trữ trong database, có thể được gọi và thực thi nhiều lần.
**Answer:** A collection of SQL statements stored in database, can be called and executed multiple times.

### 27. Ví dụ Stored Procedure đơn giản / Simple Stored Procedure example

```sql
CREATE PROCEDURE GetEmployeeByID
    @EmployeeID INT
AS
BEGIN
    SELECT * FROM Employee WHERE EmployeeID = @EmployeeID
END
```

### 28. Stored Procedure với nhiều tham số / Stored procedure with multiple parameters

```sql
CREATE PROCEDURE GetEmployeesByDepartment
    @DepartmentID INT,
    @MinSalary DECIMAL(10,2) = 0
AS
BEGIN
    SELECT FirstName, LastName, Salary
    FROM Employee
    WHERE DepartmentID = @DepartmentID
    AND Salary >= @MinSalary
    ORDER BY Salary DESC
END
```

### 29. Gọi Stored Procedure / Execute stored procedure

```sql
-- Gọi với tham số / Execute with parameters
EXEC GetEmployeeByID @EmployeeID = 1;

-- Gọi với tham số mặc định / Execute with default parameters
EXEC GetEmployeesByDepartment @DepartmentID = 1;
```

## Views

### 30. View là gì? / What is View?

**Trả lời / Answer:** View là bảng ảo được tạo từ kết quả của một câu lệnh SELECT. Nó không lưu trữ dữ liệu thực tế.
**Answer:** View is a virtual table created from the result of a SELECT statement. It doesn't store actual data.

### 31. Ví dụ tạo View / Example creating View

```sql
CREATE VIEW EmployeeSummary AS
SELECT EmployeeID, FirstName + ' ' + LastName as FullName, Salary
FROM Employee
WHERE Salary > 40000;
```

### 32. View với JOIN / View with JOIN

```sql
CREATE VIEW EmployeeDepartment AS
SELECT
    e.EmployeeID,
    e.FirstName + ' ' + e.LastName as FullName,
    e.Salary,
    d.DepartmentName
FROM Employee e
INNER JOIN Department d ON e.DepartmentID = d.DepartmentID;
```

### 33. Sử dụng View / Using View

```sql
-- Truy vấn view như bảng thường / Query view like regular table
SELECT * FROM EmployeeSummary;

-- Truy vấn với điều kiện / Query with conditions
SELECT * FROM EmployeeDepartment
WHERE DepartmentName = 'IT';
```

## Transaction

### 34. Transaction là gì? / What is Transaction?

**Trả lời / Answer:** Transaction là một đơn vị công việc logic, đảm bảo tính toàn vẹn dữ liệu thông qua ACID properties.
**Answer:** Transaction is a logical unit of work, ensuring data integrity through ACID properties.

### 35. ACID Properties

**Trả lời / Answer:**

- **Atomicity:** Hoặc tất cả hoặc không gì cả
- **Consistency:** Dữ liệu phải nhất quán
- **Isolation:** Các transaction độc lập với nhau
- **Durability:** Dữ liệu được lưu trữ vĩnh viễn

**Answer:**

- **Atomicity:** All or nothing
- **Consistency:** Data must be consistent
- **Isolation:** Transactions are independent
- **Durability:** Data is permanently stored

### 36. Ví dụ Transaction / Transaction example

```sql
BEGIN TRANSACTION;
    UPDATE Employee SET Salary = Salary * 1.1 WHERE EmployeeID = 1;
    UPDATE Department SET Budget = Budget + 1000 WHERE DepartmentID = 1;

    IF @@ERROR = 0
        COMMIT TRANSACTION;
    ELSE
        ROLLBACK TRANSACTION;
```

### 37. Sự khác biệt giữa COMMIT và ROLLBACK? / Difference between COMMIT and ROLLBACK?

**Trả lời / Answer:**

- **COMMIT:** Lưu các thay đổi vĩnh viễn
- **ROLLBACK:** Hoàn tác các thay đổi

**Answer:**

- **COMMIT:** Saves changes permanently
- **ROLLBACK:** Undoes changes

## Subqueries / Truy vấn con

### 38. Subquery là gì? / What is Subquery?

**Trả lời / Answer:** Subquery là câu lệnh SELECT được lồng bên trong câu lệnh SQL khác.
**Answer:** Subquery is a SELECT statement nested inside another SQL statement.

### 39. Subquery trong WHERE / Subquery in WHERE

```sql
SELECT FirstName, LastName, Salary
FROM Employee
WHERE Salary > (SELECT AVG(Salary) FROM Employee);
```

### 40. Subquery trong FROM / Subquery in FROM

```sql
SELECT dept.DepartmentName, emp.AvgSalary
FROM (
    SELECT DepartmentID, AVG(Salary) as AvgSalary
    FROM Employee
    GROUP BY DepartmentID
) emp
INNER JOIN Department dept ON emp.DepartmentID = dept.DepartmentID;
```

### 41. EXISTS và NOT EXISTS / EXISTS and NOT EXISTS

```sql
-- Tìm nhân viên có phòng ban / Find employees with department
SELECT FirstName, LastName
FROM Employee e
WHERE EXISTS (
    SELECT 1 FROM Department d
    WHERE d.DepartmentID = e.DepartmentID
);

-- Tìm nhân viên không có phòng ban / Find employees without department
SELECT FirstName, LastName
FROM Employee e
WHERE NOT EXISTS (
    SELECT 1 FROM Department d
    WHERE d.DepartmentID = e.DepartmentID
);
```

## Case Statement / Câu lệnh CASE

### 42. Sử dụng CASE / Using CASE

```sql
SELECT
    FirstName,
    LastName,
    Salary,
    CASE
        WHEN Salary < 30000 THEN 'Low'
        WHEN Salary BETWEEN 30000 AND 60000 THEN 'Medium'
        ELSE 'High'
    END as SalaryLevel
FROM Employee;
```

### 43. CASE với Aggregate / CASE with Aggregate

```sql
SELECT
    DepartmentID,
    COUNT(CASE WHEN Salary < 30000 THEN 1 END) as LowSalaryCount,
    COUNT(CASE WHEN Salary BETWEEN 30000 AND 60000 THEN 1 END) as MediumSalaryCount,
    COUNT(CASE WHEN Salary > 60000 THEN 1 END) as HighSalaryCount
FROM Employee
GROUP BY DepartmentID;
```

## Câu hỏi thực hành / Practice Questions

### 44. Viết câu lệnh tìm nhân viên có lương cao nhất / Find employee with highest salary

```sql
SELECT TOP 1 FirstName, LastName, Salary
FROM Employee
ORDER BY Salary DESC;
```

### 45. Viết câu lệnh đếm số nhân viên theo phòng ban / Count employees by department

```sql
SELECT DepartmentID, COUNT(*) as EmployeeCount
FROM Employee
GROUP BY DepartmentID;
```

### 46. Viết câu lệnh tìm nhân viên có tên bắt đầu bằng 'J' / Find employees with name starting with 'J'

```sql
SELECT * FROM Employee
WHERE FirstName LIKE 'J%';
```

### 47. Viết câu lệnh JOIN 2 bảng Employee và Department / JOIN Employee and Department tables

```sql
SELECT e.FirstName, e.LastName, d.DepartmentName
FROM Employee e
INNER JOIN Department d ON e.DepartmentID = d.DepartmentID;
```

### 48. Tìm nhân viên có lương cao nhất trong mỗi phòng ban / Find highest paid employee in each department

```sql
SELECT e.FirstName, e.LastName, e.Salary, d.DepartmentName
FROM Employee e
INNER JOIN Department d ON e.DepartmentID = d.DepartmentID
WHERE e.Salary = (
    SELECT MAX(Salary)
    FROM Employee
    WHERE DepartmentID = e.DepartmentID
);
```

### 49. Tính tổng lương theo phòng ban / Calculate total salary by department

```sql
SELECT
    d.DepartmentName,
    COUNT(e.EmployeeID) as EmployeeCount,
    SUM(e.Salary) as TotalSalary,
    AVG(e.Salary) as AvgSalary
FROM Department d
LEFT JOIN Employee e ON d.DepartmentID = e.DepartmentID
GROUP BY d.DepartmentID, d.DepartmentName;
```

### 50. Tìm nhân viên làm việc lâu nhất / Find longest serving employee

```sql
SELECT TOP 1
    FirstName,
    LastName,
    HireDate,
    DATEDIFF(YEAR, HireDate, GETDATE()) as YearsOfService
FROM Employee
ORDER BY HireDate ASC;
```

## Common SQL Functions / Các hàm SQL thường dùng

### 51. ISNULL và COALESCE / ISNULL and COALESCE

```sql
-- ISNULL: Thay thế NULL bằng giá trị mặc định / Replace NULL with default value
SELECT FirstName, ISNULL(ManagerID, 0) as ManagerID
FROM Employee;

-- COALESCE: Trả về giá trị đầu tiên không NULL / Return first non-NULL value
SELECT FirstName, COALESCE(ManagerID, DepartmentID, 0) as ID
FROM Employee;
```

### 52. NULLIF / NULLIF

```sql
-- NULLIF: Trả về NULL nếu 2 giá trị bằng nhau / Return NULL if two values are equal
SELECT FirstName, NULLIF(Salary, 0) as Salary
FROM Employee;
```

### 53. CAST và CONVERT / CAST and CONVERT

```sql
-- CAST: Chuyển đổi kiểu dữ liệu / Convert data type
SELECT CAST(Salary AS VARCHAR(10)) + ' USD' as SalaryText
FROM Employee;

-- CONVERT: Chuyển đổi kiểu dữ liệu với format / Convert data type with format
SELECT CONVERT(VARCHAR(10), HireDate, 101) as HireDateText
FROM Employee;
```

## Error Handling / Xử lý lỗi

### 54. TRY-CATCH trong SQL / TRY-CATCH in SQL

```sql
BEGIN TRY
    INSERT INTO Employee (EmployeeID, FirstName, LastName)
    VALUES (1, 'John', 'Doe');
END TRY
BEGIN CATCH
    SELECT
        ERROR_MESSAGE() as ErrorMessage,
        ERROR_LINE() as ErrorLine;
END CATCH
```

### 55. @@ERROR và @@ROWCOUNT / @@ERROR and @@ROWCOUNT

```sql
UPDATE Employee SET Salary = Salary * 1.1 WHERE EmployeeID = 999;

IF @@ERROR <> 0
    PRINT 'Error occurred: ' + CAST(@@ERROR AS VARCHAR(10));
ELSE
    PRINT 'Rows affected: ' + CAST(@@ROWCOUNT AS VARCHAR(10));
```

## SQL Debugging Techniques / Kỹ thuật debug SQL

### 56. Các kỹ thuật debug cơ bản / Basic debugging techniques

**Trả lời / Answer:** Debug SQL là quá trình tìm và sửa lỗi trong câu lệnh SQL. Có nhiều kỹ thuật khác nhau để debug hiệu quả.
**Answer:** SQL debugging is the process of finding and fixing errors in SQL statements. There are various techniques for effective debugging.

### 57. Sử dụng PRINT để debug / Using PRINT for debugging

```sql
-- Thêm PRINT statements để theo dõi quá trình thực thi
-- Add PRINT statements to track execution process
DECLARE @DebugValue INT = 1;
PRINT 'Debug: Starting query execution';

SELECT @DebugValue = COUNT(*) FROM Employee;
PRINT 'Debug: Found ' + CAST(@DebugValue AS VARCHAR(10)) + ' employees';

-- Tiếp tục với logic của bạn
-- Continue with your logic
```

### 58. Debug từng bước với CTE / Step-by-step debugging with CTE

```sql
-- Bước 1: Kiểm tra dữ liệu đầu vào
-- Step 1: Check input data
SELECT * FROM @Dates ORDER BY DateValue;

-- Bước 2: Test CTE đầu tiên
-- Step 2: Test first CTE
WITH Numbered AS (
    SELECT
        DateValue,
        ROW_NUMBER() OVER (ORDER BY DateValue) AS rn
    FROM @Dates
)
SELECT * FROM Numbered ORDER BY DateValue;

-- Bước 3: Test CTE thứ hai
-- Step 3: Test second CTE
WITH Numbered AS (
    SELECT
        DateValue,
        ROW_NUMBER() OVER (ORDER BY DateValue) AS rn
    FROM @Dates
),
Grouped AS (
    SELECT
        DateValue,
        DATEADD(day, -rn, DateValue) AS grp
    FROM Numbered
)
SELECT * FROM Grouped ORDER BY DateValue;
```

### 59. Sử dụng STATISTICS để debug performance / Using STATISTICS for performance debugging

```sql
-- Bật thống kê để phân tích hiệu suất query
-- Enable statistics to analyze query performance
SET STATISTICS IO ON;
SET STATISTICS TIME ON;

-- Query của bạn ở đây
-- Your query here
SELECT e.FirstName, e.LastName, d.DepartmentName
FROM Employee e
INNER JOIN Department d ON e.DepartmentID = d.DepartmentID
WHERE e.Salary > 50000;

SET STATISTICS IO OFF;
SET STATISTICS TIME OFF;
```

### 60. Debug với Execution Plan / Debugging with Execution Plan

```sql
-- Lấy execution plan chi tiết
-- Get detailed execution plan
SET SHOWPLAN_ALL ON;
GO
-- Query của bạn ở đây
-- Your query here
SELECT * FROM Employee WHERE Salary > 50000;
GO
SET SHOWPLAN_ALL OFF;
```

### 61. Sử dụng temp tables để debug / Using temp tables for debugging

```sql
-- Sử dụng temp tables để lưu kết quả trung gian
-- Use temp tables to store intermediate results
SELECT
    DateValue,
    ROW_NUMBER() OVER (ORDER BY DateValue) AS rn
INTO #TempNumbered
FROM @Dates;

-- Kiểm tra kết quả trung gian
-- Check intermediate results
SELECT * FROM #TempNumbered;

-- Tiếp tục với bước tiếp theo
-- Continue with next step
SELECT
    DateValue,
    DATEADD(day, -rn, DateValue) AS grp
INTO #TempGrouped
FROM #TempNumbered;

-- Dọn dẹp
-- Clean up
DROP TABLE #TempNumbered;
DROP TABLE #TempGrouped;
```

### 62. Debug stored procedures / Debugging stored procedures

```sql
-- Thêm debugging vào stored procedures
-- Add debugging to stored procedures
CREATE PROCEDURE DebugExample
    @Parameter1 INT
AS
BEGIN
    SET NOCOUNT ON;

    PRINT 'Debug: Starting procedure with parameter: ' + CAST(@Parameter1 AS VARCHAR(10));

    -- Logic của bạn ở đây
    -- Your logic here
    DECLARE @Result INT;
    SELECT @Result = COUNT(*) FROM Employee WHERE DepartmentID = @Parameter1;

    PRINT 'Debug: Found ' + CAST(@Result AS VARCHAR(10)) + ' employees';

    -- Tiếp tục với logic của bạn
    -- Continue with your logic
END
```

### 63. Debug với TRY-CATCH nâng cao / Advanced debugging with TRY-CATCH

```sql
BEGIN TRY
    -- Query của bạn ở đây
    -- Your query here
    INSERT INTO Employee (EmployeeID, FirstName, LastName)
    VALUES (1, 'John', 'Doe');

    PRINT 'Operation completed successfully';
END TRY
BEGIN CATCH
    SELECT
        ERROR_NUMBER() AS ErrorNumber,
        ERROR_MESSAGE() AS ErrorMessage,
        ERROR_LINE() AS ErrorLine,
        ERROR_PROCEDURE() AS ErrorProcedure;
END CATCH
```

### 64. Debug data validation / Debugging data validation

```sql
-- Kiểm tra kiểu dữ liệu
-- Check data types
SELECT
    COLUMN_NAME,
    DATA_TYPE,
    IS_NULLABLE,
    CHARACTER_MAXIMUM_LENGTH
FROM INFORMATION_SCHEMA.COLUMNS
WHERE TABLE_SCHEMA = 'catalog'
AND TABLE_NAME = 'Products';

-- Kiểm tra vi phạm ràng buộc
-- Check constraint violations
SELECT * FROM catalog.Products
WHERE CategoryId NOT IN (SELECT Id FROM catalog.Categories);

-- Tìm dữ liệu orphaned
-- Find orphaned data
SELECT p.*
FROM catalog.Products p
LEFT JOIN catalog.Categories c ON p.CategoryId = c.Id
WHERE c.Id IS NULL;
```

### 65. Debug performance issues / Debugging performance issues

```sql
-- Kiểm tra blocking
-- Check for blocking
SELECT
    s.session_id,
    s.login_name,
    r.command,
    r.status,
    r.wait_type,
    r.blocking_session_id,
    t.text AS sql_text
FROM sys.dm_exec_sessions s
INNER JOIN sys.dm_exec_requests r ON s.session_id = r.session_id
CROSS APPLY sys.dm_exec_sql_text(r.sql_handle) t
WHERE r.blocking_session_id > 0;

-- Kiểm tra index usage
-- Check index usage
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
```

### 66. Best practices cho SQL debugging / Best practices for SQL debugging

**Trả lời / Answer:**

1. **Bắt đầu nhỏ:** Test từng component riêng lẻ trước khi kết hợp
2. **Sử dụng comments:** Ghi chép các bước debug
3. **Kiểm tra kiểu dữ liệu:** Đảm bảo chuyển đổi kiểu dữ liệu đúng
4. **Validate assumptions:** Luôn kiểm tra giả định về dữ liệu
5. **Sử dụng tên có ý nghĩa:** Đặt tên biến và bảng mô tả rõ ràng
6. **Test với sample data:** Sử dụng dữ liệu test đại diện
7. **Monitor performance:** Luôn kiểm tra hiệu suất query
8. **Document changes:** Theo dõi những gì đã thay đổi

**Answer:**

1. **Start Small:** Test individual components before combining
2. **Use Comments:** Document your debugging steps
3. **Check Data Types:** Ensure proper data type conversions
4. **Validate Assumptions:** Always verify your data assumptions
5. **Use Meaningful Names:** Use descriptive variable and table names
6. **Test with Sample Data:** Use representative test data
7. **Monitor Performance:** Always check query performance
8. **Document Changes:** Keep track of what you've changed

### 67. Common debugging scenarios / Các tình huống debug thường gặp

```sql
-- Debug JOIN issues
-- Debug vấn đề JOIN
SELECT e.*, d.DepartmentName
FROM Employee e
LEFT JOIN Department d ON e.DepartmentID = d.DepartmentID
WHERE d.DepartmentID IS NULL; -- Tìm records không có department

-- Debug GROUP BY issues
-- Debug vấn đề GROUP BY
SELECT
    DepartmentID,
    COUNT(*) as EmployeeCount,
    AVG(Salary) as AvgSalary
FROM Employee
GROUP BY DepartmentID
HAVING COUNT(*) > 1; -- Chỉ xem departments có nhiều hơn 1 employee

-- Debug subquery issues
-- Debug vấn đề subquery
SELECT FirstName, LastName, Salary
FROM Employee e1
WHERE Salary > (
    SELECT AVG(Salary)
    FROM Employee e2
    WHERE e2.DepartmentID = e1.DepartmentID
);
```

## Wildcards và Pattern Matching

### 68. Sử dụng LIKE với Wildcards / Using LIKE with Wildcards

**Trả lời / Answer:**

- **%:** Đại diện cho 0 hoặc nhiều ký tự / Represents zero or more characters
- **_:** Đại diện cho 1 ký tự / Represents one character
- **[]:** Đại diện cho 1 ký tự trong tập / Represents one character in a set
- **[^]:** Đại diện cho 1 ký tự không có trong tập / Represents one character NOT in a set

```sql
-- Tên bắt đầu bằng 'J' / Name starts with 'J'
SELECT * FROM Employee WHERE FirstName LIKE 'J%';

-- Tên kết thúc bằng 'n' / Name ends with 'n'
SELECT * FROM Employee WHERE FirstName LIKE '%n';

-- Tên có chữ 'oh' ở giữa / Name contains 'oh'
SELECT * FROM Employee WHERE FirstName LIKE '%oh%';

-- Tên có đúng 4 ký tự / Name has exactly 4 characters
SELECT * FROM Employee WHERE FirstName LIKE '____';

-- Tên bắt đầu bằng J, A, hoặc M / Name starts with J, A, or M
SELECT * FROM Employee WHERE FirstName LIKE '[JAM]%';

-- Tên không bắt đầu bằng J / Name does not start with J
SELECT * FROM Employee WHERE FirstName LIKE '[^J]%';
```

## UNION và SET Operations

### 69. UNION và UNION ALL / UNION and UNION ALL

**Trả lời / Answer:**

- **UNION:** Kết hợp kết quả từ 2 query, loại bỏ duplicate / Combines results from 2 queries, removes duplicates
- **UNION ALL:** Kết hợp kết quả từ 2 query, giữ duplicate / Combines results from 2 queries, keeps duplicates

```sql
-- UNION: Loại bỏ duplicate
-- UNION: Removes duplicates
SELECT FirstName FROM Employee
UNION
SELECT FirstName FROM Manager;

-- UNION ALL: Giữ duplicate
-- UNION ALL: Keeps duplicates
SELECT FirstName FROM Employee
UNION ALL
SELECT FirstName FROM Manager;
```

### 70. INTERSECT và EXCEPT / INTERSECT and EXCEPT

```sql
-- INTERSECT: Lấy các giá trị chung giữa 2 query
-- INTERSECT: Gets common values between 2 queries
SELECT FirstName FROM Employee
INTERSECT
SELECT FirstName FROM Manager;

-- EXCEPT: Lấy các giá trị có trong query đầu nhưng không có trong query sau
-- EXCEPT: Gets values in first query but not in second
SELECT FirstName FROM Employee
EXCEPT
SELECT FirstName FROM Manager;
```

## Normalization / Chuẩn hóa dữ liệu

### 71. Database Normalization là gì? / What is Database Normalization?

**Trả lời / Answer:** Normalization là quá trình tổ chức dữ liệu trong database để giảm redundancy và cải thiện data integrity.

**Answer:** Normalization is the process of organizing data in a database to reduce redundancy and improve data integrity.

### 72. Các Normal Forms cơ bản / Basic Normal Forms

**Trả lời / Answer:**

- **1NF (First Normal Form):** Mỗi cell chỉ chứa 1 giá trị atomic / Each cell contains only one atomic value
- **2NF (Second Normal Form):** Đạt 1NF + không có partial dependency / Meets 1NF + no partial dependency
- **3NF (Third Normal Form):** Đạt 2NF + không có transitive dependency / Meets 2NF + no transitive dependency

**Example:**

```sql
-- Không chuẩn hóa (Not normalized)
CREATE TABLE Orders_Bad (
    OrderID INT,
    CustomerName VARCHAR(100),
    CustomerPhone VARCHAR(20),
    Product1 VARCHAR(100),
    Product2 VARCHAR(100)
);

-- Chuẩn hóa (Normalized)
CREATE TABLE Customers (
    CustomerID INT PRIMARY KEY,
    CustomerName VARCHAR(100),
    CustomerPhone VARCHAR(20)
);

CREATE TABLE Orders (
    OrderID INT PRIMARY KEY,
    CustomerID INT,
    FOREIGN KEY (CustomerID) REFERENCES Customers(CustomerID)
);

CREATE TABLE OrderDetails (
    OrderID INT,
    ProductID INT,
    Quantity INT,
    PRIMARY KEY (OrderID, ProductID)
);
```

## Window Functions cơ bản / Basic Window Functions

### 73. Window Functions là gì? / What are Window Functions?

**Trả lời / Answer:** Window functions thực hiện tính toán trên một tập hợp rows liên quan đến current row, không làm thay đổi số lượng rows.

**Answer:** Window functions perform calculations on a set of rows related to the current row, without changing the number of rows.

### 74. ROW_NUMBER, RANK, DENSE_RANK

```sql
SELECT
    FirstName,
    LastName,
    Salary,
    DepartmentID,
    -- ROW_NUMBER: Đánh số duy nhất cho mỗi row
    -- ROW_NUMBER: Assigns unique number to each row
    ROW_NUMBER() OVER (PARTITION BY DepartmentID ORDER BY Salary DESC) as RowNum,
    
    -- RANK: Đánh số có gaps khi có ties
    -- RANK: Assigns numbers with gaps when ties exist
    RANK() OVER (PARTITION BY DepartmentID ORDER BY Salary DESC) as Rank,
    
    -- DENSE_RANK: Đánh số không có gaps
    -- DENSE_RANK: Assigns numbers without gaps
    DENSE_RANK() OVER (PARTITION BY DepartmentID ORDER BY Salary DESC) as DenseRank
FROM Employee;
```

### 75. LAG và LEAD / LAG and LEAD

```sql
-- LAG: Truy cập giá trị của row trước đó
-- LAG: Access value from previous row
-- LEAD: Truy cập giá trị của row tiếp theo
-- LEAD: Access value from next row

SELECT
    EmployeeID,
    FirstName,
    Salary,
    LAG(Salary, 1, 0) OVER (ORDER BY EmployeeID) as PreviousSalary,
    LEAD(Salary, 1, 0) OVER (ORDER BY EmployeeID) as NextSalary,
    Salary - LAG(Salary, 1, 0) OVER (ORDER BY EmployeeID) as SalaryDiff
FROM Employee;
```

## Data Types nâng cao / Advanced Data Types

### 76. Các kiểu dữ liệu số / Numeric Data Types

```sql
-- INT, BIGINT, SMALLINT, TINYINT
-- DECIMAL(p,s), NUMERIC(p,s): Số thập phân chính xác
-- FLOAT, REAL: Số thập phân gần đúng
-- MONEY, SMALLMONEY: Tiền tệ

CREATE TABLE ProductPricing (
    ProductID INT,
    Price DECIMAL(10,2),      -- 10 chữ số, 2 số thập phân
    Tax FLOAT,                 -- Số thập phân gần đúng
    TotalCost MONEY            -- Lưu trữ tiền tệ
);
```

### 77. Các kiểu dữ liệu chuỗi / String Data Types

```sql
-- CHAR(n): Chuỗi độ dài cố định, thêm spaces nếu thiếu
-- VARCHAR(n): Chuỗi độ dài thay đổi, không thêm spaces
-- NVARCHAR(n): Unicode, hỗ trợ đa ngôn ngữ
-- TEXT: Chuỗi lớn (deprecated, nên dùng VARCHAR(MAX))

CREATE TABLE Documents (
    Title VARCHAR(100),        -- Tiêu đề (ASCII)
    TitleUnicode NVARCHAR(100),-- Tiêu đề (Unicode) 
    Content VARCHAR(MAX)       -- Nội dung lớn
);
```

### 78. Các kiểu dữ liệu ngày tháng / Date and Time Data Types

```sql
-- DATE: Chỉ ngày (YYYY-MM-DD)
-- TIME: Chỉ giờ (HH:MM:SS)
-- DATETIME: Ngày và giờ
-- DATETIME2: Datetime chính xác hơn
-- DATETIMEOFFSET: Datetime với timezone
-- SMALLDATETIME: Datetime ít chính xác hơn

CREATE TABLE Events (
    EventID INT,
    EventDate DATE,
    EventTime TIME,
    CreatedAt DATETIME2,
    ScheduledAt DATETIMEOFFSET
);
```

## Temporary Tables và Table Variables

### 79. Local Temporary Tables (#)

```sql
-- Temporary table chỉ tồn tại trong session hiện tại
-- Temporary table exists only in current session

CREATE TABLE #TempEmployee (
    EmployeeID INT,
    FirstName VARCHAR(50),
    Salary DECIMAL(10,2)
);

INSERT INTO #TempEmployee
SELECT EmployeeID, FirstName, Salary
FROM Employee
WHERE Salary > 50000;

SELECT * FROM #TempEmployee;

-- Tự động xóa khi session kết thúc
-- Automatically dropped when session ends
DROP TABLE #TempEmployee;
```

### 80. Global Temporary Tables (##)

```sql
-- Temporary table có thể truy cập từ mọi session
-- Temporary table accessible from all sessions

CREATE TABLE ##GlobalTempEmployee (
    EmployeeID INT,
    FirstName VARCHAR(50)
);

-- Có thể truy cập từ session khác
-- Can be accessed from other sessions
-- Tự động xóa khi tất cả sessions kết thúc
-- Automatically dropped when all sessions end
```

### 81. Table Variables (@)

```sql
-- Table variable: nhanh hơn temp table cho dataset nhỏ
-- Table variable: faster than temp table for small datasets

DECLARE @EmployeeTable TABLE (
    EmployeeID INT,
    FirstName VARCHAR(50),
    Salary DECIMAL(10,2)
);

INSERT INTO @EmployeeTable
SELECT EmployeeID, FirstName, Salary
FROM Employee
WHERE Salary > 50000;

SELECT * FROM @EmployeeTable;

-- Tự động xóa sau khi batch kết thúc
-- Automatically cleared after batch ends
```

## Common Table Expressions (CTE) nâng cao

### 82. Recursive CTE

```sql
-- Tìm hierarchy của nhân viên
-- Find employee hierarchy

WITH EmployeeHierarchy AS (
    -- Anchor member: Tìm manager cao nhất
    -- Anchor member: Find top manager
    SELECT EmployeeID, FirstName, ManagerID, 1 as Level
    FROM Employee
    WHERE ManagerID IS NULL
    
    UNION ALL
    
    -- Recursive member: Tìm employees thuộc managers
    -- Recursive member: Find employees under managers
    SELECT e.EmployeeID, e.FirstName, e.ManagerID, eh.Level + 1
    FROM Employee e
    INNER JOIN EmployeeHierarchy eh ON e.ManagerID = eh.EmployeeID
)
SELECT * FROM EmployeeHierarchy
ORDER BY Level, EmployeeID;
```

### 83. Multiple CTEs

```sql
-- Sử dụng nhiều CTE trong một query
-- Use multiple CTEs in one query

WITH HighEarners AS (
    SELECT EmployeeID, FirstName, Salary, DepartmentID
    FROM Employee
    WHERE Salary > 60000
),
DeptStats AS (
    SELECT DepartmentID, AVG(Salary) as AvgSalary
    FROM Employee
    GROUP BY DepartmentID
)
SELECT 
    h.FirstName,
    h.Salary,
    d.AvgSalary,
    h.Salary - d.AvgSalary as SalaryDiff
FROM HighEarners h
INNER JOIN DeptStats d ON h.DepartmentID = d.DepartmentID;
```

## Pivot và Unpivot

### 84. PIVOT - Chuyển rows thành columns

```sql
-- Chuyển dữ liệu từ rows sang columns
-- Convert data from rows to columns

SELECT *
FROM (
    SELECT DepartmentName, YEAR(HireDate) as Year
    FROM Employee e
    INNER JOIN Department d ON e.DepartmentID = d.DepartmentID
) AS SourceTable
PIVOT (
    COUNT(Year)
    FOR Year IN ([2020], [2021], [2022], [2023])
) AS PivotTable;
```

### 85. UNPIVOT - Chuyển columns thành rows

```sql
-- Chuyển dữ liệu từ columns sang rows
-- Convert data from columns to rows

SELECT Department, Year, EmployeeCount
FROM (
    SELECT DepartmentName, [2020], [2021], [2022], [2023]
    FROM DepartmentYearlyCounts
) AS SourceTable
UNPIVOT (
    EmployeeCount FOR Year IN ([2020], [2021], [2022], [2023])
) AS UnpivotTable;
```

## Data Modification với OUTPUT

### 86. OUTPUT Clause

```sql
-- Xem dữ liệu bị ảnh hưởng bởi INSERT/UPDATE/DELETE
-- View data affected by INSERT/UPDATE/DELETE

-- INSERT với OUTPUT
INSERT INTO Employee (EmployeeID, FirstName, LastName, Salary)
OUTPUT INSERTED.EmployeeID, INSERTED.FirstName, INSERTED.Salary
VALUES (100, 'John', 'Doe', 50000);

-- UPDATE với OUTPUT
UPDATE Employee
SET Salary = Salary * 1.1
OUTPUT DELETED.Salary as OldSalary, INSERTED.Salary as NewSalary
WHERE EmployeeID = 1;

-- DELETE với OUTPUT
DELETE FROM Employee
OUTPUT DELETED.*
WHERE EmployeeID = 100;
```

## Best Practices cho Junior Developers

### 87. Coding Standards và Best Practices

**Trả lời / Answer:**

**1. Đặt tên có ý nghĩa / Use Meaningful Names:**
```sql
-- Tốt / Good
SELECT EmployeeID, FirstName, LastName FROM Employee;

-- Không tốt / Bad
SELECT e1, e2, e3 FROM tbl1;
```

**2. Sử dụng JOIN thay vì WHERE cho nhiều bảng / Use JOIN instead of WHERE for multiple tables:**
```sql
-- Tốt / Good
SELECT e.FirstName, d.DepartmentName
FROM Employee e
INNER JOIN Department d ON e.DepartmentID = d.DepartmentID;

-- Không tốt / Bad (implicit join)
SELECT e.FirstName, d.DepartmentName
FROM Employee e, Department d
WHERE e.DepartmentID = d.DepartmentID;
```

**3. Luôn sử dụng schema name:**
```sql
-- Tốt / Good
SELECT * FROM dbo.Employee;

-- Không tốt / Bad
SELECT * FROM Employee;
```

**4. Tránh SELECT * trong production code:**
```sql
-- Tốt / Good
SELECT EmployeeID, FirstName, LastName FROM Employee;

-- Không tốt / Bad
SELECT * FROM Employee;
```

**5. Sử dụng EXISTS thay vì IN cho subqueries lớn:**
```sql
-- Tốt / Good (faster)
SELECT * FROM Employee e
WHERE EXISTS (
    SELECT 1 FROM Department d 
    WHERE d.DepartmentID = e.DepartmentID
);

-- Có thể chậm hơn / Can be slower
SELECT * FROM Employee
WHERE DepartmentID IN (SELECT DepartmentID FROM Department);
```

### 88. Security Best Practices

```sql
-- 1. Luôn sử dụng parameterized queries để tránh SQL injection
-- Always use parameterized queries to prevent SQL injection

-- 2. Grant permissions tối thiểu cần thiết
-- Grant minimum necessary permissions
GRANT SELECT, INSERT ON Employee TO AppUser;

-- 3. Không lưu passwords dạng plain text
-- Never store passwords as plain text
-- Sử dụng HASHBYTES
CREATE TABLE Users (
    UserID INT,
    Username VARCHAR(50),
    PasswordHash VARBINARY(64)
);

INSERT INTO Users (UserID, Username, PasswordHash)
VALUES (1, 'john', HASHBYTES('SHA2_256', 'mypassword'));
```

## Tips cho phỏng vấn / Interview Tips

1. **Hiểu rõ cú pháp cơ bản / Understand basic syntax**
2. **Thực hành viết câu lệnh thường xuyên / Practice writing statements regularly**
3. **Hiểu về performance và index / Understand performance and indexes**
4. **Biết cách debug câu lệnh SQL / Know how to debug SQL statements**
5. **Hiểu về transaction và concurrency / Understand transactions and concurrency**
6. **Thành thạo với JOIN và subquery / Proficient with JOINs and subqueries**
7. **Biết sử dụng các hàm SQL cơ bản / Know how to use basic SQL functions**
8. **Hiểu về constraints và data integrity / Understand constraints and data integrity**
9. **Thực hành với stored procedures và views / Practice with stored procedures and views**
10. **Biết cách tối ưu câu lệnh SQL đơn giản / Know how to optimize simple SQL statements**
11. **Thành thạo với Window Functions và CTEs / Master Window Functions and CTEs**
12. **Hiểu về normalization và database design / Understand normalization and database design**
13. **Biết cách sử dụng temporary tables và table variables / Know how to use temporary tables and table variables**
14. **Thực hành pattern matching với wildcards / Practice pattern matching with wildcards**
15. **Hiểu về security best practices / Understand security best practices**

## Tài liệu tham khảo / References

- Microsoft SQL Server Documentation
- SQL Server Performance Tuning
- Database Design Fundamentals
- SQL Coding Standards
