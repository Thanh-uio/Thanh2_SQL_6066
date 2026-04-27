# BÀI KIỂM TRA SỐ 02 - HỆ QUẢN TRỊ CSDL
**Họ tên:** Mông Chí Thành 

**MSSV:** k235480106066

**lớp:** k59KMT

**Chủ đề:** Hệ thống Quản lý Cửa hàng Bán điện thoại


## PHẦN 1: THIẾT KẾ VÀ KHỞI TẠO CẤU TRÚC DỮ LIỆU

### 1.1. Tạo Database và Bảng Khách Hàng
- Khóa chính `MaKhachHang` sử dụng thuộc tính `IDENTITY(1,1)` để hệ thống tự động sinh số thứ tự, tránh việc nhân viên nhập trùng mã.
- Cột `HoTenKhachHang` sử dụng kiểu dữ liệu `NVARCHAR` để hỗ trợ lưu trữ tốt tiếng Việt có dấu.
- Cột `DiemThanhVien` được thiết lập mặc định (`DEFAULT`) là 5 sao và bị giới hạn bởi ràng buộc `CHECK` để đảm bảo điểm số không bị nhập sai ngoài khoảng 1-5.

```sql
CREATE DATABASE [QuanLyBanDienThoai_k235480106066];
GO
USE [QuanLyBanDienThoai_k235480106066];
GO

CREATE TABLE [KhachHang] (
    [MaKhachHang] INT IDENTITY(1,1) NOT NULL,
    [HoTenKhachHang] NVARCHAR(100) NOT NULL,
    [NgaySinh] DATE,
    [SoDienThoai] VARCHAR(15) NOT NULL,
    [DiemThanhVien] INT DEFAULT 5,
    CONSTRAINT [PK_KhachHang] PRIMARY KEY ([MaKhachHang]),
    CONSTRAINT [CK_DiemThanhVien] CHECK ([DiemThanhVien] >= 1 AND [DiemThanhVien] <= 5)
);
```
_Hình 1.1 Tạo bảng Khách hàng:_
<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/0a6e1962-e9d5-4e59-ae5c-327805ea2f46" />



### 1.2. Tạo Bảng Điện Thoại
- Bảng này lưu danh mục sản phẩm. Cột `GiaBan` sử dụng kiểu `MONEY` chuyên dụng trong SQL Server để tối ưu hóa việc lưu trữ và tính toán tiền tệ. 
- Ràng buộc cứng `CK_GiaBan` ngăn chặn lỗi logic nghiêm trọng: Không được phép nhập giá bán âm hoặc bằng 0.

```sql
CREATE TABLE [DienThoai] (
    [MaDT] INT PRIMARY KEY,
    [TenDT] NVARCHAR(100) NOT NULL,
    [HangSX] NVARCHAR(50),
    [GiaBan] MONEY,
    [SoLuongTon] INT DEFAULT 0,
    CONSTRAINT [CK_GiaBan] CHECK ([GiaBan] > 0)
);
```
_Hình 1.2 Tạo bảng Điện thoại:_
<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/ff92f623-600b-4003-89f9-d3b0021cf80d" />




### 1.3. Tạo Bảng Hóa Đơn
- Đây là bảng giao dịch (Transaction Table) đóng vai trò kết nối quan hệ nhiều - nhiều giữa `KhachHang` và `DienThoai`. 
- Khi tạo 2 khóa ngoại (`FOREIGN KEY`), hệ thống sẽ đảm bảo tính toàn vẹn tham chiếu: Không thể tạo hóa đơn cho một khách hàng không tồn tại hoặc bán một mã điện thoại không có trong danh mục.

```sql
CREATE TABLE [HoaDon] (
    [MaHD] INT IDENTITY(1,1) PRIMARY KEY,
    [MaKhachHang] INT,
    [MaDT] INT,
    [NgayMua] DATE DEFAULT GETDATE(),
    [SoLuongMua] INT,
    FOREIGN KEY ([MaKhachHang]) REFERENCES [KhachHang]([MaKhachHang]),
    FOREIGN KEY ([MaDT]) REFERENCES [DienThoai]([MaDT]),
    CONSTRAINT [CK_SoLuongMua] CHECK ([SoLuongMua] >= 1)
);
```
_Hình 1.3 Tạo bảng Hóa đơn:_
<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/1689dce1-846d-4e05-b448-6fd89fb1eaa6" />

### 1.4. Chèn Dữ liệu mẫu (Sample Data)

