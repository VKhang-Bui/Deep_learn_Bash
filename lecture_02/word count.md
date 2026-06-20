# Lệnh `wc` (Word Count) - Cỗ Máy Thống Kê Dữ Luệu

### 1. Bản chất của `wc`
`wc` là công cụ đọc luồng dữ liệu và đếm 3 yếu tố cơ bản theo thứ tự: Số dòng (Lines), Số từ (Words), Số Byte (Bytes). Mặc định nếu không có tùy chọn (options), nó sẽ in ra cả 3 thông số này.

```bash
wc sample.txt
# Output: 107  543  3703  sample.txt
# Ý nghĩa: 107 dòng, 543 từ, 3703 byte, tên file.
```

### 2. Các tùy chọn (Options) chuyên biệt
- **`-l` (Lines):** Đếm số dòng. Đây là option sử dụng nhiều nhất, đặc biệt khi đếm dữ liệu Tin sinh.
- **`-w` (Words):** Đếm số từ (các khối chuỗi ký tự được ngăn cách bởi khoảng trắng, dấu tab, hoặc xuống dòng).
- **`-c` (Bytes):** Đếm số dung lượng vật lý (byte).
- **`-m` (Chars):** Đếm số ký tự (Hữu ích khi làm việc với mã hóa UTF-8, vì 1 ký tự như "á" chiếm 2 byte, lúc này `-c` và `-m` sẽ trả về con số khác nhau).
- **`-L` (Longest line):** In ra chiều dài (số lượng ký tự) của dòng dài nhất trong file. Rất hữu ích để đánh giá nhanh chất lượng dữ liệu.

### 3. Cạm bẫy cực kỳ nguy hiểm: Sự thật về `wc -l`
Rất nhiều người lầm tưởng `wc -l` đếm "số lượng dòng văn bản có trong file". 
Sự thật là: **Nó đếm số lượng ký tự xuống dòng (`\n` - newline)**.

Nếu một phần mềm hoặc bạn tự gõ dữ liệu ở dòng cuối cùng nhưng quên chèn ký tự `\n` (không bấm phím Enter ở dòng cuối cùng), thì `wc -l` sẽ đếm hụt mất 1 dòng!
*Bài học:* Trong giới lập trình và hệ điều hành Linux/Unix, mọi file text chuẩn mực luôn phải kết thúc bằng một ký tự rỗng xuống dòng (dấu newline ở cuối cùng).

---

### 4. Các Ví dụ Ứng dụng Thực chiến và Chuyên sâu

Lệnh `wc` ít khi đứng một mình. Sức mạnh thực sự của nó nằm ở việc kết hợp trong các đường ống (pipeline) và các bash script tự động hóa.

**A. Kỹ thuật đưa kết quả đếm vào Script (Tuyệt chiêu tránh lỗi)**
Để đếm dòng và lưu vào một biến nhằm phục vụ tính toán, đừng dùng `cat file | wc -l` (chạy 2 tiến trình gây lãng phí tài nguyên máy chủ), và đừng dùng `wc -l file` (vì kết quả in ra sẽ dính kèm tên file, làm lệnh toán học báo lỗi). Kỹ thuật chuẩn xác nhất là "bơm ngược":
```bash
SO_DONG=$(wc -l < metadata.csv)
echo "File metadata có $SO_DONG mẫu."
# Giải thích: wc nhận luồng dữ liệu vô danh từ dấu < nên nó chỉ in ra con số thuần túy (ví dụ: 50).
```

**B. Đếm số lượng reads trong file FASTQ (.fq)**
Mỗi read trong định dạng FASTQ chiếm cố định 4 dòng. Việc đếm số reads thực hiện bằng cách đếm tổng số dòng rồi chia cho 4 bằng toán tử `$(())` trong Bash:
```bash
reads=$(($(wc -l < sample.fq) / 4))
echo "Số reads của mẫu này là: $reads"

# Nếu là file nén (.gz) khổng lồ:
reads=$(($(zcat sample.fq.gz | wc -l) / 4))
```

**C. Kiểm tra tính toàn vẹn của dữ liệu Paired-End (R1 và R2)**
Khi giải trình tự Illumina Paired-End, số lượng reads trong file R1 **bắt buộc** phải bằng file R2. Nếu tải dữ liệu bị rớt mạng giữa chừng hoặc file bị lỗi, số lượng này sẽ lệch. Dùng `wc -l` là cách nhanh nhất để "khám bệnh":
```bash
lines_R1=$(zcat sample_R1.fastq.gz | wc -l)
lines_R2=$(zcat sample_R2.fastq.gz | wc -l)

if [ "$lines_R1" -ne "$lines_R2" ]; then
    echo "CẢNH BÁO NGUY HIỂM: File bị hỏng! R1 và R2 không khớp số lượng."
else
    echo "OK: R1 và R2 hoàn hảo."
fi
```

**D. Đếm số trình tự trong file FASTA (.fa) - Một cảnh báo**
Định dạng FASTA không cố định số dòng cho mỗi trình tự (một trình tự genome có thể bị ngắt dòng rất nhiều lần). Vì vậy ta **tuyệt đối không được dùng wc -l** để đếm chia trung bình. Thay vào đó, phải đếm số lượng dòng tiêu đề (luôn bắt đầu bằng ký tự `>`). 
Lúc này, ta nhường sân khấu cho lệnh `grep`:
```bash
# -c của grep nghĩa là count. Đếm các dòng bắt đầu (^) bằng dấu >
grep -c "^>" genome.fasta
```

**E. Đếm số lượng file kết quả trong một thư mục**
Ví dụ, bạn chạy công cụ tách mẫu (demultiplexing) sinh ra hàng trăm file con và muốn đếm xem có bao nhiêu file mẫu Fastq đã được tạo ra thành công:
```bash
# Lệnh ls -1 (số 1) in mỗi tên file trên 1 dòng độc lập.
# Dấu pipe | đẩy danh sách đó sang wc -l để đếm số dòng, tương đương với số file.
ls -1 *.fastq | wc -l

# Cách an toàn và chuẩn mực hơn với lệnh find:
find . -maxdepth 1 -name "*.fastq" | wc -l
```

**F. Tìm độ dài của chuỗi (Sequence) lớn nhất bằng `wc -L`**
Nếu bạn có một file FASTA mà mỗi trình tự đã được đưa về dạng 1 dòng duy nhất (single-line FASTA), hoặc một file chứa các barcodes. Bạn muốn biết độ dài tối đa là bao nhiêu:
```bash
# -L quét qua file và trả về số ký tự của dòng dài nhất
wc -L barcodes.txt
```