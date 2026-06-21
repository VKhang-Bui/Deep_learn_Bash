# Phân tích Ký tự `$`: Cơ chế Expansion trong Bash

Trong môi trường Bash, ký tự `$` không chỉ đơn thuần là dấu hiệu nhận biết biến số (Variable). Về mặt kỹ thuật, `$` đóng vai trò là ký tự kích hoạt cơ chế **Expansion (Nội suy/Khai triển)**. Việc hiểu rõ cơ chế này là vạch ranh giới giữa việc gõ lệnh cơ bản và khả năng làm chủ pipeline dữ liệu trong Tin sinh học.

---

### 1. Bản chất cốt lõi: Cơ chế Expansion
Khi Bash phân tích cú pháp (parsing) của một câu lệnh, nó sẽ xử lý từ trái sang phải. Bất cứ khi nào bắt gặp ký tự `$`, Bash sẽ tạm dừng việc phân tích chuỗi văn bản thông thường, trích xuất giá trị tương ứng từ bộ nhớ hệ thống và **thế chỗ (expand)** vào vị trí đó TRƯỚC KHI câu lệnh thực sự được thi hành.

Ví dụ: lệnh `echo $USER`
Bash không in ra cụm chữ "$USER". Nó phân tích `$USER`, truy xuất hệ thống để lấy tên người dùng (ví dụ: `admin`), thế chỗ vào lệnh thành `echo admin`, sau đó mới chuyển lệnh đi thi hành.

---

### 2. Bốn hình thức Expansion cốt lõi
Ký tự `$` đóng vai trò khởi tạo cho 4 cấu trúc Expansion chính:

**A. Parameter Expansion: `$VAR` hoặc `${VAR}`**
Sử dụng để truy xuất giá trị của một biến. Cú pháp có ngoặc nhọn `${}` được ưu tiên sử dụng nghiêm ngặt để phân định rõ ranh giới của tên biến.
```bash
SAMPLE="Patient_01"
echo "${SAMPLE}_R1.fastq"  # An toàn: Tránh việc Bash tìm kiếm nhầm biến tên là $SAMPLE_R1
```

**B. Command Substitution: `$(command)`**
Khi sử dụng `$()`, Bash sẽ khởi tạo một tiến trình con (Subshell) độc lập để chạy câu lệnh bên trong, sau đó thu thập toàn bộ đầu ra (stdout) của lệnh đó và thế chỗ văn bản vào lệnh gốc.
```bash
# Đếm số lượng trình tự trong FASTA và lưu con số kết quả vào biến
SEQ_COUNT=$(grep -c "^>" genome.fasta)
```

**C. Arithmetic Expansion: `$((expression))`**
Mặc định, Bash xử lý mọi dữ liệu dưới dạng chuỗi văn bản. Để buộc hệ thống thực hiện các phép toán số học nguyên lý, biểu thức phải được bao bọc trong `$(( ))`.
```bash
TOTAL_READS=1000
echo "Số read trên mỗi luồng xử lý: $((TOTAL_READS / 4))"
```

**D. ANSI-C Quoting: `$'string'`**
Một tính năng tinh chỉnh nâng cao giúp giải mã các ký tự điều khiển (Escape sequences) như Tab (`\t`), Newline (`\n`) ở cấp độ shell trước khi truyền vào lệnh.
```bash
# Ép công cụ grep hiểu \t là một khoảng Tab vật lý thay vì chuỗi "\t"
grep $'\t' data.vcf
```

---

### 3. Các Biến Đặc Biệt (Special Parameters)
Hệ điều hành duy trì một số biến đặc biệt được quản lý tự động. Người dùng sử dụng `$` để truy xuất thông tin hệ thống:

- **`$?` (Exit Status):** Lưu trữ mã trạng thái của câu lệnh ngay trước đó (0 = Thành công, >0 = Có lỗi). Đây là cơ sở nguyên lý cho các toán tử phân luồng `&&` và `||`.
- **`$1`, `$2`, `$@` (Positional Parameters):** Đại diện cho các đối số truyền vào khi thực thi script. `$1` là đối số thứ nhất, `$@` đại diện cho toàn bộ đối số. Chúng mang tính quyết định khi xây dựng các pipeline linh hoạt (dynamic pipeline).
- **`$$` (Process ID - PID):** Mã định danh tiến trình của shell hiện tại. 
  *Ứng dụng hệ thống:* Khi xử lý dữ liệu song song (Parallel execution) cho hàng loạt mẫu, việc gán `$$` vào đuôi tên file tạm (ví dụ: `temp_$$.txt`) đảm bảo các tiến trình không bị ghi đè chéo dữ liệu lên nhau.

---

### 4. Quản lý Quoting: Nháy kép `" "` và Nháy đơn `' '`
Mức độ hoạt động của cơ chế Expansion phụ thuộc hoàn toàn vào cách sử dụng dấu bao bọc:

- **Nháy kép `" "` (Weak Quoting):** Cho phép cơ chế Expansion tiếp tục hoạt động. Bash sẽ phân tích và khai triển mọi dấu `$`, `\`, và `` ` `` nằm bên trong nó.
- **Nháy đơn `' '` (Strong Quoting):** Vô hiệu hóa hoàn toàn cơ chế Expansion. Mọi ký tự bên trong, kể cả `$`, đều bị đóng băng và giữ nguyên bản dưới dạng văn bản tĩnh (Literal text).

```bash
PRICE="100"

# Expansion hoạt động
echo "Chi phí: $PRICE"   # Output: Chi phí: 100

# Expansion bị vô hiệu hóa
echo 'Chi phí: $PRICE'   # Output: Chi phí: $PRICE
```

**Lưu ý thực hành:** Khi viết các biểu thức Regular Expression (RegEx) phức tạp cho các công cụ như `grep`, `sed`, hoặc `awk`, quy tắc chuẩn mực là luôn bao bọc RegEx trong nháy đơn `' '`. Việc này nhằm ngăn chặn rủi ro Bash hiểu nhầm các ký hiệu toán học của RegEx thành cú pháp Expansion và làm hỏng biểu thức gốc.
