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

1. Event 1: Đăng ký hợp đồng mới (Vay tiền)

* Phân tích Logic xử lý
  * Khách hàng: Kiểm tra theo Số điện thoại hoặc CCCD. Nếu chưa có thì thêm mới, nếu có rồi thì lấy MaKH cũ.

  * Kỳ hạn vay: Đề bài yêu cầu nhập kỳ hạn. Mình sẽ quy ước kỳ hạn là số ngày.

    * Deadline 1 = Ngày lập + Kỳ hạn (Hết hạn lãi đơn).

    * Deadline 2 = Deadline 1 + Kỳ hạn (Hết hạn chờ, chuyển sang thanh lý).

  * Tài sản: Vì SQL Server khó truyền "danh sách" trực tiếp, mình sẽ chia làm 2 bước: 1 SP tạo Hợp đồng và 1 SP thêm Tài sản vào hợp đồng đó.

* Sript Cài đặt Stored Procedure

<img width="1920" height="1080" alt="1  Script Cài đặt Stored Procedure" src="https://github.com/user-attachments/assets/c5545c3d-0e8a-446c-a20c-6f311f0fd859" />

* Chay thử nghiệm

  * Bước 1: Đăng ký hợp đồng

 <img width="1920" height="1080" alt="2  Bước 1 Đăng ký hợp đồng" src="https://github.com/user-attachments/assets/085ea3cd-8ac5-442f-ae80-5b858efa2a30" />

  * Bước 2: Thêm tài sản cho hợp đồng số 1
 
<img width="1920" height="1080" alt="3  Bước 2 Thêm tài sản cho hợp đồng số 1" src="https://github.com/user-attachments/assets/c3c1830b-bd81-42ec-8098-f85cfd4c6743" />

* Kết quả

<img width="1920" height="1080" alt="4  Chạy thử nghiệm (Demo)" src="https://github.com/user-attachments/assets/e161901c-e55a-4740-9489-ef8aa514e180" />

* Giải Thích

  * Tại sao dùng SCOPE_IDENTITY()? Để lấy đúng cái ID tự tăng vừa mới tạo ra để dùng cho bảng tiếp theo.
 
  * Logic Deadline: Mình thiết kế rất linh hoạt, kỳ hạn 30 ngày thì sau 30 ngày tính lãi kép, sau 60 ngày thì thanh lý. Đúng như đề bài yêu cầu.

  * Tính chuẩn hóa: Việc tách Khách hàng và Tài sản giúp bạn không bị trùng lặp dữ liệu nếu khách đó quay lại vay lần 2
 
2. Event 2: Tính toán công nợ thời gian thực 

* Function tính tổng nợ của một Hợp đồng (fn_CalcMoneyContract)

  * Dựa trên quy tắc:

    * Lãi đơn: 0.5% (5.000đ/1.000.000đ) mỗi ngày trên gốc ban đầu.

    * Lãi kép: Sau mốc Deadline 1, lãi sẽ được tính trên tổng (Gốc + Lãi đơn đã tích lũy tính đến ngày Deadline 1).
   
* Hàm này sẽ tính tổng số tiền (Gốc + Lãi) mà khách phải trả tại một thời điểm TargetDate bất kỳ.

<img width="1920" height="1080" alt="1  Function tính tổng nợ của một Hợp đồng" src="https://github.com/user-attachments/assets/597c5a3c-eff4-4dd3-befc-bda64fbc38c7" />

* Function tính tiền của một Transaction (fn_CalcMoneyTransaction)

Hàm này thường dùng để xem trạng thái nợ tại một bản ghi Log cụ thể. Ở đây mình sẽ thiết kế theo hướng: Dựa vào MaLog, lấy MaHD tương ứng và tính nợ tại ngày giao dịch đó.

<img width="1920" height="1080" alt="2  Function tính tiền của một Transaction" src="https://github.com/user-attachments/assets/fab086e9-e60b-4dda-bb79-479af808596e" />

* Giải thích logic thuật toán

Toán học lãi kép: Mình sử dụng hàm POWER(cơ số, số mũ) của SQL Server.

$Tổng\_nợ = Gốc \times (1 + 0.5\%)^{Số\_ngày\_quá\_hạn}$

* Điểm chốt (Deadline 1): Đây là thời điểm "nhập lãi vào gốc". Toàn bộ lãi đơn thu được trong kỳ hạn đầu sẽ trở thành gốc để sinh lãi cho kỳ sau.

