# Transactions, Locking, Blocking, and Deadlocking Guide
# Hướng dẫn về Giao dịch, Khóa, Chặn và Deadlock

## Table of Contents / Mục lục
1. [Transactions / Giao dịch](#transactions--giao-dịch)
2. [Locking / Cơ chế khóa](#locking--cơ-chế-khóa)
3. [Blocking / Chặn](#blocking--chặn)
4. [Deadlocking / Deadlock](#deadlocking--deadlock)
5. [Monitoring and Troubleshooting / Giám sát và xử lý sự cố]
6. [Best Practices / Thực hành tốt nhất]
7. [Interview Questions / Câu hỏi phỏng vấn]

---

## 1. Transactions / Giao dịch

### 1.1 ACID Properties / Thuộc tính ACID

**Atomicity (Tính nguyên tử):** Tất cả các thao tác trong transaction phải thành công hoặc thất bại hoàn toàn
**Consistency (Tính nhất quán):** Database phải ở trạng thái nhất quán trước và sau transaction
**Isolation (Tính cô lập):** Các transaction chạy đồng thời không ảnh hưởng lẫn nhau
**Durability (Tính bền vững):** Kết quả của transaction được lưu trữ vĩnh viễn

### 1.2 Transaction Types / Loại giao dịch

```sql
-- Explicit Transaction / Giao dịch rõ ràng
BEGIN TRANSACTION
    INSERT INTO Orders (OrderID, CustomerID, OrderDate) 
    VALUES (1001, 'ALFKI', GETDATE())
    
    UPDATE Customers 
    SET LastOrderDate = GETDATE() 
    WHERE CustomerID = 'ALFKI'
    
    IF @@ERROR = 0
        COMMIT TRANSACTION
    ELSE
        ROLLBACK TRANSACTION

-- Implicit Transaction / Giao dịch ngầm định
SET IMPLICIT_TRANSACTIONS ON
INSERT INTO Orders (OrderID, CustomerID) VALUES (1002, 'BERGS')
COMMIT TRANSACTION
SET IMPLICIT_TRANSACTIONS OFF
```

### 1.3 Transaction Isolation Levels / Mức cô lập giao dịch

#### 1.3.1 Tổng quan về Isolation Levels

**Transaction Isolation Level** xác định mức độ cô lập giữa các transaction chạy đồng thời. Mức độ cô lập càng cao thì càng ít vấn đề concurrent, nhưng càng giảm performance.

#### 1.3.2 Các mức Isolation Level

**1. READ UNCOMMITTED (Đọc chưa commit)**
```sql
SET TRANSACTION ISOLATION LEVEL READ UNCOMMITTED
```

**Đặc điểm:**
- **Mức cô lập thấp nhất**
- **Cho phép Dirty Read** - Đọc dữ liệu chưa commit
- **Không sử dụng shared locks** khi đọc
- **Performance cao nhất** nhưng rủi ro cao nhất

**Ví dụ:**
```sql
-- Session 1
BEGIN TRANSACTION
UPDATE Customers SET ContactName = 'New Name' WHERE CustomerID = 'ALFKI'
-- Chưa commit

-- Session 2 (READ UNCOMMITTED)
SET TRANSACTION ISOLATION LEVEL READ UNCOMMITTED
SELECT ContactName FROM Customers WHERE CustomerID = 'ALFKI'
-- Kết quả: 'New Name' - Đọc được dữ liệu chưa commit!

-- Session 1
ROLLBACK -- Rollback transaction

-- Session 2
SELECT ContactName FROM Customers WHERE CustomerID = 'ALFKI'
-- Kết quả: 'Maria Anders' - Dữ liệu đã bị rollback!
```

**Khi nào sử dụng:**
- Báo cáo thống kê không cần độ chính xác cao
- Data warehouse, analytics
- Khi performance quan trọng hơn consistency

---

**2. READ COMMITTED (Đọc đã commit) - Mặc định**
```sql
SET TRANSACTION ISOLATION LEVEL READ COMMITTED
```

**Đặc điểm:**
- **Mức cô lập mặc định** của SQL Server
- **Ngăn chặn Dirty Read** - Chỉ đọc dữ liệu đã commit
- **Cho phép Non-Repeatable Read** - Dữ liệu có thể thay đổi giữa các lần đọc
- **Sử dụng shared locks** khi đọc, giải phóng ngay sau khi đọc

**Ví dụ:**
```sql
-- Session 1
BEGIN TRANSACTION
SELECT ContactName FROM Customers WHERE CustomerID = 'ALFKI'
-- Kết quả: 'Maria Anders'

-- Session 2 (cùng lúc)
UPDATE Customers SET ContactName = 'New Name' WHERE CustomerID = 'ALFKI'
COMMIT

-- Session 1 (đọc lại)
SELECT ContactName FROM Customers WHERE CustomerID = 'ALFKI'
-- Kết quả: 'New Name' - Dữ liệu đã thay đổi!
```

**Khi nào sử dụng:**
- **Hầu hết các trường hợp** trong ứng dụng
- Cân bằng tốt giữa consistency và performance
- Khi không cần đảm bảo dữ liệu không thay đổi trong transaction

---

**3. REPEATABLE READ (Đọc lặp lại)**
```sql
SET TRANSACTION ISOLATION LEVEL REPEATABLE READ
```

**Đặc điểm:**
- **Ngăn chặn Dirty Read và Non-Repeatable Read**
- **Cho phép Phantom Read** - Có thể có thêm rows mới
- **Giữ shared locks** cho đến khi transaction kết thúc
- **Performance thấp hơn** READ COMMITTED

**Ví dụ:**
```sql
-- Session 1
SET TRANSACTION ISOLATION LEVEL REPEATABLE READ
BEGIN TRANSACTION
SELECT ContactName FROM Customers WHERE CustomerID = 'ALFKI'
-- Kết quả: 'Maria Anders'

-- Session 2 (cùng lúc)
UPDATE Customers SET ContactName = 'New Name' WHERE CustomerID = 'ALFKI'
-- Bị block vì Session 1 đang giữ shared lock

-- Session 1 (đọc lại)
SELECT ContactName FROM Customers WHERE CustomerID = 'ALFKI'
-- Kết quả: 'Maria Anders' - Vẫn giống lần đầu!

COMMIT
-- Bây giờ Session 2 mới có thể update
```

**Ví dụ Phantom Read:**
```sql
-- Session 1
SET TRANSACTION ISOLATION LEVEL REPEATABLE READ
BEGIN TRANSACTION
SELECT COUNT(*) FROM Orders WHERE CustomerID = 'ALFKI'
-- Kết quả: 5 orders

-- Session 2 (cùng lúc)
INSERT INTO Orders (OrderID, CustomerID) VALUES (11000, 'ALFKI')
COMMIT

-- Session 1 (đọc lại)
SELECT COUNT(*) FROM Orders WHERE CustomerID = 'ALFKI'
-- Kết quả: 6 orders - Có thêm 1 order mới!
```

**Khi nào sử dụng:**
- Khi cần đảm bảo dữ liệu không thay đổi trong transaction
- Kiểm tra tồn kho, số dư tài khoản
- Khi cần consistency cao nhưng không cần ngăn phantom read

---

**4. SERIALIZABLE (Tuần tự hóa)**
```sql
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE
```

**Đặc điểm:**
- **Mức cô lập cao nhất**
- **Ngăn chặn tất cả vấn đề:** Dirty Read, Non-Repeatable Read, Phantom Read
- **Giữ shared locks** cho đến khi transaction kết thúc
- **Performance thấp nhất** do blocking nhiều

**Ví dụ:**
```sql
-- Session 1
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE
BEGIN TRANSACTION
SELECT COUNT(*) FROM Orders WHERE CustomerID = 'ALFKI'
-- Kết quả: 5 orders

-- Session 2 (cùng lúc)
INSERT INTO Orders (OrderID, CustomerID) VALUES (11000, 'ALFKI')
-- Bị block vì Session 1 đang giữ shared lock

-- Session 1 (đọc lại)
SELECT COUNT(*) FROM Orders WHERE CustomerID = 'ALFKI'
-- Kết quả: 5 orders - Vẫn giống lần đầu!

COMMIT
-- Bây giờ Session 2 mới có thể insert
```

**Khi nào sử dụng:**
- Khi cần đảm bảo dữ liệu hoàn toàn nhất quán
- Các thao tác critical như thanh toán, chuyển tiền
- Khi không thể chấp nhận bất kỳ thay đổi nào

---

**5. SNAPSHOT (Ảnh chụp)**
```sql
SET TRANSACTION ISOLATION LEVEL SNAPSHOT
```

**Đặc điểm:**
- **Sử dụng versioning** thay vì locking
- **Ngăn chặn tất cả vấn đề** concurrent
- **Không block** các transaction khác
- **Cần bật SNAPSHOT ISOLATION** cho database

**Cài đặt:**
```sql
-- Bật SNAPSHOT ISOLATION cho database
ALTER DATABASE YourDatabase SET ALLOW_SNAPSHOT_ISOLATION ON

-- Bật READ_COMMITTED_SNAPSHOT (tùy chọn)
ALTER DATABASE YourDatabase SET READ_COMMITTED_SNAPSHOT ON
```

**Ví dụ:**
```sql
-- Session 1
SET TRANSACTION ISOLATION LEVEL SNAPSHOT
BEGIN TRANSACTION
SELECT ContactName FROM Customers WHERE CustomerID = 'ALFKI'
-- Kết quả: 'Maria Anders'

-- Session 2 (cùng lúc)
UPDATE Customers SET ContactName = 'New Name' WHERE CustomerID = 'ALFKI'
COMMIT

-- Session 1 (đọc lại)
SELECT ContactName FROM Customers WHERE CustomerID = 'ALFKI'
-- Kết quả: 'Maria Anders' - Vẫn giống lần đầu!

COMMIT
```

**Khi nào sử dụng:**
- Khi cần consistency cao nhưng không muốn block
- Các ứng dụng có nhiều concurrent users
- Khi cần đảm bảo dữ liệu không thay đổi trong transaction

#### 1.3.3 So sánh các Isolation Levels

| Isolation Level | Dirty Read | Non-Repeatable Read | Phantom Read | Performance | Blocking |
|----------------|------------|-------------------|--------------|-------------|----------|
| **READ UNCOMMITTED** | ✓ | ✓ | ✓ | Cao nhất | Ít nhất |
| **READ COMMITTED** | ✗ | ✓ | ✓ | Cao | Trung bình |
| **REPEATABLE READ** | ✗ | ✗ | ✓ | Trung bình | Nhiều |
| **SERIALIZABLE** | ✗ | ✗ | ✗ | Thấp | Nhiều nhất |
| **SNAPSHOT** | ✗ | ✗ | ✗ | Trung bình | Không |

#### 1.3.4 Cách chọn Isolation Level phù hợp

**1. Cho các thao tác đọc đơn giản:**
```sql
-- Browse sản phẩm, xem danh sách
SET TRANSACTION ISOLATION LEVEL READ COMMITTED
```

**2. Cho các thao tác cần consistency cao:**
```sql
-- Kiểm tra tồn kho, số dư tài khoản
SET TRANSACTION ISOLATION LEVEL REPEATABLE READ
```

**3. Cho các thao tác critical:**
```sql
-- Thanh toán, chuyển tiền, cập nhật tồn kho
SET TRANSACTION ISOLATION LEVEL SNAPSHOT
```

**4. Cho báo cáo thống kê:**
```sql
-- Báo cáo, analytics, data warehouse
SET TRANSACTION ISOLATION LEVEL READ UNCOMMITTED
```

#### 1.3.5 Best Practices

**1. Sử dụng mức cô lập thấp nhất có thể:**
```sql
-- Chỉ sử dụng SERIALIZABLE khi thực sự cần thiết
-- Ưu tiên READ COMMITTED cho hầu hết trường hợp
```

**2. Kiểm tra performance impact:**
```sql
-- Monitor blocking và deadlock
-- Sử dụng Extended Events để theo dõi
```

**3. Sử dụng SNAPSHOT cho high-concurrency:**
```sql
-- Khi có nhiều concurrent users
-- Khi không muốn block các transaction khác
```

**4. Kết hợp với Lock Hints:**
```sql
-- Sử dụng lock hints khi cần thiết
SELECT * FROM Products WITH (HOLDLOCK) WHERE ProductID = 1
```

### 1.4 Concurrent Transaction Problems / Vấn đề giao dịch đồng thời

#### 1.4.1 Dirty Read (Đọc bẩn)
- **Định nghĩa:** Transaction A đọc dữ liệu chưa được commit từ Transaction B
- **Vấn đề:** Có thể đọc dữ liệu không nhất quán hoặc sẽ bị rollback
- **Ví dụ:**
```sql
-- Transaction A
BEGIN TRANSACTION
UPDATE Customers SET ContactName = 'New Name' WHERE CustomerID = 'ALFKI'
-- Chưa commit

-- Transaction B (cùng lúc)
SELECT ContactName FROM Customers WHERE CustomerID = 'ALFKI'
-- Đọc được 'New Name' mặc dù chưa commit!
```

#### 1.4.2 Non-Repeatable Read (Đọc không lặp lại)
- **Định nghĩa:** Transaction A đọc cùng một dữ liệu hai lần, nhưng kết quả khác nhau
- **Vấn đề:** Dữ liệu thay đổi giữa hai lần đọc
- **Ví dụ:**
```sql
-- Transaction A
BEGIN TRANSACTION
SELECT ContactName FROM Customers WHERE CustomerID = 'ALFKI'
-- Kết quả: 'Maria Anders'

-- Transaction B (cùng lúc)
UPDATE Customers SET ContactName = 'New Name' WHERE CustomerID = 'ALFKI'
COMMIT

-- Transaction A (đọc lại)
SELECT ContactName FROM Customers WHERE CustomerID = 'ALFKI'
-- Kết quả: 'New Name' - Khác với lần đọc đầu!
```

#### 1.4.3 Phantom Read (Đọc ma)
- **Định nghĩa:** Transaction A đọc một range dữ liệu, nhưng khi đọc lại thì có thêm rows mới
- **Vấn đề:** Số lượng rows thay đổi giữa các lần đọc
- **Ví dụ:**
```sql
-- Transaction A
BEGIN TRANSACTION
SELECT COUNT(*) FROM Orders WHERE CustomerID = 'ALFKI'
-- Kết quả: 5 orders

-- Transaction B (cùng lúc)
INSERT INTO Orders (OrderID, CustomerID) VALUES (11000, 'ALFKI')
COMMIT

-- Transaction A (đọc lại)
SELECT COUNT(*) FROM Orders WHERE CustomerID = 'ALFKI'
-- Kết quả: 6 orders - Có thêm 1 order mới!
```

#### 1.4.4 Lost Update (Cập nhật bị mất)
- **Định nghĩa:** Hai transactions cùng update cùng một dữ liệu, một trong hai bị mất
- **Vấn đề:** Dữ liệu bị ghi đè không mong muốn
- **Ví dụ:**
```sql
-- Transaction A
BEGIN TRANSACTION
SELECT UnitsInStock FROM Products WHERE ProductID = 1
-- Đọc: 100 units

-- Transaction B (cùng lúc)
BEGIN TRANSACTION
SELECT UnitsInStock FROM Products WHERE ProductID = 1
-- Đọc: 100 units

-- Transaction A
UPDATE Products SET UnitsInStock = UnitsInStock - 10 WHERE ProductID = 1
-- Cập nhật: 90 units
COMMIT

-- Transaction B
UPDATE Products SET UnitsInStock = UnitsInStock - 5 WHERE ProductID = 1
-- Cập nhật: 95 units (ghi đè kết quả của Transaction A!)
COMMIT
-- Kết quả cuối: 95 units thay vì 85 units
```

### 1.5 Tại sao cần "Đọc lại" và vấn đề gặp phải

#### 1.5.1 "Đọc lại" là gì?
**"Đọc lại"** có nghĩa là trong cùng một transaction, bạn thực hiện câu lệnh SELECT giống nhau nhiều lần để lấy cùng một dữ liệu.

#### 1.5.2 Tại sao phải đọc lại?

**1. Business Logic yêu cầu:**
```sql
-- Ví dụ: Kiểm tra số dư trước và sau khi thực hiện giao dịch
BEGIN TRANSACTION

-- Kiểm tra số dư ban đầu
SELECT Balance FROM BankAccount WHERE AccountID = 'ACC001'
-- Kết quả: $1000

-- Thực hiện giao dịch
UPDATE BankAccount SET Balance = Balance - 100 WHERE AccountID = 'ACC001'

-- Kiểm tra số dư sau giao dịch (ĐỌC LẠI)
SELECT Balance FROM BankAccount WHERE AccountID = 'ACC001'
-- Kết quả: $900

COMMIT
```

**2. Validation trong transaction:**
```sql
-- Kiểm tra điều kiện trước và sau khi update
BEGIN TRANSACTION

-- Kiểm tra trạng thái ban đầu
SELECT Status FROM Orders WHERE OrderID = 10248
-- Kết quả: 'Pending'

-- Cập nhật trạng thái
UPDATE Orders SET Status = 'Processing' WHERE OrderID = 10248

-- Kiểm tra trạng thái đã thay đổi chưa (ĐỌC LẠI)
SELECT Status FROM Orders WHERE OrderID = 10248
-- Kết quả: 'Processing'

COMMIT
```

**3. Tính toán phức tạp:**
```sql
-- Tính toán dựa trên dữ liệu đã thay đổi
BEGIN TRANSACTION

-- Đọc dữ liệu gốc
SELECT UnitsInStock, ReorderLevel FROM Products WHERE ProductID = 1
-- Kết quả: UnitsInStock = 50, ReorderLevel = 20

-- Cập nhật số lượng
UPDATE Products SET UnitsInStock = UnitsInStock - 30 WHERE ProductID = 1

-- Kiểm tra có cần đặt hàng không (ĐỌC LẠI)
SELECT UnitsInStock, ReorderLevel FROM Products WHERE ProductID = 1
-- Kết quả: UnitsInStock = 20, ReorderLevel = 20
-- Logic: Nếu UnitsInStock <= ReorderLevel thì cần đặt hàng

COMMIT
```

#### 1.5.3 Vấn đề khi đọc lại trong môi trường concurrent:

**Scenario thực tế:**
```sql
-- Session 1: Transaction A
BEGIN TRANSACTION
SELECT ContactName FROM Customers WHERE CustomerID = 'ALFKI'
-- Kết quả: 'Maria Anders'
-- Đang xử lý logic...

-- Session 2: Transaction B (cùng lúc)
BEGIN TRANSACTION
UPDATE Customers SET ContactName = 'New Name' WHERE CustomerID = 'ALFKI'
COMMIT
-- Đã thay đổi thành 'New Name'

-- Session 1: Transaction A (tiếp tục)
-- Làm một số việc khác...
UPDATE Orders SET ShipCity = 'Berlin' WHERE OrderID = 10248

-- ĐỌC LẠI để kiểm tra
SELECT ContactName FROM Customers WHERE CustomerID = 'ALFKI'
-- Kết quả: 'New Name' - Đã thay đổi!
-- Điều này có thể gây ra vấn đề trong business logic
```

### 1.6 Cách ngăn chặn các vấn đề concurrent transactions

#### 1.6.1 Sử dụng Isolation Levels phù hợp:

```sql
-- 1. REPEATABLE READ - Ngăn Non-Repeatable Read
SET TRANSACTION ISOLATION LEVEL REPEATABLE READ

BEGIN TRANSACTION
SELECT ContactName FROM Customers WHERE CustomerID = 'ALFKI'
-- Kết quả: 'Maria Anders'

-- Làm việc khác...

SELECT ContactName FROM Customers WHERE CustomerID = 'ALFKI'
-- Kết quả: Vẫn là 'Maria Anders' - Không thay đổi!
COMMIT

-- 2. SNAPSHOT - Sử dụng versioning
SET TRANSACTION ISOLATION LEVEL SNAPSHOT

BEGIN TRANSACTION
SELECT ContactName FROM Customers WHERE CustomerID = 'ALFKI'
-- Kết quả: 'Maria Anders'

-- Làm việc khác...

SELECT ContactName FROM Customers WHERE CustomerID = 'ALFKI'
-- Kết quả: Vẫn là 'Maria Anders' - Dữ liệu được snapshot!
COMMIT
```

#### 1.6.2 Sử dụng Lock Hints:

```sql
-- Ngăn Lost Update
SELECT UnitsInStock FROM Products WITH (UPDLOCK) WHERE ProductID = 1

-- Ngăn Phantom Read
SELECT * FROM Orders WITH (HOLDLOCK) WHERE CustomerID = 'ALFKI'

-- Ngăn Dirty Read
SELECT * FROM Customers WITH (READCOMMITTED) WHERE CustomerID = 'ALFKI'

-- Lock dữ liệu để không bị thay đổi
SELECT ContactName FROM Customers WITH (HOLDLOCK) WHERE CustomerID = 'ALFKI'
```

#### 1.6.3 Sử dụng Optimistic Concurrency:

```sql
-- Kiểm tra version trước khi update
UPDATE Products 
SET UnitsInStock = @NewStock, 
    RowVersion = @NewRowVersion
WHERE ProductID = @ProductID 
  AND RowVersion = @OldRowVersion

-- Nếu @@ROWCOUNT = 0, có nghĩa là dữ liệu đã thay đổi
```

#### 1.6.4 Tóm tắt các giải pháp:

| Vấn đề | Mô tả | Isolation Level ngăn chặn | Lock Hint |
|--------|-------|---------------------------|-----------|
| **Dirty Read** | Đọc dữ liệu chưa commit | READ COMMITTED+ | READCOMMITTED |
| **Non-Repeatable Read** | Dữ liệu thay đổi giữa các lần đọc | REPEATABLE READ+ | HOLDLOCK |
| **Phantom Read** | Có thêm rows mới | SERIALIZABLE | HOLDLOCK |
| **Lost Update** | Dữ liệu bị ghi đè | SERIALIZABLE | UPDLOCK |

### 1.7 Ví dụ thực tế trong E-Commerce / Real-world E-Commerce Examples

#### 1.7.1 Bài toán "Đọc lại" trong E-Commerce

**Scenario 1: Kiểm tra tồn kho và đặt hàng**

```sql
-- Vấn đề: Kiểm tra tồn kho trước và sau khi đặt hàng
BEGIN TRANSACTION

-- Lần đọc đầu tiên: Kiểm tra tồn kho
SELECT UnitsInStock FROM Products WHERE ProductID = 1001
-- Kết quả: 50 units

-- Kiểm tra giá hiện tại
SELECT UnitPrice FROM Products WHERE ProductID = 1001
-- Kết quả: $29.99

-- Xử lý logic đặt hàng...
-- Có thể mất thời gian để xử lý payment, validation...

-- Lần đọc thứ hai: Kiểm tra lại tồn kho (ĐỌC LẠI)
SELECT UnitsInStock FROM Products WHERE ProductID = 1001
-- Kết quả: 45 units - Đã thay đổi!
-- Có thể do user khác đã mua trong lúc này

-- Lần đọc thứ ba: Kiểm tra lại giá (ĐỌC LẠI)
SELECT UnitPrice FROM Products WHERE ProductID = 1001
-- Kết quả: $34.99 - Giá đã tăng!
-- Có thể do admin thay đổi giá

-- Vấn đề: Dữ liệu không nhất quán trong cùng transaction!
COMMIT
```

**Scenario 2: Quản lý giỏ hàng và thanh toán**

```sql
-- Vấn đề: Kiểm tra trạng thái giỏ hàng trước và sau khi xử lý
BEGIN TRANSACTION

-- Lần đọc đầu tiên: Kiểm tra trạng thái giỏ hàng
SELECT CartStatus, TotalAmount FROM ShoppingCart WHERE CartID = 5001
-- Kết quả: CartStatus = 'Active', TotalAmount = $150.00

-- Kiểm tra số lượng sản phẩm
SELECT COUNT(*) FROM CartItems WHERE CartID = 5001
-- Kết quả: 3 items

-- Xử lý thanh toán...
-- Có thể mất thời gian để xác thực thẻ, xử lý payment gateway...

-- Lần đọc thứ hai: Kiểm tra lại trạng thái (ĐỌC LẠI)
SELECT CartStatus, TotalAmount FROM ShoppingCart WHERE CartID = 5001
-- Kết quả: CartStatus = 'Processing', TotalAmount = $150.00

-- Lần đọc thứ ba: Kiểm tra lại số lượng (ĐỌC LẠI)
SELECT COUNT(*) FROM CartItems WHERE CartID = 5001
-- Kết quả: 2 items - Có thể user đã xóa 1 item!

-- Vấn đề: Giỏ hàng thay đổi trong quá trình thanh toán!
COMMIT
```

**Scenario 3: Quản lý coupon và discount**

```sql
-- Vấn đề: Kiểm tra coupon trước và sau khi áp dụng
BEGIN TRANSACTION

-- Lần đọc đầu tiên: Kiểm tra coupon còn hiệu lực không
SELECT CouponCode, DiscountPercent, ValidUntil, UsageCount, MaxUsage 
FROM Coupons WHERE CouponCode = 'SAVE20'
-- Kết quả: DiscountPercent = 20%, ValidUntil = '2024-12-31', UsageCount = 45, MaxUsage = 100

-- Kiểm tra user đã sử dụng coupon này chưa
SELECT COUNT(*) FROM CouponUsage WHERE CouponCode = 'SAVE20' AND UserID = 1001
-- Kết quả: 0 - Chưa sử dụng

-- Xử lý logic áp dụng coupon...
-- Có thể mất thời gian để tính toán discount...

-- Lần đọc thứ hai: Kiểm tra lại coupon (ĐỌC LẠI)
SELECT CouponCode, DiscountPercent, ValidUntil, UsageCount, MaxUsage 
FROM Coupons WHERE CouponCode = 'SAVE20'
-- Kết quả: UsageCount = 47 - Đã có 2 người khác sử dụng!

-- Lần đọc thứ ba: Kiểm tra lại user (ĐỌC LẠI)
SELECT COUNT(*) FROM CouponUsage WHERE CouponCode = 'SAVE20' AND UserID = 1001
-- Kết quả: 1 - User đã sử dụng coupon này rồi!

-- Vấn đề: Coupon có thể đã hết hạn hoặc user đã sử dụng!
COMMIT
```

#### 1.7.2 Giải pháp cho bài toán E-Commerce

**Solution 1: Sử dụng REPEATABLE READ Isolation Level**

```sql
-- Áp dụng cho scenario kiểm tra tồn kho
SET TRANSACTION ISOLATION LEVEL REPEATABLE READ

BEGIN TRANSACTION

-- Lần đọc đầu tiên: Kiểm tra tồn kho
SELECT UnitsInStock, UnitPrice FROM Products WITH (HOLDLOCK) WHERE ProductID = 1001
-- Kết quả: 50 units, $29.99

-- Xử lý logic đặt hàng...
-- Có thể mất thời gian để xử lý payment, validation...

-- Lần đọc thứ hai: Kiểm tra lại tồn kho (ĐỌC LẠI)
SELECT UnitsInStock, UnitPrice FROM Products WHERE ProductID = 1001
-- Kết quả: Vẫn 50 units, $29.99 - Không thay đổi!

-- Tiếp tục xử lý với dữ liệu nhất quán
COMMIT
```

**Solution 2: Sử dụng SNAPSHOT Isolation Level**

```sql
-- Áp dụng cho scenario quản lý giỏ hàng
SET TRANSACTION ISOLATION LEVEL SNAPSHOT

BEGIN TRANSACTION

-- Lần đọc đầu tiên: Kiểm tra trạng thái giỏ hàng
SELECT CartStatus, TotalAmount, 
       (SELECT COUNT(*) FROM CartItems WHERE CartID = 5001) AS ItemCount
FROM ShoppingCart WHERE CartID = 5001
-- Kết quả: CartStatus = 'Active', TotalAmount = $150.00, ItemCount = 3

-- Xử lý thanh toán...
-- Có thể mất thời gian để xác thực thẻ, xử lý payment gateway...

-- Lần đọc thứ hai: Kiểm tra lại (ĐỌC LẠI)
SELECT CartStatus, TotalAmount, 
       (SELECT COUNT(*) FROM CartItems WHERE CartID = 5001) AS ItemCount
FROM ShoppingCart WHERE CartID = 5001
-- Kết quả: Vẫn giống hệt lần đầu - Dữ liệu được snapshot!

COMMIT
```

**Solution 3: Sử dụng Optimistic Concurrency Control**

```sql
-- Áp dụng cho scenario quản lý coupon
BEGIN TRANSACTION

-- Lần đọc đầu tiên: Kiểm tra coupon với version
SELECT CouponCode, DiscountPercent, ValidUntil, UsageCount, MaxUsage, RowVersion
FROM Coupons WHERE CouponCode = 'SAVE20'
-- Kết quả: DiscountPercent = 20%, UsageCount = 45, RowVersion = 123

-- Xử lý logic áp dụng coupon...
-- Có thể mất thời gian để tính toán discount...

-- Cập nhật coupon với kiểm tra version
UPDATE Coupons 
SET UsageCount = UsageCount + 1,
    RowVersion = RowVersion + 1
WHERE CouponCode = 'SAVE20' 
  AND RowVersion = 123  -- Version ban đầu
  AND UsageCount < MaxUsage
  AND ValidUntil > GETDATE()

-- Kiểm tra xem update có thành công không
IF @@ROWCOUNT = 0
BEGIN
    -- Coupon đã thay đổi, rollback và thông báo lỗi
    ROLLBACK
    RAISERROR('Coupon đã thay đổi hoặc hết hạn, vui lòng thử lại', 16, 1)
    RETURN
END

-- Thêm record sử dụng coupon
INSERT INTO CouponUsage (CouponCode, UserID, UsedDate, OrderID)
VALUES ('SAVE20', 1001, GETDATE(), @OrderID)

COMMIT
```

**Solution 4: Sử dụng Lock Hints cụ thể**

```sql
-- Áp dụng cho scenario kiểm tra tồn kho
BEGIN TRANSACTION

-- Lock sản phẩm để đảm bảo không bị thay đổi
SELECT UnitsInStock, UnitPrice 
FROM Products WITH (UPDLOCK, HOLDLOCK) 
WHERE ProductID = 1001
-- Kết quả: 50 units, $29.99

-- Xử lý logic đặt hàng...
-- Có thể mất thời gian để xử lý payment, validation...

-- Lần đọc thứ hai: Kiểm tra lại (ĐỌC LẠI)
SELECT UnitsInStock, UnitPrice 
FROM Products 
WHERE ProductID = 1001
-- Kết quả: Vẫn 50 units, $29.99 - Không thay đổi!

-- Cập nhật tồn kho
UPDATE Products 
SET UnitsInStock = UnitsInStock - @Quantity
WHERE ProductID = 1001

COMMIT
```

#### 1.7.3 Best Practices cho E-Commerce

**1. Sử dụng Isolation Level phù hợp:**
```sql
-- Cho các thao tác đọc đơn giản
SET TRANSACTION ISOLATION LEVEL READ COMMITTED

-- Cho các thao tác cần consistency cao
SET TRANSACTION ISOLATION LEVEL REPEATABLE READ

-- Cho các thao tác critical (thanh toán, tồn kho)
SET TRANSACTION ISOLATION LEVEL SNAPSHOT
```

**2. Implement Retry Logic:**
```sql
-- Retry logic cho deadlock hoặc update conflict
DECLARE @RetryCount INT = 0
DECLARE @MaxRetries INT = 3

WHILE @RetryCount < @MaxRetries
BEGIN
    BEGIN TRY
        BEGIN TRANSACTION
        
        -- Thực hiện logic business
        UPDATE Products SET UnitsInStock = UnitsInStock - @Quantity
        WHERE ProductID = @ProductID AND UnitsInStock >= @Quantity
        
        IF @@ROWCOUNT = 0
            RAISERROR('Không đủ tồn kho', 16, 1)
            
        COMMIT
        BREAK -- Thành công, thoát vòng lặp
        
    END TRY
    BEGIN CATCH
        IF @@TRANCOUNT > 0
            ROLLBACK
            
        IF ERROR_NUMBER() = 1205 -- Deadlock
        BEGIN
            SET @RetryCount = @RetryCount + 1
            WAITFOR DELAY '00:00:01' -- Chờ 1 giây
            CONTINUE
        END
        
        -- Lỗi khác, không retry
        THROW
    END CATCH
END
```

**3. Sử dụng Application-Level Locking:**
```sql
-- Sử dụng application lock để đồng bộ hóa
DECLARE @LockName NVARCHAR(32) = 'ECommerce_Order_' + CAST(@OrderID AS NVARCHAR(10))

-- Acquire lock
IF APPLOCK_TEST('public', @LockName, 'Exclusive') = 0
BEGIN
    -- Không thể acquire lock, có thể do order đang được xử lý
    RAISERROR('Order đang được xử lý, vui lòng thử lại', 16, 1)
    RETURN
END

-- Acquire lock
EXEC sp_getapplock @LockName, 'Exclusive'

BEGIN TRY
    -- Thực hiện logic business
    -- ...
    
    COMMIT
END TRY
BEGIN CATCH
    IF @@TRANCOUNT > 0
        ROLLBACK
    THROW
END CATCH

-- Release lock
EXEC sp_releaseapplock @LockName
```

---

## 2. Locking / Cơ chế khóa

### 2.1 Lock Types / Loại khóa

#### 2.1.1 Shared Locks (S) / Khóa chia sẻ
- Được sử dụng khi đọc dữ liệu
- Nhiều transaction có thể giữ shared lock cùng lúc
- Không tương thích với Exclusive lock

#### 2.1.2 Exclusive Locks (X) / Khóa độc quyền
- Được sử dụng khi ghi dữ liệu (INSERT, UPDATE, DELETE)
- Chỉ một transaction có thể giữ exclusive lock
- Không tương thích với bất kỳ lock nào khác

#### 2.1.3 Update Locks (U) / Khóa cập nhật
- Được sử dụng trước khi chuyển thành exclusive lock
- Ngăn chặn deadlock trong UPDATE operations

#### 2.1.4 Intent Locks (IS, IX, IU) / Khóa ý định
- Báo hiệu ý định khóa ở mức thấp hơn
- IS: Intent Shared, IX: Intent Exclusive, IU: Intent Update

### 2.2 Lock Granularity / Mức độ chi tiết của khóa

```sql
-- Row-level locks (Default)
ALTER TABLE Orders SET (LOCK_ESCALATION = DISABLE)

-- Page-level locks
ALTER TABLE Orders SET (LOCK_ESCALATION = AUTO)

-- Table-level locks
ALTER TABLE Orders SET (LOCK_ESCALATION = TABLE)
```

### 2.3 Lock Compatibility Matrix / Ma trận tương thích khóa

| Lock Type | S | U | X | IS | IU | IX |
|-----------|---|---|---|----|----|----|
| S         | ✓ | ✓ | ✗ | ✓  | ✓  | ✗  |
| U         | ✓ | ✗ | ✗ | ✓  | ✗  | ✗  |
| X         | ✗ | ✗ | ✗ | ✗  | ✗  | ✗  |
| IS        | ✓ | ✓ | ✗ | ✓  | ✓  | ✓  |
| IU        | ✓ | ✗ | ✗ | ✓  | ✗  | ✓  |
| IX        | ✗ | ✗ | ✗ | ✓  | ✓  | ✓  |

---

## 3. Blocking / Chặn

### 3.1 What is Blocking? / Blocking là gì?

Blocking xảy ra khi một transaction đang chờ lock được giải phóng bởi transaction khác.

### 3.2 Identifying Blocking / Xác định blocking

```sql
-- Query để tìm blocking
SELECT 
    w.session_id AS blocked_session_id,
    w.wait_duration_ms,
    w.wait_type,
    w.resource_description,
    w.blocking_session_id,
    s.login_name,
    s.host_name,
    s.program_name,
    s.status,
    s.cpu_time,
    s.memory_usage,
    s.reads,
    s.writes,
    s.logical_reads,
    s.total_elapsed_time,
    s.last_request_start_time,
    s.last_request_end_time,
    s.row_count,
    s.prev_error
FROM sys.dm_os_waiting_tasks w
INNER JOIN sys.dm_exec_sessions s ON w.session_id = s.session_id
WHERE w.blocking_session_id > 0
ORDER BY w.wait_duration_ms DESC

-- Query để tìm blocking chain
WITH BlockingChain AS (
    SELECT 
        session_id,
        blocking_session_id,
        0 AS level
    FROM sys.dm_exec_requests
    WHERE blocking_session_id > 0
    
    UNION ALL
    
    SELECT 
        r.session_id,
        r.blocking_session_id,
        bc.level + 1
    FROM sys.dm_exec_requests r
    INNER JOIN BlockingChain bc ON r.session_id = bc.blocking_session_id
    WHERE r.blocking_session_id > 0
)
SELECT 
    session_id,
    blocking_session_id,
    level
FROM BlockingChain
ORDER BY level, session_id
```

### 3.3 Resolving Blocking / Giải quyết blocking

```sql
-- Kill blocking session (cẩn thận!)
KILL 123

-- Kill blocking session with rollback
KILL 123 WITH STATUSONLY

-- View what session is doing
DBCC INPUTBUFFER(123)

-- View session details
SELECT * FROM sys.dm_exec_sessions WHERE session_id = 123
```

---

## 4. Deadlocking / Deadlock

### 4.1 What is Deadlock? / Deadlock là gì?

Deadlock xảy ra khi hai hoặc nhiều transaction chờ đợi lẫn nhau để giải phóng lock.

### 4.2 Deadlock Example / Ví dụ về deadlock

```sql
-- Session 1
BEGIN TRANSACTION
UPDATE Customers SET ContactName = 'New Name' WHERE CustomerID = 'ALFKI'
-- Chờ lock từ Session 2

-- Session 2  
BEGIN TRANSACTION
UPDATE Orders SET ShipCity = 'New City' WHERE OrderID = 10248
-- Chờ lock từ Session 1
-- DEADLOCK!
```

### 4.3 Deadlock Detection / Phát hiện deadlock

```sql
-- Enable deadlock monitoring
DBCC TRACEON (1222, -1)
DBCC TRACEON (1204, -1)

-- View deadlock information
SELECT * FROM sys.event_log 
WHERE event_type = 'deadlock'

-- Check deadlock graph
SELECT * FROM sys.dm_xe_sessions s
INNER JOIN sys.dm_xe_session_events e ON s.address = e.event_session_address
WHERE s.name = 'system_health'
AND e.event_name = 'xml_deadlock_report'
```

### 4.4 Deadlock Prevention / Ngăn chặn deadlock

```sql
-- 1. Consistent access order
-- Luôn truy cập bảng theo thứ tự nhất quán
BEGIN TRANSACTION
UPDATE Customers SET ContactName = 'New Name' WHERE CustomerID = 'ALFKI'
UPDATE Orders SET ShipCity = 'New City' WHERE OrderID = 10248
COMMIT

-- 2. Use NOLOCK hint (cẩn thận!)
SELECT * FROM Customers WITH (NOLOCK) WHERE CustomerID = 'ALFKI'

-- 3. Use READPAST hint
SELECT * FROM Orders WITH (READPAST) WHERE OrderID = 10248

-- 4. Use UPDLOCK hint
SELECT * FROM Customers WITH (UPDLOCK) WHERE CustomerID = 'ALFKI'
```

---

## 5. Monitoring and Troubleshooting / Giám sát và xử lý sự cố

### 5.1 Performance Counters / Bộ đếm hiệu suất

```sql
-- Lock wait time
SELECT 
    object_name,
    counter_name,
    cntr_value
FROM sys.dm_os_performance_counters
WHERE counter_name LIKE '%lock%'

-- Deadlock rate
SELECT 
    object_name,
    counter_name,
    cntr_value
FROM sys.dm_os_performance_counters
WHERE counter_name LIKE '%deadlock%'
```

### 5.2 Extended Events / Sự kiện mở rộng

```sql
-- Create deadlock monitoring session
CREATE EVENT SESSION [Deadlock_Monitoring] ON SERVER 
ADD EVENT sqlserver.xml_deadlock_report
ADD TARGET package0.event_file(SET filename=N'C:\Deadlock_Monitoring.xel')
WITH (MAX_MEMORY=4096 KB,EVENT_RETENTION_MODE=ALLOW_SINGLE_EVENT_LOSS,
MAX_DISPATCH_LATENCY=30 SECONDS,MAX_EVENT_SIZE=0 KB,MEMORY_PARTITION_MODE=NONE,
TRACK_CAUSALITY=OFF,STARTUP_STATE=OFF)
GO

-- Start the session
ALTER EVENT SESSION [Deadlock_Monitoring] ON SERVER STATE = START
```

### 5.3 Dynamic Management Views / Khung nhìn quản lý động

```sql
-- Current locks
SELECT 
    l.resource_type,
    l.resource_database_id,
    l.resource_description,
    l.request_mode,
    l.request_type,
    l.request_status,
    s.login_name,
    s.host_name,
    s.program_name
FROM sys.dm_tran_locks l
INNER JOIN sys.dm_exec_sessions s ON l.request_session_id = s.session_id
WHERE l.request_status = 'WAIT'

-- Lock escalation events
SELECT 
    object_name,
    counter_name,
    cntr_value
FROM sys.dm_os_performance_counters
WHERE counter_name LIKE '%escalation%'
```

---

## 6. Best Practices / Thực hành tốt nhất

### 6.1 Transaction Design / Thiết kế giao dịch

1. **Keep transactions short** / Giữ giao dịch ngắn gọn
2. **Access tables in consistent order** / Truy cập bảng theo thứ tự nhất quán
3. **Use appropriate isolation levels** / Sử dụng mức cô lập phù hợp
4. **Avoid long-running transactions** / Tránh giao dịch chạy lâu

### 6.2 Lock Management / Quản lý khóa

1. **Use row-level locking when possible** / Sử dụng khóa cấp hàng khi có thể
2. **Avoid table hints unless necessary** / Tránh table hints trừ khi cần thiết
3. **Monitor lock escalation** / Giám sát lock escalation
4. **Use appropriate lock timeouts** / Sử dụng timeout khóa phù hợp

### 6.3 Deadlock Prevention / Ngăn chặn deadlock

1. **Implement retry logic** / Triển khai logic thử lại
2. **Use deadlock priority** / Sử dụng ưu tiên deadlock
3. **Monitor and analyze deadlock patterns** / Giám sát và phân tích mẫu deadlock
4. **Use application-level deadlock handling** / Xử lý deadlock ở cấp ứng dụng

---

## 7. Interview Questions / Câu hỏi phỏng vấn

### 7.1 Basic Questions / Câu hỏi cơ bản

**Q: What is the difference between blocking and deadlock?**
**A:** Blocking occurs when one transaction waits for another to release a lock, while deadlock occurs when two or more transactions wait for each other in a circular manner.

**Q: Explain ACID properties of transactions.**
**A:** ACID stands for Atomicity, Consistency, Isolation, and Durability. These properties ensure data integrity and reliability in database transactions.

**Q: What are the different isolation levels in SQL Server?**
**A:** READ UNCOMMITTED, READ COMMITTED, REPEATABLE READ, SERIALIZABLE, and SNAPSHOT.

### 7.2 Advanced Questions / Câu hỏi nâng cao

**Q: How would you troubleshoot a blocking issue in production?**
**A:** Use sys.dm_os_waiting_tasks to identify blocked sessions, analyze the blocking chain, and resolve by either waiting, killing the blocking session, or optimizing the query.

**Q: What strategies would you use to prevent deadlocks?**
**A:** Consistent access order, appropriate isolation levels, retry logic, deadlock priority, and application-level deadlock handling.

**Q: How do you monitor lock escalation?**
**A:** Use sys.dm_os_performance_counters, Extended Events, and analyze lock escalation events to understand when and why locks escalate.

### 7.3 Practical Scenarios / Tình huống thực tế

**Scenario 1:** A production database is experiencing frequent blocking during peak hours. How would you investigate and resolve this?

**Scenario 2:** Users are reporting deadlock errors when updating customer information. What steps would you take to diagnose and fix this issue?

**Scenario 3:** You need to implement a solution to prevent deadlocks in a high-concurrency environment. What approach would you recommend?

---

## 8. Useful Scripts / Script hữu ích

### 8.1 Blocking Monitor / Giám sát blocking

```sql
-- Real-time blocking monitor
CREATE PROCEDURE sp_BlockingMonitor
AS
BEGIN
    SET NOCOUNT ON
    
    WHILE 1=1
    BEGIN
        IF EXISTS (
            SELECT 1 FROM sys.dm_os_waiting_tasks 
            WHERE blocking_session_id > 0
        )
        BEGIN
            SELECT 
                GETDATE() AS CheckTime,
                w.session_id AS blocked_session_id,
                w.blocking_session_id,
                w.wait_duration_ms,
                w.wait_type,
                s.login_name,
                s.host_name,
                s.program_name
            FROM sys.dm_os_waiting_tasks w
            INNER JOIN sys.dm_exec_sessions s ON w.session_id = s.session_id
            WHERE w.blocking_session_id > 0
            
            WAITFOR DELAY '00:00:05'
        END
        ELSE
        BEGIN
            PRINT 'No blocking detected at ' + CAST(GETDATE() AS VARCHAR(20))
            WAITFOR DELAY '00:00:10'
        END
    END
END
```

### 8.2 Deadlock Analysis / Phân tích deadlock

```sql
-- Analyze deadlock patterns
SELECT 
    event_data.value('(/event/@timestamp)[1]', 'DATETIME2') AS event_time,
    event_data.value('(/event/data[@name="xml_report"]/value)[1]', 'XML') AS deadlock_graph
FROM (
    SELECT CAST(target_data AS XML) AS target_data
    FROM sys.dm_xe_session_targets st
    INNER JOIN sys.dm_xe_sessions s ON st.event_session_address = s.address
    WHERE s.name = 'system_health'
    AND st.target_name = 'ring_buffer'
) AS tab
CROSS APPLY target_data.nodes('RingBufferTarget/event[@name="xml_deadlock_report"]') AS tab2(event_data)
ORDER BY event_time DESC
```

---

## 9. References / Tài liệu tham khảo

1. Microsoft SQL Server Documentation
2. SQL Server Internals by Kalen Delaney
3. Professional SQL Server Performance Tuning
4. SQL Server Wait Types and Locks

---

*Tài liệu này cung cấp kiến thức cơ bản và nâng cao về Transactions, Locking, Blocking, và Deadlocking trong SQL Server, phù hợp cho cả học tập và phỏng vấn DBA.*
