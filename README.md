## Bài tập Hệ quản trị cơ sở dữ liệu (TEE560)

+ **Họ và tên:** Đào Tuấn Hưng
+ **Lớp:** K59KMT.K01
+ **Mã số sinh viên:** K235480106032
+ **Trường:** Đại học Kỹ thuật Công Nghiệp Thái Nguyên - TNUT
---

### Nhiệm vụ 1: Thiết kế CSDL 

1. Tạo Database
* Tên Database : QuanLyCamDo

<img width="1920" height="1080" alt="1  tạo database" src="https://github.com/user-attachments/assets/287a2699-5a81-4761-ba02-71074cc16863" />

2. Tạo ít nhất 4 bảng có quan hệ với nhau :

* Tạo bảng [HopDong].

<img width="1920" height="1080" alt="3  Bảng Hợp Đồng" src="https://github.com/user-attachments/assets/a2344d8b-95d8-44bc-93fa-22ce5d01973e" />

* Tạo bảng [KhachHang].

<img width="1920" height="1080" alt="4  Bảng Khách Hàng" src="https://github.com/user-attachments/assets/003b3d72-5df6-4612-a99b-a5f63a62e234" />

* Tạo bảng [TaiSan].

<img width="1920" height="1080" alt="5  Bảng Tài Sản" src="https://github.com/user-attachments/assets/c878d587-3328-43ab-83a3-1c97baadfed8" />

* Tạo bảng [TransactionLog].

<img width="1920" height="1080" alt="6  Bảng Bảng Log" src="https://github.com/user-attachments/assets/d2af023c-cd23-4e41-93bd-32326bfd1a3a" />

3. Kết Quả

4. Giải thích các thành phần

**Bảng Khách Hàng (KhachHang) - Quản lý thực thể định danh**

Đây là nơi lưu trữ thông tin cơ bản của người đến cầm đồ. Việc thiết kế bảng này tập trung vào tính duy nhất để quản lý rủi ro nợ xấu.
* MaKH: Sử dụng kiểu INT IDENTITY(1,1) làm Khóa chính. Hệ thống sẽ tự động cấp mã số tăng dần cho mỗi khách hàng mới.
* SDT & CCCD: Được đặt ràng buộc UNIQUE. Điều này cực kỳ quan trọng vì nó ngăn chặn việc một người dùng nhiều hồ sơ khác nhau để vay vượt hạn mức, giúp tiệm kiểm soát danh tính chính xác.
* HoTen, DiaChi: Lưu trữ thông tin liên lạc để phục vụ việc nhắc nợ hoặc gửi thông báo thanh lý tài sản sau này.

**Bảng Hợp Đồng (HopDong) - Trung tâm điều phối giao dịch**

Đây là bảng quan trọng nhất, lưu trữ các thỏa thuận tài chính và các mốc thời gian để hệ thống tự động tính lãi.
* MaHD: Khóa chính định danh cho mỗi lần giao dịch.
* MaKH: Khóa ngoại liên kết với bảng KhachHang, giúp biết hợp đồng này thuộc về ai.
* Các mốc thời gian (Deadline1, Deadline2):

   * Deadline1: Mốc kết thúc lãi đơn. Nếu quá ngày này, hệ thống sẽ tự động chuyển sang tính lãi kép thông qua các Trigger đã thiết lập.

   * Deadline2: Mốc bắt đầu quyền thanh lý. Nếu khách không đóng lãi/gốc, tiệm có quyền bán tài sản để thu hồi vốn.
  
* TrangThai: Được giới hạn bởi ràng buộc CHECK để đảm bảo dữ liệu luôn nhất quán (ví dụ: 'Đang vay', 'Quá hạn', 'Đã thanh toán').

**Bảng Tài Sản (TaiSan) - Quản lý hiện vật cầm cố**

Bảng này tách biệt với hợp đồng để hỗ trợ trường hợp một hợp đồng có thể cầm nhiều món đồ khác nhau.
* TenTaiSan, GiaTriDinhGia: Lưu trữ mô tả và giá trị thực tế của món đồ tại thời điểm cầm để làm căn cứ cho vay.
* TrangThaiTS: Theo dõi vòng đời của món đồ (Đang cầm cố -> Sẵn sàng thanh lý -> Đã bán).
* IsSold: Cờ đánh dấu kiểu BIT (0/1). Khi tài sản được bán thành công, cờ này bật lên 1 để loại món đồ ra khỏi danh sách quản lý kho hiện tại.

**Bảng Nhật Ký Giao Dịch (TransactionLog) - Minh bạch tài chính**

Đây là bảng ghi chép lại mọi biến động về dòng tiền trong suốt thời gian hợp đồng diễn ra.
* MaLog: Khóa chính tự tăng để theo dõi thứ tự các lần đóng tiền.
* SoTienTra: Lưu lại số tiền khách đóng mỗi lần. Dữ liệu này giúp SP sp_XuLyTraNo tính toán dư nợ còn lại chính xác.
* NgayGiaoDich: Mặc định lấy GETDATE() để ghi lại chính xác thời điểm khách đến đóng tiền, phục vụ việc đối soát tài chính cuối ngày cho chủ tiệm.

### Nhiệm vụ 2: Cài đặt SQL 