* Tính chính xác: Sử dụng kiểu dữ liệu FLOAT trong tính toán trung gian (hàm POWER) để tránh mất mát số dư, sau đó mới ép kiểu về MONEY để hiển thị đúng định dạng tiền tệ.

* Chạy thử nghiệm

<img width="1920" height="1080" alt="3  chạy thử nghiệm (Test)" src="https://github.com/user-attachments/assets/59be1961-8176-41ac-9f4a-78155dc6a045" />

3. Event 3: Xử lý trả nợ và hoàn trả tài sản

* Cài đặt Stored Procedure sp_XuLyTraNo

<img width="1920" height="1080" alt="1  Cài đặt Stored Procedure sp_XuLyTraNo" src="https://github.com/user-attachments/assets/b3cd5a96-2ace-4650-bb32-8c7164128dce" />

* Giải thích chi tiết thuật toán

* Tính toàn vẹn dữ liệu: Hệ thống kiểm tra cờ IsSold trước tiên. Nếu đồ đã bán (thanh lý xong), nhân viên không được phép thu thêm bất kỳ khoản tiền nào từ khách để tránh tranh chấp pháp lý.

* Logic Trả góp: Khi khách trả một phần tiền, trạng thái tự động chuyển từ "Đang vay" hoặc "Quá hạn" sang "Đang trả góp". Điều này giúp chủ tiệm phân loại được những khách hàng có thiện chí trả nợ.

* Thuật toán gợi ý trả đồ: * Hệ thống tính tổng giá trị của tất cả món đồ đang giữ của khách đó.

  * Sau đó, giả định nếu trả món đồ A cho khách, hệ thống tính toán xem: (Tổng giá trị cũ - Giá trị món A) có còn lớn hơn số tiền khách đang nợ hay không.

  * Nếu có, hệ thống gắn nhãn "CÓ THỂ TRẢ", giúp nhân viên quyết định trả lại đồ cho khách mà vẫn đảm bảo cửa hàng không bị rủi ro (mất vốn).
 
* Chạy thử nghiệm

<img width="1920" height="1080" alt="2  chạy thử nghiệm (Demo)" src="https://github.com/user-attachments/assets/e06addaf-d8c6-4ffa-9082-5dc54cec02ec" />

Khi chạy, Kết quả sẽ thấy ở cửa sổ kết quả hiện ra danh sách các món đồ. Món nào có chữ "CÓ THỂ TRẢ" thì có thể báo khách lấy về.

4. Event 4: Truy vấn danh sách nợ xấu (Nợ khó đòi)

* Script truy vấn danh sách nợ xấu

Tạo một View (để dùng như một bảng báo cáo động) hoặc một Stored Procedure. Ở đây mình sẽ viết một câu lệnh SELECT chi tiết, bạn có thể chạy nó bất cứ khi nào cần báo cáo

<img width="1920" height="1080" alt="1  Script truy vấn danh sách nợ xấu" src="https://github.com/user-attachments/assets/4292c541-41e6-4103-b605-c1173f0b041a" />

* Giải thích các thành phần

  * Số ngày quá hạn: Sử dụng hàm DATEDIFF(DAY, hd.Deadline1, GETDATE()). Con số này càng lớn thì tiền lãi kép càng tăng khủng khiếp.

  * Tổng nợ hiện tại: Gọi hàm fn_CalcMoneyContract với tham số là GETDATE() (ngày hôm nay). Hàm này tự động nhận diện đã qua Deadline 1 nên sẽ tính toán theo công thức lãi kép.

  * Tổng nợ sau 1 tháng: Đây là phần "dự báo". Chúng ta sử dụng hàm DATEADD(MONTH, 1, GETDATE()) để truyền vào hàm tính tiền. Kết quả trả về sẽ giúp chủ tiệm thấy được sự chênh lệch (lãi sinh ra) trong 30 ngày tới.

  * Hàm FORMAT(..., 'N0'): Giúp định dạng con số tiền tệ có dấu phẩy ngăn cách hàng nghìn (ví dụ: 10,000,000) để báo cáo dễ đọc hơn.
 
* Kiểm tra dữ liệu (Sample Data)

<img width="1920" height="1080" alt="2  kiểm tra dữ liệu (Sample Data)" src="https://github.com/user-attachments/assets/190da3aa-0577-4688-9c66-136b3ee94faf" />

5. Event 5: Quản lý thanh lý tài sản

* Trigger tự động chuyển sang "Quá hạn (nợ xấu)"

