### 1. Bản chất cốt lõi: Vòng đời của một lệnh `grep`
Nhiều người nghĩ `grep` chỉ là công cụ tìm chữ (giống tính năng Ctrl+F trên Word). Sự thật: `grep` là một **Stream Processing Engine** hoạt động chặt chẽ theo cơ chế line-by-line.

**Vòng đời hoạt động:**
1. Đọc đúng **1 dòng văn bản** từ luồng đầu vào vào bộ nhớ đệm (buffer).
2. Nạp dòng đó vào **RegEx Engine** để đối chiếu với Mẫu (Pattern) bạn cung cấp.
3. Nếu động cơ báo "Khớp" (Match) $\rightarrow$ Đẩy dòng đó ra màn hình (hoặc sang ống nước tiếp theo).
4. Nếu báo "Không khớp" (No match) $\rightarrow$ Xóa dòng đó khỏi bộ nhớ, đọc dòng tiếp theo.

*Hệ quả to lớn:* Lệnh `grep` cực kỳ tiết kiệm RAM. Nó có thể rà soát file FASTQ nặng 500GB trên máy tính chỉ có 4GB RAM mà không bao giờ bị treo máy, vì nó xử lý xong dòng nào là vứt dòng đó đi ngay lập tức.

---

### 2. Sự phân ly giữa "Vỏ Bash" và "Lõi grep" (Lỗi sai kinh điển)
Khi bạn gõ lệnh: `grep > file.fasta` (Bạn muốn tìm dòng chứa dấu lớn hơn `>`).
Nếu bạn gõ như trên, file FASTA của bạn sẽ bị **xóa trắng ngay lập tức**! 

Tại sao? Vì Bash (lớp vỏ ngoài) sẽ xử lý trước khi lệnh `grep` kịp chạy. Bash nhìn thấy dấu `>` và tưởng bạn đang dùng toán tử "Ghi đè luồng" (Redirection) vào file.fasta. Lệnh `grep` bên trong bị thiếu đối số và đứng treo ở đó.

**Quy luật Tối thượng:** Mọi Mẫu (Pattern) truyền vào `grep` **BẮT BUỘC** phải được bọc trong dấu nháy kép `"..."` hoặc nháy đơn `'...'` để bảo vệ nó khỏi lớp vỏ Bash (bảo vệ khỏi các dấu `*`, `>`, `<`, `$`, khoảng trắng).
- Dùng `'...'` (Nháy đơn): Chặn tuyệt đối mọi can thiệp. (Dùng khi Pattern có các ký tự logic phức tạp).
- Dùng `"..."` (Nháy kép): Bảo vệ mẫu nhưng vẫn cho phép Bash chèn giá trị của biến vào (Ví dụ: `grep "$GENE_NAME" file.txt`).

---

### 3. Động cơ Biểu Thức Chính Quy: Sự khác biệt giữa BRE và ERE
Đây là rào cản lớn nhất khiến người học Bash Script nản chí. Tại sao bạn viết mẫu `"A|B"` (để tìm chữ A hoặc B), nhưng `grep` không tìm ra kết quả nào?
Nguyên nhân là `grep` sở hữu 2 động cơ tư duy khác nhau:

**A. BRE (Basic Regular Expression) - Chế độ Mặc định**
Ở chế độ này, các ký tự logic toán học như `|`, `(`, `)`, `+`, `?` bị coi là các chữ cái bình thường (literal character). Khi bạn gõ `grep "A|B"`, máy tính hiểu là: "Hãy đi tìm một dòng có đúng 3 ký tự liền nhau: Chữ A, Dấu gạch dọc, Chữ B".
*(Muốn dùng tính năng logic của chúng trong BRE, bạn phải thêm dấu escape trước chúng: `\|`)*

**B. ERE (Extended Regular Expression) - Kích hoạt bằng `grep -E` (hoặc lệnh `egrep`)**
Ở chế độ này, các ký tự trên được thức tỉnh và đóng vai trò là "Lệnh Logic".
- Mẫu `"A|B"` $\rightarrow$ Hiểu là: Tìm chữ A **HOẶC** chữ B.
- Mẫu `"A+"` $\rightarrow$ Hiểu là: Chữ A xuất hiện **1 hoặc nhiều lần**.
- Mẫu `"(Human)? virus"` $\rightarrow$ Hiểu là: Nhóm chữ "Human" có thể xuất hiện **hoặc không** đều được.

*Bài học:* Trong Tin sinh học, khi cần tìm kiếm phức tạp (Nhiều điều kiện HOẶC), hãy luôn dùng `grep -E`.

---

### 4. Thay đổi hành vi Hiển thị của `grep`
Bản chất của `grep` là in ra **TOÀN BỘ DÒNG** chứa từ khóa. Nhưng đôi khi điều đó là thảm họa (Ví dụ file FASTA một dòng Sequence dài 1 triệu ký tự). Bạn phải ép `grep` đổi tính cách bằng các Option sau:

**A. Kẻ phá luật: Tùy chọn `-o` (Only matching)**
Phá vỡ nguyên tắc "in nguyên dòng". Nó **CHỈ** cắt và in ra đúng phần text khớp với mẫu, vứt bỏ toàn bộ phần râu ria của dòng đó.
*Ví dụ:* `grep -o "ATG[ATGC]*TAA" genome.fa` (Bắt lệnh trích xuất chính xác các trình tự mã hóa bắt đầu bằng ATG và kết thúc bằng TAA. Nếu không có `-o`, máy sẽ in ra cả nhiễm sắc thể!).

**B. Kẻ thu thập ngữ cảnh: Nhóm Context (`-A`, `-B`, `-C`)**
Đôi khi thông tin bạn cần không nằm ở dòng khớp, mà nằm ở dòng sát bên cạnh nó.
- **`-A n` (After):** In dòng khớp + `n` dòng **ngay sau** nó. (Rất hay dùng để lấy Sequence liền sau dòng Header trong FASTQ).
- **`-B n` (Before):** In dòng khớp + `n` dòng **ngay trước** nó.
- **`-C n` (Context):** In dòng khớp + `n` dòng **cả trước và sau** nó.
*(Lưu ý: Nếu `grep` tìm được nhiều đoạn khớp rời rạc, nó sẽ chèn một dòng dấu gạch ngang `--` vào giữa các cụm để ngăn cách, giúp mắt người dễ nhìn).*

---

### 5. Giao tiếp với Hệ thống: Exit Status và `grep -q`
Trong Bash Script, đôi khi bạn KHÔNG cần `grep` in text ra màn hình. Bạn chỉ dùng nó như một công cụ thăm dò: *"Trong file này có chữ ERROR hay không?"*

- **Tùy chọn `-q` (Quiet / Yên lặng):** `grep` sẽ câm lặng hoàn toàn. Ngay khi tìm thấy 1 dòng khớp đầu tiên, nó lập tức tự sát (thoát chương trình) và gửi ngầm một "Báo cáo" (Exit Status) cho hệ điều hành. Rất tiết kiệm thời gian vì không phải quét hết file.
- **Báo cáo Exit Status:**
  - Trả về `0`: Tôi đã tìm thấy! (Thành công / True).
  - Trả về `1`: Tôi đã quét hết file mà không thấy! (Thất bại / False).

*Ứng dụng thực chiến (Lệnh rẽ nhánh If/Else):*
```bash
if grep -q "FAIL" pipeline_qc.log; then
    echo "Dừng mọi thứ! Dữ liệu đầu vào không đạt chất lượng."
    exit 1
else
    echo "Dữ liệu QC đạt chuẩn. Tiếp tục phân tích..."
fi
```

---

### 6. Lọc khối lượng khổng lồ với Bảng Băm: Tuyệt kỹ `grep -f` (File)
*Bài toán:* Bạn có 1 file BAM/FASTQ lớn. Bạn có 1 file danh sách chứa 10,000 tên Barcode/Read_ID cần lọc ra. Bạn không thể viết một câu lệnh `grep` nối 10,000 chữ `|`.
*Giải pháp:* Dùng tùy chọn `-f` (File truyền vào).
```bash
grep -f danh_sach_barcode.txt data.fastq > filtered_data.fastq
```
*Bản chất:* Khi dùng `-f`, `grep` sẽ nạp toàn bộ danh sách Barcode vào RAM dưới dạng một cấu trúc dữ liệu cực nhanh (Hash table / Aho-Corasick automaton). Sau đó nó lấy từng dòng của file FASTQ chạy qua cỗ máy này. Tốc độ tìm kiếm 10,000 mẫu cùng lúc bằng `-f` nhanh ngang ngửa với tìm 1 mẫu bình thường!

---

### 7. Tóm gọn các tùy chọn (Options) cơ bản
Những tùy chọn này kết hợp với các cơ chế ở trên sẽ tạo ra vô số khả năng:
- **`-v` (Invert match):** Đảo ngược logic. Xóa bỏ dòng khớp, lấy dòng KHÔNG khớp (Dùng để loại bỏ Contaminant, hoặc dọn rác log).
- **`-w` (Word):** Buộc phải khớp nguyên vẹn cả một từ độc lập (Bảo vệ bạn khỏi sai lầm khi tìm tên gen. Tìm `TP53` sẽ không bị dính `TP53TG1`).
- **`-i` (Ignore case):** Mù phân biệt chữ hoa/thường (Ví dụ tìm chuỗi `atgc` sẽ khớp luôn cả `ATGC` hay `aTgc`).
- **`-c` (Count):** Không in text, chỉ đếm tổng số lần thỏa mãn điều kiện.