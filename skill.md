# 🎓 Kỹ năng nhân bản & Triển khai nhanh dự án Phòng khám AI (Google Sheets + Apps Script + Vercel)

Tài liệu skill này được tóm tắt rút gọn để anh tham khảo nhanh khi cần nhân bản hoặc triển khai một dự án phòng khám tương tự mà không mắc lỗi, tiết kiệm 90% thời gian nghiên cứu lại từ đầu.

---

## 🛠️ Quy trình 4 bước Triển khai nhanh

### 📊 BƯỚC 1: Sao chép trang tính & Thiết lập vùng miền
1. Truy cập [Google Sheet Mẫu](https://docs.google.com/spreadsheets/d/1rJ5uWlXauy7M8r-VsQ70jujhrsR_chjKKIJC0jzLGs4/edit?usp=sharing) -> **Tệp (File)** -> **Tạo bản sao (Make a copy)**.
2. Kiểm tra cài đặt quốc gia (BẮT BUỘC):
   - Vào **Tệp (File)** -> **Cài đặt (Settings)** -> kiểm tra mục **Quốc gia (Locale)**.
   - Nếu quốc gia là **Việt Nam**: Múi giờ đặt là `Asia/Ho_Chi_Minh`. Lưu ý các công thức trên bảng tính sẽ dùng dấu chấm phẩy (`;`) thay vì dấu phẩy (`,`) để phân tách đối số.

---

### ⚙️ BƯỚC 2: Cấu hình Apps Script Backend
1. Trên trang tính mới, chọn **Tiện ích mở rộng** -> **Apps Script**.
2. **Khai báo `appsscript.json` (Manifest):**
   - Nhấp vào biểu tượng Bánh răng (Cài đặt dự án) -> tích chọn **"Hiển thị file appsscript.json trong trình chỉnh sửa"**.
   - Quay lại Editor, chọn file `appsscript.json`, dán nội dung sau:
   ```json
   {
     "timeZone": "Asia/Ho_Chi_Minh",
     "dependencies": {
       "enabledAdvancedServices": [
         {
           "userSymbol": "Sheets",
           "serviceId": "sheets",
           "version": "v4"
         }
       ]
     },
     "webapp": {
       "executeAs": "USER_DEPLOYING",
       "access": "ANYONE_ANONYMOUS"
     },
     "exceptionLogging": "STACKDRIVER",
     "runtimeVersion": "V8",
     "oauthScopes": [
       "https://www.googleapis.com/auth/spreadsheets",
       "https://www.googleapis.com/auth/script.external_request"
     ]
   }
   ```
3. **Dán mã nguồn `Code.gs`:**
   - Mở file `Code.gs`, dán toàn bộ code từ file [Code.gs gốc](file:///d:/phongkham/Code.gs).
   - Đổi giá trị `SPREADSHEET_ID` ở đầu file thành ID của trang tính mới.
4. **Chạy dọn dẹp & Triển khai:**
   - Chọn hàm `fixBrokenFormulas` trên thanh menu và nhấn **Chạy (Run)**, sau đó nhấn cấp quyền truy cập đầy đủ nếu Google yêu cầu.
   - Nhấn **Triển khai (Deploy)** -> **Triển khai mới (New deployment)** -> Chọn **Ứng dụng web (Web App)**.
   - Chọn quyền truy cập: **Mọi người (Anyone)** và thực thi dưới dạng **Tôi**.
   - **Copy lại URL Web App** sau khi triển khai thành công.

---

### 🤖 BƯỚC 3: Điền API Key DeepSeek cho Chatbot
1. Mở trang tính, vào sheet **"Cài đặt"**.
2. Điền chính xác các cấu hình sau:
   - **`ai_api_key`**: `sk-a5a64a3ceb5e438bb318ab004af2457a` (hoặc API Key DeepSeek mới của anh).
   - **`ai_base_url`**: `https://api.deepseek.com/v1`
   - **`ai_model`**: `deepseek-chat`

---

### 🌐 BƯỚC 4: Kết nối Frontend Vercel
1. Mở file `index.html` của mã nguồn Frontend.
2. Tìm biến `API_URL` (khoảng dòng 3927) và thay bằng **URL Web App Apps Script** vừa copy ở Bước 2.
   ```javascript
   const API_URL = "URL_APPS_SCRIPT_CỦA_BẠN";
   ```
3. Sử dụng terminal hoặc GitHub để đẩy mã nguồn Frontend lên và deploy trên Vercel.
   - *Mẹo deploy nhanh qua terminal:* Chạy lệnh `npx vercel --prod --yes` tại thư mục chứa file `index.html`.

---

## ⚠️ 5 Lỗi Chí Mạng & Cách Khắc Phục (Kinh nghiệm thực tế)

### 1. Lỗi công thức Google Sheets tiếng Việt (`#ERROR!`)
* **Nguyên nhân:** Apps Script sử dụng cấu trúc dấu phẩy `,` mặc định của chuẩn US khi gán công thức tự động, gây lỗi cú pháp trên trang tính Việt Nam (dùng dấu chấm phẩy `;`).
* **Giải pháp:** Luôn sử dụng `.setFormula(formula)` với chuỗi công thức dùng dấu chấm phẩy `;` thay vì dấu phẩy `,` (ví dụ: `=SUMIFS(J:J; B:B; A2)`).

### 2. Lỗi lệch ngày hiển thị trên Timeline lịch hẹn
* **Nguyên nhân:** Khi đọc dữ liệu từ Sheets dạng ngày tháng, múi giờ mặc định chuyển đổi ngày sang UTC gây lệch lùi lại 1 ngày.
* **Giải pháp:** Sử dụng `Utilities.formatDate` đồng bộ múi giờ của script để chuyển đổi:
  ```javascript
  Utilities.formatDate(val, Session.getScriptTimeZone(), "yyyy-MM-dd")
  ```

### 3. Lỗi ô chứa Giờ bị biến mất hoặc đổi thành ngày 1899-12-30
* **Nguyên nhân:** Google Sheet lưu trữ các ô có định dạng chỉ chứa Giờ (ví dụ `08:00`) dưới dạng ngày mặc định năm 1899.
* **Giải pháp:** Phân biệt trong `parseSheetData` của Apps Script: nếu năm là `1899`, hãy định dạng nó thành `HH:mm`.
  ```javascript
  if (val.getFullYear() === 1899) {
    val = Utilities.formatDate(val, Session.getScriptTimeZone(), "HH:mm");
  }
  ```

### 4. Lỗi chọn khách hàng xong nhưng không hiển thị tên (trắng tinh)
* **Nguyên nhân:** Sử dụng quy tắc CSS của Dark Mode đặt màu chữ khi select là màu trắng (`#FFFFFF !important`) trong khi ô nhập liệu Light Mode có nền trắng.
* **Giải pháp:** Ghi đè CSS cho phần Light Mode để màu chữ hiển thị đúng màu tối:
  ```css
  .searchable-dropdown-input.selected {
    color: #1E2937 !important;
  }
  ```

### 5. Lỗi `hex.slice is not a function` hoặc `services.map is not a function`
* **Nguyên nhân:** (1) Trả về chuỗi lỗi như `#REF!` từ Sheets làm sập frontend; hoặc (2) Mảng dịch vụ lưu dạng chuỗi JSON thô trong Sheets khi lấy lên Frontend không được parse lại.
* **Giải pháp:**
  - Lọc bỏ chuỗi lỗi xuất phát từ dấu `#` ở backend `Code.gs` nhưng chừa lại các mã màu Hex.
  - Sử dụng hàm `parseAppointments()` ở Frontend để `JSON.parse` trường dịch vụ trước khi hiển thị.