```sql
-- 1. Thêm 10 Khách hàng
INSERT INTO [KhachHang] ([HoTenKhachHang], [NgaySinh], [SoDienThoai], [DiemThanhVien])
VALUES 
(N'Nguyễn Văn An', '1995-05-12', '0988123456', 5),
(N'Trần Thị Bích', '2000-10-20', '0912345678', 4),
(N'Lê Hoàng Nam', '1998-02-28', '0909112233', 3),
(N'Phạm Thu Thủy', '2002-07-15', '0933445566', 5),
(N'Vũ Đại Dương', '1990-11-05', '0987654321', 2),
(N'Hoàng Ngọc Hoa', '1993-04-18', '0977889900', 4),
(N'Đinh Quang Hải', '2001-08-30', '0922334455', 1),
(N'Bùi Thị Mai', '1988-12-12', '0944556677', 5),
(N'Ngô Tiến Đạt', '1999-06-25', '0966778899', 3),
(N'Lý Hải Yến', '2003-01-09', '0911223344', 4);

-- 2. Thêm 15 Sản phẩm
INSERT INTO [DienThoai] ([MaDT], [TenDT], [HangSX], [GiaBan], [SoLuongTon])
VALUES 
(101, N'iPhone 15 Pro Max', 'Apple', 29990000, 50),
(102, N'iPhone 15 Plus', 'Apple', 23500000, 30),
(103, N'iPhone 14', 'Apple', 17500000, 45),
(104, N'iPhone 13', 'Apple', 13990000, 100),
(105, N'Galaxy S24 Ultra', 'Samsung', 33990000, 20),
(106, N'Galaxy S23 FE', 'Samsung', 13500000, 60),
(107, N'Galaxy A54 5G', 'Samsung', 9500000, 120),
(108, N'Galaxy Z Fold 5', 'Samsung', 38990000, 15),
(109, N'Xiaomi 14 5G', 'Xiaomi', 22990000, 40),
(110, N'Redmi Note 13 Pro', 'Xiaomi', 7500000, 0), -- Tồn kho = 0
(111, N'Poco X6 Pro', 'Xiaomi', 8990000, 80),
(112, N'Oppo Reno 11 5G', 'Oppo', 10990000, 55),
(113, N'Oppo Find N3 Flip', 'Oppo', 22990000, 25),
(114, N'Oppo A79 5G', 'Oppo', 7490000, 0),    -- Tồn kho = 0
(115, N'Vivo V29 5G', 'Vivo', 12990000, 35);

-- 3. Thêm Giao dịch Hóa đơn
INSERT INTO [HoaDon] ([MaKhachHang], [MaDT], [NgayMua], [SoLuongMua])
VALUES 
(1, 101, '2026-01-15', 1),
(2, 105, '2026-01-20', 1),
(3, 107, '2026-02-05', 2),
(1, 115, '2026-02-14', 1), -- Khách số 1 mua lần 2
(4, 102, '2026-02-18', 1),
(5, 109, '2026-02-20', 3),
(6, 111, '2026-03-01', 1),
(7, 103, '2026-03-08', 2),
(4, 108, '2026-03-15', 1), 
(8, 104, '2026-03-22', 5), 
(1, 113, '2026-04-10', 1); -- Khách số 1 mua lần 3
```
_Hình 1.4 Chèn dữ liệu mẫu:_
<img width="1919" height="1079" alt="Ảnh chụp màn hình 2026-04-27 130524" src="https://github.com/user-attachments/assets/662b4894-00bf-4810-ab59-b81944537f4f" />

## PHẦN 2: XÂY DỰNG FUNCTION

### 2.1. Scalar Function (Tính tổng tiền hóa đơn)
**Mục đích:** Đóng gói công thức tính tiền (`Số lượng * Đơn giá`) thành một hàm duy nhất. Sau này khi viết báo cáo doanh thu, chỉ cần gọi hàm này thay vì phải viết lại câu lệnh JOIN rườm rà.

```sql
CREATE FUNCTION fn_TinhTienHoaDon (@MaHD INT)
RETURNS MONEY
AS
BEGIN
    DECLARE @TongTien MONEY;
    SELECT @TongTien = HD.SoLuongMua * DT.GiaBan
    FROM [HoaDon] HD JOIN [DienThoai] DT ON HD.MaDT = DT.MaDT
    WHERE HD.MaHD = @MaHD;
    RETURN ISNULL(@TongTien, 0);
END;
```
_Hình 2.1 Test Scalar Function:_
<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/6001eb49-c65b-4254-bc89-76c7b52efddf" />


_Kết quả:_
<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/03c68bd9-4d89-4008-b97e-e4ac56ce4124" />


### 2.2. Inline Table-Valued Function (Lọc sản phẩm theo hãng)
**Mục đích:** Giúp nhân viên tra cứu nhanh các sản phẩm của một hãng (ví dụ 'Samsung') mà hiện tại cửa hàng **vẫn còn hàng trong kho** (`SoLuongTon > 0`). Hàm Inline có tốc độ thực thi rất nhanh vì nó tương đương với một View có tham số.

