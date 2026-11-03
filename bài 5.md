


# Bài 5: Tối ưu tiện ích mở rộng cho hệ thống điều phối vận tải


---

## Phần 1. Phân tích

### Vai trò của JSON Viewer trong gỡ lỗi API
Dữ liệu API thô từ máy chủ thường là **một chuỗi JSON dài, dính liền một dòng**, rất khó đọc. Ví dụ dữ liệu GPS thô:

```json
{"vehicle_id":"GL-102","status":"moving","location":{"lat":10.7769,"lng":106.7009},"speed":45,"updated_at":"2026-11-03T08:30:00Z"}
```

JSON Viewer biến chuỗi trên thành **cây có thụt lề, tô màu, thu gọn/mở rộng**:

```json
{
  "vehicle_id": "GL-102",
  "status": "moving",
  "location": { "lat": 10.7769, "lng": 106.7009 },
  "speed": 45,
  "updated_at": "2026-11-03T08:30:00Z"
}
```

| Vai trò | Lợi ích khi gỡ lỗi |
|---|---|
| Định dạng, thụt lề, tô màu | Đọc nhanh cấu trúc dữ liệu, phân biệt chuỗi, số, mảng |
| Thu gọn/mở rộng từng nhánh | Tập trung vào đúng trường cần kiểm tra (ví dụ `location`) |
| Phát hiện lỗi cú pháp | Thiếu dấu phẩy, ngoặc, sai kiểu dữ liệu được báo ngay |
| Tìm kiếm, sao chép đường dẫn trường | Tìm nhanh `lat`, `lng`, `speed` trong phản hồi lớn |
| Đối chiếu với tab Network | Kiểm tra phản hồi (Response) có đúng định dạng và giá trị mong đợi không |

---

## Phần 2. Triển khai

### Chức năng 1: 5 Extensions thiết yếu, chia 3 nhóm
| STT | Nhóm | Extension | Công dụng | Tải tài nguyên |
|---|---|---|---|---|
| 1 | **Dev Suite** | **JSON Viewer** (hoặc JSON Formatter) | Soi và định dạng dữ liệu API/GPS dạng JSON | Rất nhẹ |
| 2 | **Dev Suite** | **Wappalyzer** | Nhận diện công nghệ, framework web đang dùng, hỗ trợ chẩn đoán | Nhẹ |
| 3 | **Productivity** | **ColorZilla** | Chấm mã màu (eyedropper) lấy mã HEX/RGB của giao diện | Rất nhẹ |
| 4 | **Security** | **uBlock Origin** (trên Chrome có thể là *uBlock Origin Lite*) | Chặn quảng cáo, tracker độc hại gây chậm trang | Nhẹ |
| 5 | **Security** | **Privacy Badger** | Chặn tracker theo dõi bên thứ ba | Rất nhẹ |

**Nguyên tắc chọn:** mỗi nhóm có đại diện, ưu tiên tiện ích **nhẹ, uy tín, nhiều người dùng, cập nhật thường xuyên**, không chọn các tiện ích trùng chức năng để tiết kiệm RAM.

### Chức năng 2: Thiết lập Site Access (On click)
Mục đích: tiện ích **chỉ được đọc dữ liệu trang khi người dùng chủ động bấm vào**, hạn chế quyền đọc dữ liệu nhạy cảm (trang quản trị, API GPS).

1. Mở `chrome://extensions`.
2. Chọn **Details (Chi tiết)** của từng tiện ích.
3. Ở mục **Site access (Quyền truy cập trang web)**, chọn **On click (Khi nhấp)**.
4. Với tiện ích cần chạy tự động (ví dụ trình chặn quảng cáo), chọn **On specific sites** và chỉ thêm các trang cần thiết.
5. Hạn chế chọn **On all sites** (tất cả trang) vì tiện ích đọc được mọi dữ liệu.

| Extension | Site access đề xuất | Lý do |
|---|---|---|
| JSON Viewer | On click | Chỉ cần khi soi API |
| Wappalyzer | On click | Chỉ cần khi phân tích trang |
| ColorZilla | On click | Chỉ dùng khi lấy màu |
| uBlock Origin | On specific sites / On all sites | Cần chạy nền để chặn quảng cáo |
| Privacy Badger | On all sites | Cần chạy nền để chặn tracker |

### Ghim (Pin) lên Toolbar để thao tác nhanh
1. Bấm biểu tượng **mảnh ghép (Extensions)** trên thanh công cụ.
2. Bấm biểu tượng **ghim** bên cạnh tiện ích muốn ghim.
3. Chỉ nên ghim các tiện ích **dùng thường xuyên** (JSON Viewer, ColorZilla, uBlock Origin) để thanh công cụ gọn.
4. Muốn bỏ ghim, bấm lại biểu tượng ghim.

### Đáp ứng ràng buộc về RAM
- Chỉ cài **5 tiện ích**, mỗi chức năng một cái, không trùng lặp.
- Dùng chế độ **On click** để tiện ích không chạy ngầm trên mọi trang.
- Dùng **Profile riêng cho công việc kỹ thuật** (Work), không cài thêm tiện ích không cần thiết.
- Khi **phân tích biểu đồ Waterfall** (DevTools, tab Network), có thể **tắt tạm** tiện ích không cần để kết quả đo chính xác, hoặc dùng cửa sổ ẩn danh (tiện ích mặc định bị tắt).

---

## Phần 3. Kiểm thử: trình duyệt bị lag

### Quy trình kiểm tra mức tiêu thụ tài nguyên từng Extension

**Bước 1. Xem mức tiêu thụ bằng Task Manager của Chrome**
1. Nhấn **Shift + Esc** (hoặc menu ⋮, **Công cụ khác**, **Trình quản lý tác vụ**).
2. Tìm các dòng bắt đầu bằng **"Extension: ..."**.
3. Xem cột **Memory footprint** (RAM) và **CPU** của từng tiện ích.
4. Bấm vào tiêu đề cột để **sắp xếp** và tìm tiện ích ngốn tài nguyên nhất.

**Bước 2. Khoanh vùng tiện ích gây lag trên `chrome://extensions`**
1. Mở `chrome://extensions`.
2. **Tắt lần lượt** từng tiện ích bằng công tắc, sau mỗi lần thử tải lại trang và đo độ mượt.
3. Khi tắt tiện ích nào mà hết lag, đó là **thủ phạm**.

**Bước 3. Xử lý**
| Tình huống | Hướng xử lý |
|---|---|
| Tiện ích chiếm nhiều RAM/CPU | Đổi sang **On click**, hoặc thay bằng tiện ích nhẹ hơn |
| Tiện ích ít dùng | **Tắt** hoặc **gỡ** (Remove) |
| Hai tiện ích trùng chức năng | Giữ một cái, gỡ cái còn lại |
| Tiện ích không rõ nguồn gốc | Gỡ ngay và kiểm tra bảo mật |
| Vẫn còn lag | Cập nhật trình duyệt, tắt bớt tab, kiểm tra tiện ích trong DevTools |

**Bước 4. Kiểm tra lại**
Mở lại Task Manager (Shift + Esc) so sánh RAM trước và sau, đồng thời nạp lại biểu đồ **Waterfall** (F12, tab Network) để xác nhận trình duyệt đã mượt.


| Trước tối ưu | Sau tối ưu |
|---|---|
| Nhiều tiện ích chạy trên mọi trang, RAM cao, trình duyệt lag | Chỉ 5 tiện ích cần thiết, đa số ở chế độ On click, trình duyệt mượt khi nạp Waterfall |

---


