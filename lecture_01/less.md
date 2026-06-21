# Lệnh `less` - Trình Phân Trang (Terminal Pager)

### 1. Cơ sở lý thuyết và chức năng cốt lõi
`less` là một tiện ích dòng lệnh thuộc nhóm "pager". Chức năng của nó là cho phép người dùng xem (view) nội dung của các tập tin văn bản hoặc luồng dữ liệu chuẩn (`stdout`) theo từng trang màn hình.

Ưu điểm kỹ thuật cốt lõi của `less`: **Cơ chế tải dữ liệu lười (Lazy Loading)**.
Trái ngược với lệnh `cat` (bơm liên tục toàn bộ dữ liệu ra luồng đầu ra) hoặc các trình soạn thảo văn bản như `vi`/`nano` (phải nạp toàn bộ kích thước tập tin vào RAM), `less` chỉ nạp vào bộ nhớ lượng dữ liệu tương ứng với khu vực đang hiển thị trên màn hình. Đặc tính này biến `less` trở thành công cụ tiêu chuẩn và an toàn tuyệt đối để truy xuất các tập tin dữ liệu cực lớn (hàng trăm Gigabyte như tập tin FASTQ, SAM, VCF) mà không gây rò rỉ tài nguyên hay quá tải bộ nhớ hệ thống (Out of Memory - OOM).

### 2. Cú pháp khởi tạo
```bash
# Truy xuất trực tiếp tập tin vật lý
less large_data.vcf

# Tiếp nhận luồng dữ liệu thông qua đường ống (pipeline)
grep "missense_variant" annotations.vcf | less
```

### 3. Hệ thống phím điều hướng và thao tác dữ liệu (Navigation)
Môi trường tương tác của `less` phụ thuộc hoàn toàn vào hệ thống phím tắt (tương đồng với trình soạn thảo `vim`). Dưới đây là các thao tác kỹ thuật bắt buộc phải nắm vững:

**a. Điều hướng cục bộ (Local Navigation):**
- `Space` hoặc `f` (forward): Cuộn tiến 1 trang màn hình.
- `b` (backward): Cuộn lùi 1 trang màn hình.
- `j` hoặc `Down Arrow`: Cuộn tiến 1 dòng.
- `k` hoặc `Up Arrow`: Cuộn lùi 1 dòng.

**b. Điều hướng toàn cục (Global Navigation):**
- `g` (Go): Lập tức đưa con trỏ về dòng đầu tiên của tập tin.
- `G` (Shift + g): Lập tức đưa con trỏ đến dòng cuối cùng của tập tin.
- `[Số]g` (Ví dụ: `150g`): Di chuyển nhảy vọt chính xác đến dòng số 150.

**c. Tìm kiếm chuỗi ký tự (Pattern Matching):**
- `/chuỗi_cần_tìm`: Tìm kiếm từ khóa theo chiều thuận (từ trên xuống dưới).
- `?chuỗi_cần_tìm`: Tìm kiếm từ khóa theo chiều nghịch (từ dưới lên trên).
*Sau khi nhập truy vấn tìm kiếm:*
- `n` (next): Di chuyển đến kết quả khớp tiếp theo (cùng chiều với lệnh tìm kiếm ban đầu).
- `N` (Shift + n): Di chuyển đến kết quả khớp trước đó (ngược chiều với lệnh tìm kiếm ban đầu).

**d. Chấm dứt tiến trình:**
- `q` (quit): Đóng tiến trình `less`, giải phóng buffer và hoàn trả luồng điều khiển cho terminal.

### 4. Các tham số kỹ thuật thiết yếu (Options)
Trong phân tích Tin Sinh Học, cấu trúc dữ liệu chủ yếu ở định dạng ma trận (dạng bảng). Việc cấu hình lại hành vi hiển thị mặc định của `less` là bước tối quan trọng.

**a. Vô hiệu hóa tính năng ngắt dòng tự động (`-S` hoặc `--chop-long-lines`)**
Mặc định, nếu chiều dài của một dòng vượt quá kích thước chiều ngang của terminal, `less` sẽ tự động bẻ gãy dòng (text wrapping) đẩy phần thừa xuống hàng tiếp theo, làm vỡ hoàn toàn cấu trúc ma trận cột.
Tham số `-S` vô hiệu hóa cơ chế này. Toàn bộ bản ghi sẽ được duy trì trên một dòng duy nhất. Người dùng sử dụng phím Mũi tên Trái/Phải để thực hiện cuộn ngang (horizontal scroll).
```bash
less -S snp_database.vcf
```
*(Lưu ý: Tham số `-S` là quy chuẩn kỹ thuật bắt buộc phải sử dụng khi kiểm tra dữ liệu cấu trúc dạng cột như SAM, VCF, BED, GTF).*

**b. Kích hoạt định danh tọa độ dòng (`-N` hoặc `--LINE-NUMBERS`)**
Tham số `-N` khởi tạo cột số thứ tự tương ứng cho từng dòng ở lề trái màn hình. Tính năng này đóng vai trò quan trọng trong việc đối chiếu tọa độ dữ liệu lỗi hoặc debug kịch bản (script) từ tập tin log.
```bash
less -N pipeline_execution.log
```

*(Mẹo vận hành: Khác với hầu hết các tiện ích khác, bạn hoàn toàn có thể kích hoạt hoặc vô hiệu hóa các tham số này theo thời gian thực (real-time) ngay bên trong môi trường `less`. Khi đang xem tập tin, gõ trực tiếp cụm ký tự `-S` rồi nhấn Enter, cơ chế chop-long-lines sẽ thay đổi trạng thái ngay lập tức mà không cần thoát lệnh).*
