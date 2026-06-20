# Lệnh `head` và `tail` - Kẻ Gác Cổng Luồng Dữ Liệu

### 1. Bản chất của `head` (Đầu)
`head` là một bộ lọc luồng (stream filter) hoạt động như một cái van ngắt tự động (Lazy Evaluation).
Nhiệm vụ của nó: Đứng ở luồng dữ liệu, đếm số dòng (hoặc byte) chảy qua. Khi đạt đủ số lượng, nó **đóng van**, từ chối nhận thêm, và chủ động kết thúc chương trình.

Khi `head` đóng, nó làm gãy đường ống (broken pipe). Linux lập tức gửi tín hiệu `SIGPIPE` để ngắt luôn phần mềm đang nhả dữ liệu ở phía trước nó. 
Nhờ vậy, lệnh `cat 100GB_file | head -n 4` kết thúc ngay lập tức chỉ trong vài mili-giây, thay vì phải đọc hết 100GB.

### 2. Bản chất của `tail` (Đuôi)
Nếu `head` là lấy đầu, `tail` là lấy đuôi. Tuy nhiên, bản chất tính toán đằng sau hoàn toàn khác biệt!
Để biết đâu là 10 dòng cuối cùng của một luồng dữ liệu, `tail` **bắt buộc phải đọc toàn bộ luồng đó** từ đầu đến cuối và giữ một bộ đệm (buffer) trượt tịnh tiến. Do đó, chạy lệnh `cat 100GB_file | tail -n 10` sẽ tốn rất nhiều thời gian vì nó phải đọc qua toàn bộ 100GB.
*(Ghi chú: Nếu `tail` được cung cấp tên file trực tiếp như `tail -n 10 file.txt`, nó thông minh hơn bằng cách nhảy thẳng tới cuối byte của file trên đĩa cứng và đọc ngược lên, tốc độ là ngay lập tức).*

### 3. Cú pháp và Tùy chọn (Options) cơ bản
Mặc định cả hai lệnh in ra 10 dòng.
- **`-n [số_lượng]`:** Chỉ định số dòng.
  ```bash
  head -n 8 sample.fq  # Trích xuất 2 reads đầu tiên của FASTQ (vì 1 read có 4 dòng)
  ```
- **`-c [số_lượng]`:** Chỉ định số byte. Dung lượng vật lý.

### 4. Ứng dụng "Sống còn" trong Tin sinh học

**A. Đọc thử file nén khổng lồ cực nhanh**
Thay vì giải nén file FASTQ, ta dùng `zcat` đẩy qua `head` để nhìn thử cấu trúc của file:
```bash
zcat sample_R1.fastq.gz | head -n 12
```

**B. Theo dõi quá trình chạy thực tế với `tail -f`**
Tùy chọn `-f` (Follow) giữ cho lệnh `tail` không thoát mà liên tục lắng nghe file. Khi file được một phần mềm phân tích khác ghi thêm dòng mới, nó in ngay ra màn hình.
```bash
# Xem log chạy của công cụ căn chỉnh (alignment) theo thời gian thực
tail -f bwa_alignment.log
```

**C. Tuyệt kỹ: Cắt bỏ dòng tiêu đề (Header)**
Rất nhiều file dữ liệu (VCF, CSV, GFF) có dòng đầu tiên là tiêu đề. Làm sao lấy nội dung và bỏ qua dòng 1 khi đưa vào đường ống? Dùng cú pháp dấu `+` của lệnh `tail`:
```bash
# Lấy từ dòng số 2 cho đến hết luồng/file
tail -n +2 metadata.csv
```

**D. Kết hợp `head` và `tail` để "Cắt lát" (Slicing)**
Làm sao để lấy read thứ 2 (dòng 5 đến 8) trong file FASTQ? Cách đơn giản nhất: Lấy 8 dòng đầu tiên, rồi trong 8 dòng đó lọc ra 4 dòng cuối:
```bash
head -n 8 sample.fq | tail -n 4
```
