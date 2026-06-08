# Hướng dẫn quy trình cài đặt & Triển khai Hệ thống Phòng khám AI

Tài liệu này ghi lại toàn bộ các bước từ khởi tạo, cấu hình Google Sheets, thiết lập Google Apps Script, kết nối AI DeepSeek, đến việc đẩy mã nguồn lên GitHub và deploy lên Vercel để anh có một website hoàn chỉnh và chuyên nghiệp.

---

## 📊 Bước 1: Chuẩn bị Google Sheet (Database)

1. **Sao chép Google Sheet mẫu:**
   Mở liên kết [Google Sheet Mẫu](https://docs.google.com/spreadsheets/d/1rJ5uWlXauy7M8r-VsQ70jujhrsR_chjKKIJC0jzLGs4/edit?usp=sharing) và chọn **Tệp (File)** -> **Tạo bản sao (Make a copy)** để lưu về Drive của anh.
2. **Kiểm tra định dạng vùng miền (Locale):**
   - Vào **Tệp (File)** -> **Cài đặt (Settings)** -> kiểm tra mục **Quốc gia (Locale)**.
   - Nếu quốc gia là **Việt Nam**:
     - Định dạng ngày tháng sẽ là `DD/MM/YYYY`.
     - Phân cách hàng nghìn bằng dấu chấm (`.`), phân cách thập phân bằng dấu phẩy (`,`).
     - **Cú pháp công thức trên trang tính bắt buộc phải dùng dấu chấm phẩy (`;`) làm dấu phân tách đối số** (ví dụ: `=SUMIFS(A:A; B:B; "x")` thay vì dấu phẩy `,`).
   - Dự án này đã được tối ưu hóa tự động để nhận diện dấu chấm phẩy (`;`) khi lưu dữ liệu từ Web về Google Sheets nhằm tránh lỗi `#ERROR!`.

---

## ⚙️ Bước 2: Cấu hình Google Apps Script (Backend)

1. Từ Google Sheet của anh, chọn **Tiện ích mở rộng (Extensions)** -> **Apps Script**.
2. **Cấu hình Manifest (`appsscript.json`):**
   - Trong giao diện Apps Script, chọn **Cài đặt dự án (Project Settings - biểu tượng bánh răng)**.
   - Tích chọn **Hiển thị file "appsscript.json" trong trình chỉnh sửa (Show "appsscript.json" manifest file in editor)**.
   - Quay lại phần **Trình chỉnh sửa (Editor - biểu tượng `<>`)**, mở file `appsscript.json` và thay thế toàn bộ nội dung bằng cấu hình dưới đây:
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
3. **Thêm mã nguồn `Code.gs`:**
   - Mở file `Code.gs` trong Apps Script, xóa hết code cũ.
   - Copy toàn bộ nội dung của file [Code.gs mới nhất](file:///d:/phongkham/Code.gs) trong thư mục này và dán vào.
   - Thay thế biến `SPREADSHEET_ID` ở dòng 4 thành ID bảng tính của anh (chuỗi ký tự nằm giữa `/d/` và `/edit` trên đường dẫn URL trang tính của anh).
   - Nhấn **Lưu (Save - biểu tượng đĩa mềm)**.

4. **Triển khai Web App (Deploy):**
   - Nhấn nút **Triển khai (Deploy)** ở góc trên bên phải -> chọn **Triển khai mới (New deployment)**.
   - Chọn loại cấu hình là **Ứng dụng web (Web app)**.
   - Cấu hình các mục:
     - **Mô tả (Description):** Phiên bản 1.0.
     - **Thực thi dưới dạng (Execute as):** Tôi (Tài khoản Google của anh - `USER_DEPLOYING`).
     - **Ai có quyền truy cập (Who has access):** Mọi người (Anyone - `ANYONE_ANONYMOUS`).
   - Nhấn **Triển khai (Deploy)**.
   - *Cấp quyền truy cập:* Nếu có hộp thoại xuất hiện, chọn **Xem quyền (Review Permissions)** -> Chọn tài khoản Google của anh -> **Nâng cao (Advanced)** -> **Đi tới dự án (Go to ... (unsafe))** -> **Cho phép (Allow)**.
   - **Lưu lại URL ứng dụng web** (đường dẫn này dùng để kết nối với Frontend trên Vercel).

---

## 🤖 Bước 3: Thiết lập AI Chatbot (DeepSeek)

Để kích hoạt tính năng Chatbot AI thông minh trên website:
1. Quay lại Google Sheet của anh, chuyển sang tab **"Cài đặt"**.
2. Điền các thông số cấu hình DeepSeek vào các dòng tương ứng:
   - Dòng chứa key `ai_api_key`: Điền API key của anh: `sk-a5a64a3ceb5e438bb318ab004af2457a`
   - Dòng chứa key `ai_base_url`: Điền URL API DeepSeek: `https://api.deepseek.com/v1`
   - Dòng chứa key `ai_model`: Điền model sử dụng: `deepseek-chat`
3. Quay lại Apps Script, chọn hàm `fixBrokenFormulas` hoặc `seedSampleData` từ thanh menu thả xuống và nhấn **Chạy (Run)** để khởi tạo hoặc sửa lỗi công thức cũ nếu có.

---

## 🌐 Bước 4: Triển khai Frontend lên Vercel & Github

Để website chạy độc lập chuyên nghiệp (như đường dẫn `https://website-phongkham.vercel.app/` thay vì chạy trong iframe của Google):

### 1. Đồng bộ mã nguồn Frontend
- File giao diện chính là `Index.html` (ở thư mục dự án) đã được cấu hình gọi API Web App thông qua hàm `callServerFunction` (bằng `fetch` method).
- Hãy chắc chắn đã copy toàn bộ nội dung file này vào `website-phongkham/index.html`.
- Mở file `website-phongkham/index.html` tìm đến dòng 3927 và cập nhật biến `API_URL` thành URL Web App Google Apps Script anh vừa lấy được ở **Bước 2**.
  ```javascript
  const API_URL = "URL_APPS_SCRIPT_WEB_APP_CỦA_BẠN";
  ```

### 2. Đẩy mã nguồn lên GitHub
Mở terminal tại thư mục `website-phongkham` và chạy các lệnh:
```bash
# Khởi tạo Git nếu chưa có
git init

# Thêm tất cả file
git add .

# Commit thay đổi
git commit -m "Triển khai phiên bản quản lý phòng khám mới với chatbot DeepSeek"

# Liên kết với repo GitHub của anh (nếu chưa có)
git remote add origin https://github.com/quocminhai-hub/website-phongkham.git

# Đẩy mã nguồn lên nhánh chính
git push -u origin main -f
```

### 3. Deploy lên Vercel
1. Truy cập vào [Vercel Dashboard](https://vercel.com).
2. Nhấn **Add New** -> **Project**.
3. Chọn kho lưu trữ **`website-phongkham`** từ danh sách GitHub của anh.
4. Giữ nguyên các cấu hình mặc định (Framework Preset: *Other*, Build Command: để trống).
5. Nhấn **Deploy**. Vercel sẽ tự động tạo cho anh một website hoàn chỉnh với tên miền cực kỳ chuyên nghiệp (dạng `https://tên-dự-án.vercel.app`).

---

## 🛠️ Một số kinh nghiệm phòng tránh lỗi cho tương lai
* **Tuyệt đối không dùng `google.script.run`** khi chạy web độc lập trên Vercel, vì đối tượng `google` chỉ tồn tại khi nhúng web trực tiếp bên trong Google Sheets. Mọi cuộc gọi backend bắt buộc phải thông qua `fetch` tới `doPost(e)` của Apps Script.
* **Lưu ý dấu phân cách đối số trong công thức:** Khi viết code Apps Script tự động thêm công thức vào Google Sheets, luôn dùng `.setFormula(formula)` và kiểm tra ngôn ngữ sheet. Nếu là Tiếng Việt, hãy viết chuỗi công thức dùng dấu chấm phẩy (`;`) thay vì dấu phẩy (`,`).
* **Mỗi lần sửa file Code.gs** trên Apps Script, bắt buộc phải chọn **Triển khai mới (New deployment)** thành một phiên bản mới thì mã nguồn trên Vercel mới nhận được tính năng mới cập nhật.

---

## 📋 Checklist Hoàn Thiện & Nghiệm Thu Dự Án Phòng Khám
Anh hãy kiểm tra kỹ các đầu mục sau đây để đảm bảo dự án hoạt động 100% không phát sinh lỗi:

### 1. Cấu hình Cơ sở dữ liệu (Google Sheets)
- [ ] Bảng tính đã cài đặt múi giờ `Asia/Ho_Chi_Minh` và quốc gia `Việt Nam` (trong *Tệp -> Cài đặt*).
- [ ] Tên các sheet khớp chính xác: `Bác sĩ`, `Đặt lịch`, `Khách hàng`, `Dịch vụ`, `Users`, `Cài đặt`.
- [ ] Tiêu đề cột `F1` của sheet *Khách hàng* hiển thị chữ `"Doanh thu"` (không bị lỗi `#REF!`).

### 2. Cấu hình Backend (Apps Script)
- [ ] Manifest `appsscript.json` đã được bật dịch vụ `Sheets v4` và khai báo các `oauthScopes`.
- [ ] ID bảng tính trong `Code.gs` (biến `SPREADSHEET_ID`) trỏ chính xác tới Sheet hiện tại.
- [ ] Hàm `fixBrokenFormulas` đã chạy thành công ít nhất một lần để dọn dẹp các lỗi công thức cũ.
- [ ] Web App đã được Deploy dưới dạng **Triển khai mới** (phiên bản mới nhất), chế độ thực thi: **Tôi** (`USER_DEPLOYING`), quyền truy cập: **Mọi người** (`ANYONE_ANONYMOUS`).

### 3. Cấu hình AI Chatbot
- [ ] API Key của DeepSeek đã điền chính xác vào cột giá trị của key `ai_api_key` trong sheet *Cài đặt*.
- [ ] `ai_base_url` điền là `https://api.deepseek.com/v1` và `ai_model` điền là `deepseek-chat`.

### 4. Giao diện Website (Vercel Frontend)
- [ ] Đường dẫn Web App Apps Script đã được điền chính xác vào biến `API_URL` của file `index.html`.
- [ ] Trạng thái dự án trên Vercel báo **Ready** và chạy đúng tên miền `.vercel.app`.

### 5. Nghiệm thu Tính năng chính
- [ ] Đăng nhập thành công bằng tài khoản trong sheet *Users*.
- [ ] Thêm lịch hẹn mới từ form giao diện web thành công, bảng tính cập nhật tức thì.
- [ ] Ô tìm kiếm/chọn Khách hàng và Bác sĩ hiển thị rõ tên đã chọn (chữ không bị trắng mờ trên nền trắng của Light Mode).
- [ ] Lịch hẹn hiển thị đúng ngày giờ trên thanh Timeline, không bị lệch ngày do lệch múi giờ.
- [ ] Chatbot AI phản hồi trơn tru, hiển thị khung xác nhận đặt lịch hẹn/thêm khách hàng.
- [ ] Khi bấm nút *"Đặt lịch"* từ khung xác nhận trong chatbot, hệ thống lưu lịch thành công và không báo lỗi `hex.slice is not a function`.
- [ ] Số liệu Doanh thu và Lần khám gần nhất của Khách hàng/Bác sĩ tự động tính toán đúng và hiển thị số tiền mà không có lỗi công thức `#ERROR!`.

