# Lệnh `printf` - Nghệ Thuật Định Dạng Dữ Liệu Trong Bash

### 1. Sự thật về lệnh `print` trong Bash
Trước khi nói về `printf`, cần làm rõ một sự nhầm lẫn vô cùng phổ biến đối với người mới: **Trong môi trường Bash nguyên thủy (pure Bash), KHÔNG CÓ lệnh `print`**. 

- Nếu bạn gõ `print "Hello"` trực tiếp trên terminal Bash, hệ thống thường sẽ báo lỗi `command not found`.
- Bạn có thể đã thấy lệnh `print` xuất hiện nhan nhản trong các pipeline Bioinformatics (ví dụ: `awk '{print $1}'`), nhưng hãy nhớ kỹ: đó là lệnh nội bộ của ngôn ngữ **`awk`** (hoặc Python, Perl), KHÔNG phải của Bash.
- Trong môi trường Bash, để xuất và xử lý văn bản, bạn chỉ có 2 vũ khí cốt lõi: `echo` (nhanh, gọn nhẹ) và `printf` (chuyên nghiệp, định dạng nghiêm ngặt).

### 2. Sức mạnh của `printf` (Print Formatted)
`printf` mượn cú pháp và tư duy từ ngôn ngữ lập trình C. Nó khắc phục toàn bộ điểm yếu của `echo`:
- **Không tự động xuống dòng:** `echo` luôn lén thêm `\n`, còn `printf` thì không. Bạn kiểm soát 100% việc khi nào xuống dòng.
- **Tính nhất quán:** `echo` hoạt động khác nhau trên Mac và Linux. `printf` thì giống nhau trên mọi hệ thống.
- **Khả năng định dạng:** Giúp bạn tạo cột, căn lề, làm tròn số thập phân – cực kỳ cần thiết để tạo các file báo cáo sinh học đẹp mắt.

Cú pháp chuẩn: `printf "chuỗi_định_dạng_format" [các_giá_trị_truyền_vào]`

### 3. Hướng dẫn sử dụng từng bước (Cho người mới bắt đầu)
Nếu bạn chưa từng viết code C hay dùng `printf` bao giờ, hãy làm theo 3 bước tư duy sau:

**Bước 1: Hình dung câu bạn muốn in.**
Giả sử bạn muốn in ra màn hình: *Mẫu Tumor_01 có 1500 reads.*

**Bước 2: Tạo "Khuôn" bằng cách thay dữ liệu thành các Lỗ hổng (Placeholders).**
* Chữ `Tumor_01` là một chuỗi văn bản (String) -> Thay bằng `%s`.
* Số `1500` là một số nguyên (Integer) -> Thay bằng `%d`.
=> Câu của bạn trở thành một cái Khuôn: `"Mẫu %s có %d reads.\n"` *(Lưu ý quan trọng: bạn phải tự gõ `\n` ở cuối để nó chịu xuống dòng)*.

**Bước 3: Lắp ráp lệnh.**
Viết `printf`, đặt cái Khuôn vào, rồi liệt kê lần lượt các dữ liệu thực tế để lấp vào lỗ hổng theo đúng thứ tự.
```bash
printf "Mẫu %s có %d reads.\n" "Tumor_01" 1500
```
*Cơ chế hoạt động: "Tumor_01" sẽ tự chạy vào lấp lỗ hổng `%s` đầu tiên, 1500 chạy vào lấp lỗ hổng `%d` tiếp theo.*

---

### 4. Phẫu thuật các Format Specifiers (Ký hiệu `%` định dạng)
Trong `printf`, bạn đóng vai trò là thợ rèn: bạn đúc ra một cái "khuôn" (chứa các biến định dạng bắt đầu bằng chữ `%`), sau đó đổ dữ liệu vào cái khuôn đó.

Dưới đây là các loại khuôn từ cơ bản đến nâng cao:

**a. Các ký hiệu cơ bản (The Basics):**
- `%s`: Dành cho **Chuỗi (String)**. Đây là ký hiệu được dùng nhiều nhất.
- `%d` hoặc `%i`: Dành cho **Số nguyên (Integer)**. Nếu bạn vô tình truyền một chữ cái vào vị trí của `%d`, Bash sẽ báo lỗi (một cách ép kiểu dữ liệu rất tốt để tránh script chạy sai lệnh).
- `%f`: Dành cho **Số thập phân (Float)**. Mặc định nó sẽ in ra 6 số lẻ phía sau (ví dụ: in `3.14` sẽ thành `3.140000`).
- `%%`: Vì ký hiệu `%` bị hệ thống dùng làm dấu định dạng, nên nếu bạn muốn in ra ký hiệu phần trăm thật (ví dụ tỷ lệ mapping 99%), bạn PHẢI viết `%%` (hai dấu liên tiếp).