```sql
CREATE FUNCTION fn_TimDienThoai_ConHang (@Hang NVARCHAR(50))
RETURNS TABLE
AS
RETURN (
    SELECT * FROM [DienThoai] 
    WHERE [HangSX] = @Hang AND [SoLuongTon] > 0
);
```
_Hình 2.2 Test Inline Function:_
<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/ae3287f7-b755-4927-9ba9-2d6faef8b3b0" />


_Kết quả:_
<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/3baab270-eef9-468b-baab-60c0645fac01" />


## PHẦN 3: XÂY DỰNG STORE PROCEDURE

### 3.1. SP Mua hàng (Kiểm tra tồn kho trước khi bán)
Trong thực tế, hệ thống không thể cho phép bán số lượng máy nhiều hơn số lượng máy đang có trong kho. Store Procedure này sử dụng lệnh `IF EXISTS` để chặn đứng giao dịch nếu phát hiện số lượng mua vượt quá số lượng tồn, giúp bảo vệ tính đúng đắn của dữ liệu kho hàng.

```sql
CREATE PROCEDURE sp_MuaDienThoai
    @MaKH INT, @MaDT INT, @SoLuong INT
AS
BEGIN
    IF EXISTS (SELECT 1 FROM [DienThoai] WHERE MaDT = @MaDT AND SoLuongTon < @SoLuong)
        PRINT N'Lỗi: Giao dịch bị hủy! Số lượng tồn kho không đủ để đáp ứng.';
    ELSE
    BEGIN
        INSERT INTO [HoaDon](MaKhachHang, MaDT, SoLuongMua) VALUES (@MaKH, @MaDT, @SoLuong);
        PRINT N'Giao dịch thành công! Đã tạo hóa đơn mới.';
    END
END;
```
_Hình 3.1 Test SP báo lỗi bán hàng:_
<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/d63b87a1-a831-455b-919b-3ef8bd318f50" />


_Kết quả:_
<img width="1919" height="1079" alt="Ảnh chụp màn hình 2026-04-27 131146" src="https://github.com/user-attachments/assets/c601c5cb-dab3-4994-b3d3-47e5c110e943" />



## PHẦN 4: TRIGGER TỰ ĐỘNG HÓA

### 4.1. Trigger tự động trừ tồn kho khi xuất hóa đơn
 Trigger này được gắn vào sự kiện `AFTER INSERT` của bảng `HoaDon`. Khi SP ở phần 3 thực hiện Insert thành công, Trigger sẽ lập tức được kích hoạt. Nó lấy số lượng vừa mua (nằm trong bảng ảo `inserted` của SQL Server) để trừ thẳng vào cột `SoLuongTon` của bảng `DienThoai`. Điều này giúp nhân viên không phải tự tay cập nhật lại kho sau mỗi lần bán hàng.

```sql
CREATE TRIGGER trg_CapNhatTonKho
ON [HoaDon]
AFTER INSERT
AS
BEGIN
    UPDATE [DienThoai]
    SET [SoLuongTon] = [SoLuongTon] - inserted.SoLuongMua
    FROM [DienThoai] JOIN inserted ON [DienThoai].MaDT = inserted.MaDT;
END;
```
_Hình 3.2 Test SP đếm số lần mua:_
<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/a0c92462-21ef-4714-9d82-cec14e54850d" />


_Kết quả:_
<img width="1919" height="1072" alt="image" src="https://github.com/user-attachments/assets/e1e7f8c2-7e06-4e69-83ce-ce5493b32ab7" />



## PHẦN 5: CURSOR VÀ TỐI ƯU HÓA TRUY VẤN

### 5.1. Sử dụng Cursor để tăng giá sản phẩm
**Bối cảnh:** Giả sử cửa hàng có chính sách tăng giá 5% cho tất cả sản phẩm của hãng Apple. Dưới đây là cách giải quyết bằng con trỏ (CURSOR) - duyệt qua từng dòng dữ liệu để Update.

```sql
DECLARE @Ma INT, @Gia MONEY;
DECLARE cur_TangGia CURSOR FOR SELECT MaDT, GiaBan FROM [DienThoai] WHERE HangSX = 'Apple';

OPEN cur_TangGia;
FETCH NEXT FROM cur_TangGia INTO @Ma, @Gia;

-- Vòng lặp duyệt từng dòng dữ liệu
WHILE @@FETCH_STATUS = 0
BEGIN
    UPDATE [DienThoai] SET GiaBan = @Gia * 1.05 WHERE MaDT = @Ma;
    FETCH NEXT FROM cur_TangGia INTO @Ma, @Gia;
END;

CLOSE cur_TangGia;
DEALLOCATE cur_TangGia;
```

<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/a78eb9b0-1970-459e-9b33-e332ba774c87" />



_Kết quả:_
<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/98d5a13e-4d75-43ca-ac0d-b1c182300d91" />




