# Lệnh `nano` - Trình Soạn Thảo Văn Bản Dòng Lệnh

### 1. Tổng quan kỹ thuật
`nano` là một trình soạn thảo văn bản (text editor) hoạt động hoàn toàn trên giao diện dòng lệnh (CLI), được tích hợp mặc định trên hầu hết các bản phân phối Linux và macOS.

Khác với các trình soạn thảo phức tạp như `vim` hay `emacs` (đòi hỏi đường cong học tập rất cao), `nano` được thiết kế hướng đến sự tối giản và thân thiện. Giao diện của phần mềm luôn thường trực một dải danh sách các tổ hợp phím tắt (shortcuts) ở cạnh dưới màn hình, loại bỏ hoàn toàn gánh nặng phải ghi nhớ mã lệnh cho người mới tiếp cận.

### 2. Phạm vi ứng dụng
- Biên soạn, chỉnh sửa và cấu hình các kịch bản thực thi phân tích (Bash scripts - `.sh`).
- Cấu hình các tập tin biến môi trường hệ thống (Ví dụ: `.bashrc`, `.bash_profile`).
- Chỉnh sửa thủ công các tập tin metadata văn bản thuần túy nhỏ.

### 3. Cú pháp khởi động
```bash
# Khởi tạo tiến trình mở tập tin có sẵn, hoặc tạo tệp mới nếu chưa tồn tại
nano script_pipeline.sh
```

### 4. Hệ thống phím tắt điều khiển (Control Shortcuts)
Trong môi trường hiển thị của `nano`, ký hiệu dấu mũ `^` đại diện cho phím **`Ctrl`** trên bàn phím.

**a. Thao tác Lưu và Thoát:**
- **`Ctrl + O`** (Write Out): Thực thi lệnh lưu tập tin vào ổ cứng. (Hệ thống sẽ xác nhận lại tên tập tin, nhấn `Enter` để đồng ý ghi).
- **`Ctrl + X`** (Exit): Yêu cầu thoát tiến trình. (Nếu phát hiện thay đổi chưa được lưu, `nano` sẽ kích hoạt luồng cảnh báo: Bấm `Y` (Đồng ý lưu) hoặc `N` (Hủy bỏ thay đổi)).

**b. Thao tác Xử lý khối văn bản (Cắt/Dán):**
- **`Ctrl + K`** (Cut Text): Cắt (xóa tạm vào khay nhớ) toàn bộ dòng mã lệnh nơi con trỏ đang định vị.
- **`Ctrl + U`** (Uncut Text): Dán (Paste) dòng dữ liệu vừa được lưu trữ trong khay nhớ ra vị trí con trỏ hiện tại.

**c. Thao tác Tìm kiếm:**
- **`Ctrl + W`** (Where Is): Kích hoạt hộp thoại tìm kiếm. Nhập từ khóa và bấm `Enter`.
- Để nhảy đến kết quả khớp tiếp theo, tiếp tục ấn lại `Ctrl + W` và nhấn `Enter` (mà không cần gõ lại từ khóa).

### 5. Cảnh báo mức độ nghiêm trọng trong Tin Sinh Học
- **Tuyệt đối KHÔNG sử dụng trình soạn thảo (`nano`, `vim`)** để mở các tập tin dữ liệu hệ gen thô (FASTQ, BAM, VCF, GTF). Về mặt kiến trúc, trình soạn thảo sẽ cưỡng bức nạp toàn bộ cấu trúc tập tin vào vùng nhớ RAM. Với các tập tin lớn, hành động này sẽ dẫn đến sự cố sập bộ nhớ (Out of Memory) làm treo cứng máy chủ tính toán.
- **Quy chuẩn thực thi:** Dữ liệu lớn tuân thủ quy tắc chỉ đọc (read-only) bằng các công cụ Pager như `less` hoặc Pumper như `head`. Trình soạn thảo văn bản chỉ giới hạn cho tập tin mã nguồn (script) và tập tin cấu hình.