**b. Các ký hiệu "Deep Bash" ít người biết:**
- `%c`: Chỉ in ra **DUY NHẤT một ký tự đầu tiên** của chuỗi truyền vào. 
  *Ứng dụng trong Bioinformatics:* Khi bạn muốn trích xuất ký tự đầu của một sequence motif dài.
  ```bash
  printf "Nucleotide đầu tiên là: %c\n" "ATGC"  # -> Kết quả: A
  ```
- `%b`: Tương tự `%s` (dành cho chuỗi), nhưng thông minh hơn. Nếu chuỗi truyền vào chứa các escape character (như `\n`, `\t`), `%b` sẽ "hiểu" và phiên dịch chúng ra, còn `%s` thì sẽ in ra nguyên văn chữ `\n`.
  ```bash
  printf "%s\n" "A\tT\tG\tC"  # -> In ra nguyên chữ: A\tT\tG\tC
  printf "%b\n" "A\tT\tG\tC"  # -> Dịch \t ra Tab:   A       T       G       C
  ```

**c. Cơ chế "Tự động lặp khuôn" (Implicit Looping) - Siêu năng lực của bash printf**
Đây là tính năng độc nhất của `printf` trong Bash so với C hay Python.
Nếu "khuôn" của bạn yêu cầu ít dữ liệu, nhưng bạn lại truyền vào số lượng tham số lớn hơn, `printf` sẽ tự động **tái sử dụng cái khuôn đó nhiều lần** cho tới khi hết sạch dữ liệu.

```bash
# Khuôn chỉ yêu cầu một %s, nhưng bạn lại đưa vào tới 3 gene
printf "Tiến hành phân tích gene: %s\n" "BRCA1" "TP53" "MYC"

# Kết quả: printf tự động chạy lặp 3 lần!
# Tiến hành phân tích gene: BRCA1
# Tiến hành phân tích gene: TP53
# Tiến hành phân tích gene: MYC
```
*Bạn thấy đấy! Nhờ cơ chế này, bạn có thể in danh sách dài dằng dặc thành từng dòng chuẩn xác mà hoàn toàn KHÔNG CẦN viết vòng lặp `for` rườm rà!*

### 5. Kỹ thuật định dạng chuyên sâu (Format Modifiers)
Đây là lúc `printf` chứng minh nó đẳng cấp hơn `echo`. Khi lập bảng báo cáo thống kê số lượng read của các mẫu, bạn cần các cột thẳng hàng ngay ngắn.

**a. Căn lề và định hình độ rộng cột (Width Alignment)**
- `%10s`: Dành đúng 10 khoảng trống, **căn lề phải**.
- `%-10s`: Dành đúng 10 khoảng trống, **căn lề trái** (Được sử dụng nhiều nhất).

```bash
# Tạo một cái bảng siêu chuẩn
printf "%-15s %-10s %-10s\n" "Sample_ID" "Reads" "Status"
printf "%-15s %-10d %-10s\n" "Tumor_01" 1500000 "PASS"
printf "%-15s %-10d %-10s\n" "Normal_Ctrl_02" 800 "FAIL"
```
*Dù chữ "Tumor_01" dài ngắn khác nhau, bảng kết quả in ra sẽ luôn thẳng tắp.*

**b. Đệm số 0 ở đầu (Zero Padding)**
Rất hay dùng để đánh số thứ tự thư mục hoặc nhiễm sắc thể (VD: chr01, chr02... chr22).
- `%02d`: Buộc số nguyên luôn chiếm 2 chữ số, nếu thiếu thì bù số 0 ở đầu.

```bash
printf "chr%02d\n" 3   # -> Kết quả in ra: chr03
printf "chr%02d\n" 14  # -> Kết quả in ra: chr14
```

**c. Làm tròn số thập phân (Float Precision)**
Khi tính toán tỷ lệ mapping (Mapping rate) hay P-value, bạn chỉ muốn lấy 2 hoặc 3 số lẻ.
- `%.2f`: Giới hạn lấy đúng 2 chữ số thập phân.

```bash
printf "Mapping rate: %.2f%%\n" 98.127456
# -> Mapping rate: 98.13% (Lưu ý: Nó tự động làm tròn lên!)
# (Mẹo: Để in ra ký tự % thật sự, bạn phải viết %% liền nhau)
```

### 6. Ứng dụng `printf` vào Tin Sinh Học
Khi thao tác với hàng nghìn mẫu FASTA/FASTQ, `printf` giúp bạn tổ chức metadata hoặc tự động sinh ra tên file cực chuẩn.

```bash
# 1. Tạo cấu trúc một file BED cực chuẩn bằng phím Tab (\t) thay vì dấu cách
printf "%s\t%d\t%d\t%s\n" "chr1" 10000 15000 "GeneA" > regions.bed

# 2. Vòng lặp tự động sinh ra tên file đánh số chuẩn format
for i in {1..12}; do
    # Lưu kết quả printf vào một biến
    filename=$(printf "patient_sample_%03d.fastq" $i)
    # Vòng lặp sẽ lần lượt tạo ra: patient_sample_001.fastq đến patient_sample_012.fastq
done
```