Trigger này sẽ chạy mỗi khi bảng HopDong được cập nhật hoặc chèn mới. Nếu ngày hiện tại vượt quá Deadline 1, trạng thái sẽ tự đổi.
 
<img width="1950" height="1080" alt="1  Trigger tự động chuyển sang Quá hạn" src="https://github.com/user-attachments/assets/1d2873d7-8b8b-465b-b6e9-e2ef3d21ee1f" />

* Trigger tự động chuyển tài sản sang "Sẵn sàng thanh lý"

Khi trạng thái hợp đồng đã là "Quá hạn (nợ xấu)" và thời gian vượt quá Deadline 2, tài sản liên quan sẽ tự động chuyển sang trạng thái chờ bán.

<img width="1920" height="1080" alt="2  Trigger tự động chuyển tài sản sang Sẵn sàng thanh lý" src="https://github.com/user-attachments/assets/6cc8ed64-7a1f-4a1a-bc5e-e10311c9f770" />

* Trigger tự động chuyển tài sản thành "Đã bán thanh lý"

Khi nhân viên xác nhận hợp đồng này "Đã thanh lý" (nghĩa là tiệm đã bán xong đồ để thu hồi vốn), Trigger sẽ tự động cập nhật cờ IsSold và trạng thái từng món đồ.

<img width="1920" height="1080" alt="3  Trigger tự động chuyển tài sản thành Đã bán thanh lý" src="https://github.com/user-attachments/assets/160a86a1-ae67-4042-adbf-a01b1b0daf7d" />

* Chạy Thử Nghiệm

  * Bước 1: Tạo dữ liệu mẫu (Khách hàng, Hợp đồng, Tài sản)

<img width="1920" height="1080" alt="4  Bước 1 Tạo dữ liệu mẫu (Khách hàng, Hợp đồng, Tài sản)" src="https://github.com/user-attachments/assets/1e81c5cb-13df-4d6f-8f96-5bc573db83fa" />

  * Bước 2: Kiểm tra Trigger Tự động chuyển Nợ xấu & Sẵn sàng thanh lý
  
<img width="1920" height="1080" alt="5  BƯỚC 2 Kiểm tra Trigger Tự động chuyển Nợ xấu   Sẵn sàng thanh lý" src="https://github.com/user-attachments/assets/0fbe3166-08e5-415a-9ef8-139bcd7a604a" />

  * Bước 3 Kiểm tra Trigger Xác nhận đã bán thanh lý

<img width="1920" height="1080" alt="6  BƯỚC 3 Kiểm tra Trigger Xác nhận đã bán thanh lý" src="https://github.com/user-attachments/assets/064ff600-c6c0-49cd-9eb9-5c8e1cb02fc5" />

* Giải Thích

  * Cơ chế "Đánh thức" (Triggering)
Thuật toán không chạy liên tục mà chỉ "thức dậy" khi có một hành động INSERT hoặc UPDATE tác động vào bảng HopDong.

    * Điều này giúp tiết kiệm tài nguyên hệ thống, chỉ xử lý khi dữ liệu có sự thay đổi.

  * Logic chuyển đổi tầng 1: Nợ xấu
Điều kiện: Ngày hiện tại (GETDATE()) vượt quá mốc Deadline1.

    * Hành động: Trigger trg_TuDongChuyenNoXau tự động đổi trạng thái hợp đồng thành 'Quá hạn'.

    * Mục đích: Chốt sổ lãi đơn và bắt đầu áp dụng công thức lãi kép cho giai đoạn tiếp theo.

  * Logic chuyển đổi tầng 2: Thanh lý tài sản
Điều kiện: Trạng thái hợp đồng là 'Quá hạn' và ngày hiện tại đã vượt quá mốc Deadline2.

    * Hành động: Trigger trg_TaiSanSanSangThanhLy tác động sang bảng TaiSan, chuyển TrangThaiTS thành 'Sẵn sàng thanh lý'.

    * Mục đích: Thông báo cho chủ tiệm rằng món đồ này đã đủ điều kiện pháp lý để đem bán thu hồi vốn.

  * Logic kết thúc: Đồng bộ hóa bán hàng
Điều kiện: Khi chủ tiệm xác nhận đã bán được tài sản bằng cách cập nhật trạng thái hợp đồng thành 'Đã thanh lý'.

    * Hành động: Trigger trg_XacNhanDaBanThanhLy tự động bật cờ IsSold = 1 cho tất cả tài sản liên quan.

    * Mục đích: Đảm bảo kho hàng luôn chính xác và không bán nhầm một món đồ đã bán rồi.
