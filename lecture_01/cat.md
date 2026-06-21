# Lệnh `cat` (Concatenate)

### 1. Chức năng cốt lõi
Lệnh `cat` xuất phát từ **Concatenate**.
Chức năng chính của `cat` là đọc nội dung từ một hoặc nhiều tập tin một cách tuần tự và xuất (output) ra luồng đầu ra chuẩn (Standard Output - `stdout`).

### 2. Cú pháp và ứng dụng cơ bản

**a. Đọc và xuất nội dung tập tin**
```bash
cat filename.txt
```
*Lưu ý kỹ thuật:* Không sử dụng `cat` để đọc các tập tin dữ liệu lớn (như FASTQ, VCF) vì dữ liệu sẽ xuất trực tiếp và liên tục ra terminal, có thể gây tràn bộ nhớ đệm hiển thị (buffer). Hãy dùng `less` hoặc `head` để thay thế.

**b. Nối nhiều tập tin (Concatenation)**
Kết hợp `cat` với toán tử điều hướng `>` để ghép nội dung từ nhiều tập tin nguồn thành một tập tin đích.
```bash
cat lane1.fastq lane2.fastq lane3.fastq > merged_sample.fastq
```

**c. Khởi tạo tập tin bằng khối văn bản (Here Document)**
Tạo hoặc ghi đè tập tin với nhiều dòng văn bản liên tiếp mà không cần lặp lại lệnh `echo`.
```bash
cat << EOF > script.sh
#!/bin/bash
fastqc sample.fastq
EOF
```
*(Chuỗi `EOF` hoạt động như một delimiter (chuỗi phân cách) báo hiệu kết thúc khối đầu vào).*

### 3. Tùy chọn rà soát định dạng (Option `-A`)
Các tùy chọn định dạng văn bản mặc định của `cat` (như `-n` để đánh số dòng) thường ít được sử dụng trong thực tế do có các công cụ chuyên dụng hơn (`awk`, `sed`, `less`).

Tuy nhiên, tùy chọn **`-A` (`--show-all`)** có giá trị chẩn đoán cao trong việc rà soát lỗi định dạng dữ liệu (Format Error). Tùy chọn này hiển thị trực quan các ký tự điều khiển ẩn (control characters):

- **`^I`**: Tương ứng với ký tự Tab (`\t`).
  *Ứng dụng:* Xác minh tính toàn vẹn của cấu trúc cột trong các tập tin yêu cầu phân cách bằng Tab (TSV, BED, VCF).
- **`^M$`**: Tương ứng với ký tự ngắt dòng của Windows (`\r\n`).
  *Ứng dụng:* Nhận diện tàn dư định dạng khi chuyển tập tin script hoặc metadata từ môi trường Windows sang Linux. Ký tự `\r` ẩn này là nguyên nhân gây lỗi thực thi script.

```bash
# Kiểm tra các ký tự điều khiển ẩn
cat -A script.sh
```

### 4. Anti-pattern: Useless Use of Cat (UUOC)
UUOC (Lạm dụng lệnh `cat`) là một lỗi thiết kế pipeline phổ biến. Lỗi này xảy ra khi lệnh `cat` được khởi tạo chỉ để truyền nội dung của một tập tin duy nhất vào đường ống (pipe `|`), trong khi bản thân lệnh tiếp thu (như `grep`, `awk`) có hỗ trợ tham số đọc tập tin trực tiếp.

**Cú pháp thừa (Sinh thêm process không cần thiết):**
```bash
cat sample.txt | grep "BRCA1"
```

**Cú pháp chuẩn (Tối ưu tài nguyên hệ thống):**
```bash
grep "BRCA1" sample.txt
```