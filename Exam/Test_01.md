# BÀI KIỂM TRA THỰC HÀNH 01
**Phạm vi:** Các lệnh cơ bản `echo`, `cat`, `head`, `tail`, `wc`, `grep`, `cut`.
**Mục tiêu:** Ứng dụng để giải quyết các pipeline phân tích cơ bản trong thực tế Tin sinh học.
**Dữ liệu:** Các file mẫu (`.fastq`, `.fasta`, `.vcf`) đã được chuẩn bị sẵn trong thư mục `data/` cùng cấp.

---

### Hướng dẫn
Hãy mở Terminal (Git Bash, WSL hoặc Terminal của VSCode), di chuyển (`cd`) vào thư mục `D:\1.HocViec\Tin Sinh Hoc\Deep_learn_Bash\Exam\data` và thử ghép nối các lệnh bạn đã học để giải các bài toán sau. KHÔNG nhìn đáp án, chỉ dùng tài liệu lý thuyết để tư duy.

---

### Bài 1 (Khởi động - Mức độ Dễ)
- **Tình huống:** Bạn vừa tải xong file `sample_01.fastq`. Bạn cần kiểm tra xem trong file này có tổng cộng bao nhiêu reads (trình tự). 
- **Yêu cầu:** Dùng MỘT lệnh (hoặc kết hợp bash math) để tính toán và in ra con số số lượng reads có trong file.

```bash
# dùng grep để bóc ký tự @ đầu, -c để điếm nó lấy bao nhiêu lần
grep -c "^@" sample_01.fastq
```

### Bài 2 (Đếm thông minh - Mức độ Trung bình)
- **Tình huống:** Kế tiếp, bạn có file `genome.fasta`. 
- **Yêu cầu:** Bạn hãy dùng lệnh để đếm xem có bao nhiêu chuỗi trình tự (sequence) bên trong file này. (Gợi ý: Cẩn thận, FASTA không có số dòng cố định như FASTQ đâu!).

```bash
# dùng grep bóc ký tự > đầu, -c đếm số lần nó xuất hiện, tương ứng với số chuỗi trình tự
grep -c ">" genome.fasta
```

### Bài 3 (Gọt dữ liệu - Mức độ Khá)
- **Tình huống:** File `variants.vcf` chứa các kết quả đột biến, nhưng bị dính các dòng Header giải thích lằng nhằng ở đầu.
- **Yêu cầu:** Viết một đường ống (pipeline) gộp 3 lệnh liên tiếp thực hiện 3 công việc:
  1. Lọc vứt bỏ tất cả các dòng Header.
  2. Bắt lấy luồng dữ liệu, dùng "dao" gọt lấy nguyên Cột 1 (NST) và Cột 2 (Tọa độ).
  3. Lọc qua một "van ngắt" để chỉ in ra màn hình đúng **2 dòng đầu tiên** của kết quả gọt được.

```bash
grep -v "^#" variants.vcf | cut -f 1,2 | head -n 2
```

### Bài 4 (Truy vết Lỗi - Mức độ Khó)
- **Tình huống:** Một công cụ chạy lỗi và báo rằng file `sample_01.fastq` của bạn có chứa các Nu lạ chưa xác định. Bạn nghi ngờ đó là cụm chuỗi `NNNNN`.
- **Yêu cầu:** Dùng `grep` tìm trong file `sample_01.fastq` những dòng chứa chuỗi `NNNNN`. **TUY NHIÊN**, nếu chỉ in ra dòng chứa chuỗi đó thì bạn sẽ không biết nó thuộc về Read ID nào. Bạn bắt buộc phải cấu hình `grep` sao cho hiển thị được cả dòng Header (bắt đầu bằng `@`) nằm ngay trước dòng lỗi đó.

```bash
grep -A "NNNNN" sample_01.fastq
```

### Bài 5 (Tạo Pipeline Hoàn Chỉnh - Mức độ Cực Khó)
- **Tình huống:** Bạn đang làm báo cáo nộp cho Sếp bằng file đột biến `variants.vcf`. Sếp yêu cầu: *"Chỉ lấy những đột biến đạt chuẩn (chữ PASS ở cột FILTER), và anh chỉ cần quan tâm tọa độ và mã ID của đột biến thôi"*.
- **Yêu cầu:** Hãy viết **MỘT ĐƯỜNG ỐNG DUY NHẤT** thực hiện toàn bộ các việc sau theo thứ tự:
  1. Đọc luồng dữ liệu file `variants.vcf`.
  2. Dẹp bỏ các dòng Header.
  3. Chỉ giữ lại những dòng đột biến đạt chuẩn (có chữ `PASS`).
  4. Cắt bóc tách lấy **Cột 2 (Tọa độ)** và **Cột 3 (ID)**.
  5. Đưa qua ống xả cuối cùng để lấy đúng **2 đột biến ở phần đuôi** (cuối cùng) của danh sách.
  6. Điều hướng để lưu kết quả thu được vào một file mới tên là `PASS_report.txt` (KHÔNG in gì ra màn hình).

```bash
grep -v "^#" variants.vcf | grep 'PASS' | cut -f 2,3 >PASS_report.txt
```

---
*Làm xong mỗi bài, bạn có thể dán lệnh lên đây để tôi chấm điểm và hướng dẫn bạn cách tối ưu hóa (nếu có)!*
