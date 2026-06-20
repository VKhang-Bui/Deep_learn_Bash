# Lệnh `echo` và `printf` - Xuất Dữ Liệu Trong Bash

### 1. Bản chất của lệnh `echo`
Trong Bash, `echo` là một lệnh nội trú (built-in command). Bản chất của nó là nhận arguments được ngăn cách bởi khoảng trắng, in chúng ra luồng đầu ra chuẩn (Standard Output - `stdout`), nối với nhau bằng một khoảng trắng, và tự động thêm một ký tự xuống dòng (`\n` - newline) ở cuối.

Cú pháp: `echo [options] [chuỗi_ký_tự...]`

```bash
echo "Hello" "World"
# Kết quả: Hello World
```

### 2. Sự khác biệt sống còn giữa Nháy Kép (""), Nháy Đơn ('') và Không Nháy
Khi làm việc với `echo` (và Bash nói chung), cách bạn bọc chuỗi quyết định việc Bash sẽ "hiểu" chuỗi đó như thế nào trước khi truyền cho `echo`.

- **Nháy kép (`"..."`) - Mềm dẻo (Weak Quoting):** Cho phép "nở" biến (Variable Expansion - biến `$VAR` thành giá trị) và thực thi lệnh (Command Substitution - `` `lệnh` `` hoặc `$(lệnh)`).
- **Nháy đơn (`'...'`) - Cứng nhắc (Strong Quoting):** Khóa cứng mọi thứ (Literal). Mọi ký tự bên trong đều được giữ nguyên vẹn, kể cả `$`.
- **Không dùng nháy (Unquoted):** Bash sẽ tự động phân tách chuỗi bằng khoảng trắng (Word Splitting) và mở rộng ký tự đại diện (Globbing như `*`). Rất nguy hiểm và dễ gây lỗi!

**Ví dụ:**
```bash
FILE="sample.fq"
echo "Xử lý file: $FILE"   # -> Xử lý file: sample.fq
echo 'Xử lý file: $FILE'   # -> Xử lý file: $FILE
echo *                     # -> Sẽ in ra toàn bộ tên file trong thư mục hiện tại! (Globbing)
echo "*"                   # -> *
```

### 3. Các tùy chọn (Options) thay đổi hành vi
- **`-n` (No newline):** Không tự động chèn ký tự xuống dòng ở cuối. Rất hữu ích khi viết script cần người dùng nhập thông tin trên cùng 1 dòng.
  ```bash
  echo -n "Nhập tên mẫu: "
  read SAMPLE_NAME
  ```
- **`-e` (Enable escapes):** Kích hoạt khả năng "dịch" các ký tự thoát (Escape characters) như `\n` (xuống dòng), `\t` (thụt lề tab).
  ```bash
  echo -e "SampleID\tStatus\nS01\t\tPass"
  ```
*Lưu ý: Hành vi của `echo -e` có thể khác nhau giữa các shell (bash, sh, zsh). Điều này dẫn đến sự ra đời của `printf`.*

### 4. Nâng cao: `printf` - Kẻ thay thế hoàn hảo cho `echo`
Trong các script chuyên nghiệp, đặc biệt khi cần định dạng bảng biểu hoặc số liệu, `printf` mạnh mẽ và chuẩn xác hơn `echo` rất nhiều. Nó không tự động thêm ký tự xuống dòng (bạn phải tự thêm `\n`).

```bash
# Định dạng độ rộng cột (ví dụ 10 ký tự, căn trái)
printf "%-10s %-10s\n" "SampleID" "Reads"
printf "%-10s %-10s\n" "S01" "1000000"
```

### 5. Ứng dụng trong Tin sinh học
Sử dụng kết hợp toán tử điều hướng (`>` ghi đè, `>>` ghi nối tiếp) để tự động hóa việc tạo file metadata hoặc script:
```bash
# Tạo nhanh file metadata
echo "Sample,Group" > metadata.csv
echo "S1,Control" >> metadata.csv
```
