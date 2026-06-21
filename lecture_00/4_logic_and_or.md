# Giải mã Toán tử Logic: `&&` (AND) và `||` (OR)

Đúng như sự nhạy bén của bạn, `&&` là viết tắt của **AND** (Và), còn `||` là viết tắt của **OR** (Hoặc). Đây là những ký hiệu phân luồng thông minh, được sinh ra để khắc phục sự "vô tâm" của dấu chấm phẩy `;`.

---

### 1. Bản chất cốt lõi: Mật mã "Exit Status"
Để hiểu được bản chất của `&&` và `||`, bạn bắt buộc phải biết một bí mật của nhân hệ điều hành Linux:
Mọi câu lệnh (như `ls`, `cd`, `grep`...) khi chạy xong và tắt đi, đều để lại một con số tàng hình gọi là **Mã trạng thái thoát (Exit Status)**.
- **Mã `0`**: Lệnh chạy thành công mỹ mãn.
- **Mã từ `1` đến `255`**: Có lỗi xảy ra (sai cú pháp, thiếu file, đầy ổ cứng...).

Các toán tử `&&` và `||` bản chất chính là **những trạm kiểm soát**. Chúng đứng chặn ở giữa để đọc cái mã tàng hình `0` hay `1` của lệnh đi trước, từ đó chúng mới đưa ra quyết định xem có cho phép lệnh tiếp theo chạy hay không.

---

### 2. Toán tử `&&` (Logical AND - Bước đệm Thành công)
**Quy luật:** `Lệnh_A && Lệnh_B`
- **Lệnh B** CHỈ ĐƯỢC PHÉP CHẠY nếu **Lệnh A** thành công (Mã trả về là `0`).
- Nếu Lệnh A báo lỗi, toàn bộ phần còn lại của chuỗi sẽ bị cắt đứt (hủy bỏ) lập tức.

**Ứng dụng thực chiến:** (Đây là nguyên tắc sống còn trong Tin Sinh)
Khi bạn chạy một công cụ Mapping nặng mất 5 tiếng đồng hồ, và bạn muốn ngay sau khi nó xong thì tự động nén file kết quả lại để đỡ tốn dung lượng ổ cứng.
```bash
# Nếu bwa chạy thất bại (ví dụ do sập RAM giữa chừng), lệnh gzip phía sau sẽ tự động bị hủy bỏ!
# Điều này vô cùng an toàn, giúp bạn không tạo ra các file nén rác bị hỏng hóc.
bwa mem ref.fa sample.fq > result.sam && gzip result.sam
```

---

### 3. Toán tử `||` (Logical OR - Túi khí Cứu hộ)
**Quy luật:** `Lệnh_A || Lệnh_B`
- Ngược lại hoàn toàn với AND. **Lệnh B** CHỈ ĐƯỢC PHÉP CHẠY nếu **Lệnh A** bị LỖI (Mã trả về khác `0`).
- Nếu Lệnh A thành công trót lọt, Lệnh B sẽ bị bỏ qua hoàn toàn.

**Ứng dụng thực chiến:**
Rất hay được dùng để làm phương án dự phòng (Fallback) hoặc in ra cảnh báo khi có biến cố.
```bash
# Thử tạo thư mục kết quả. Nếu thư mục đã tồn tại gây ra lỗi,
# thì in ra dòng chữ cảnh báo cho người dùng biết thay vì văng lỗi kỹ thuật khó hiểu.
mkdir /data/ket_qua || echo "CẢNH BÁO: Thư mục đã tồn tại trước đó!"
```

---

### 4. Tuyệt chiêu gộp: Viết lệnh `If/Else` trên đúng 1 dòng
Thay vì phải viết một cấu trúc `if...then...else...fi` dài 5 dòng lằng nhằng tốn diện tích, bạn có thể kết hợp `&&` và `||` với dấu ngoặc vuông kiểm tra `[ ]` để thu gọn mọi thứ vào đúng 1 dòng duy nhất.

**Công thức chung:**
```bash
[ Lệnh_Kiểm_Tra ] && Chạy_nếu_Đúng || Chạy_nếu_Sai
```

**Ví dụ thực tế:** Kiểm tra xem file `sample.fastq` có tồn tại trong máy hay không (dùng cờ `-f` trong ngoặc vuông có nghĩa là File exists).
```bash
[ -f "sample.fastq" ] && echo "Tốt, file đã sẵn sàng!" || echo "LỖI: Mất file FASTQ!"
```
*(Cỗ máy hoạt động: Nếu ngoặc vuông tìm thấy file, nó báo Thành công `0`, toán tử `&&` được kích hoạt và in ra báo sẵn sàng. Nếu ngoặc vuông không thấy file, nó báo Lỗi `1`, toán tử `&&` bị chặn lại, luồng dữ liệu tự rẽ sang nhánh `||` và in ra cảnh báo báo lỗi).*

---

### ⚠️ Cạm bẫy cực độ (Dành cho Chuyên gia)
Cú pháp 1 dòng `A && B || C` ở trên trông rất thanh lịch và giống hệt như một câu lệnh `if/else`, nhưng bản chất nó chứa một rủi ro ngầm.
Giả sử Lệnh `A` chạy thành công, nó kích hoạt Lệnh `B`. Nhưng nếu không may **Lệnh B lại bị lỗi**, thì **Lệnh C** đằng sau đó cũng sẽ bị kích hoạt luôn! 
*(Vì luồng đánh giá từ trái sang phải: B lỗi $\rightarrow$ kích hoạt túi khí || C).*

**Bài học đắt giá:** Bạn chỉ nên dùng cấu trúc 1 dòng này khi bạn nắm chắc 100% rằng Lệnh B (`echo` in chữ ra màn hình) là một lệnh vô hại và không bao giờ bị lỗi. Nếu Lệnh B là một lệnh tính toán phức tạp dễ hỏng, hãy ngoan ngoãn quay về dùng khối cấu trúc `if...else` truyền thống!
