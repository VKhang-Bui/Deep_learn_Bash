# Lệnh `head` và `tail`

### 1. Chức năng cốt lõi
`head` và `tail` là hai công cụ chuyên dụng để trích xuất một phần cụ thể (đầu hoặc cuối) của tập tin hoặc luồng dữ liệu (stream). 

Trái ngược với lệnh `cat` (truy xuất toàn bộ dữ liệu), `head` và `tail` được thiết kế để tối ưu hóa bộ nhớ. Chúng là công cụ tiêu chuẩn để kiểm tra cấu trúc của các tập tin dữ liệu lớn (như FASTQ, SAM/BAM đã chuyển đổi dạng text, VCF) mà không gây tràn bộ nhớ đệm (buffer overflow).

- **`head`**: Xuất luồng dữ liệu bắt đầu từ dòng đầu tiên.
- **`tail`**: Xuất luồng dữ liệu ngược từ dòng cuối cùng.

Mặc định (khi không kèm tham số), cả hai lệnh đều xuất 10 dòng.

### 2. Cú pháp và các tham số kỹ thuật (Options)

**a. Trích xuất theo số lượng dòng (`-n`)**
Tùy chọn `-n` (number) cho phép giới hạn hoặc chỉ định chính xác số lượng dòng cần xử lý.

```bash
# Xuất 20 dòng đầu tiên của tập tin
head -n 20 sample.fastq

# Xuất 5 dòng cuối cùng của tập tin
tail -n 5 execution.log
```

*Lưu ý kỹ thuật mở rộng với toán tử âm/dương (+/-):*
- `head -n -5`: Xuất toàn bộ nội dung tập tin, **NGOẠI TRỪ** 5 dòng cuối cùng.
- `tail -n +5`: Bắt đầu xuất từ dòng thứ 5 cho đến hết tập tin (nghĩa là cắt bỏ 4 dòng đầu). 
  *Ứng dụng:* Kỹ thuật này đặc biệt thiết yếu khi cần loại bỏ phần Header (meta-information) của các tập tin VCF hoặc SAM trước khi đưa vào các bước phân tích tính toán tiếp theo.

**b. Trích xuất theo dung lượng byte (`-c`)**
Tùy chọn `-c` (bytes) dùng để cắt dữ liệu theo dung lượng byte vật lý thay vì theo số dòng (`\n`). Cần thiết khi xử lý dữ liệu nhị phân hoặc chuỗi mã hóa không có ký tự ngắt dòng.

```bash
# Xuất 100 byte đầu tiên
head -c 100 binary_data.bin
```

### 3. Giám sát luồng dữ liệu thời gian thực (`tail -f`)
Tùy chọn `-f` (follow) là một tính năng đặc biệt của lệnh `tail`, đóng vai trò công cụ giám sát tiến trình (monitoring tool).
Thay vì kết thúc tiến trình sau khi in các dòng cuối, `tail -f` duy trì trạng thái mở của `stdout`. Khi tập tin được ghi thêm dữ liệu (append), `tail -f` sẽ lập tức cập nhật và in các dòng mới ra terminal.

```bash
# Theo dõi tiến trình phân tích trực tiếp (Real-time monitoring)
tail -f pipeline_output.log
```

### 4. Ứng dụng kết hợp trong đường ống (Pipeline)
Hiệu năng của `head` và `tail` được khai thác tối đa khi kết hợp thông qua toán tử pipe (`|`).

**a. Kiểm tra an toàn định dạng dữ liệu nén:**
Sử dụng để trích xuất dòng Header nhằm kiểm định cấu trúc cột mà không cần giải nén toàn bộ tập tin.
```bash
zcat data.vcf.gz | head -n 1
```

**b. Trích xuất tọa độ dòng cụ thể (Sub-setting):**
Sử dụng kết hợp `head` và `tail` để cô lập một tập hợp dòng nằm ở giữa tập tin (Ví dụ: Chỉ lấy từ dòng số 101 đến dòng số 120).
```bash
# Bước 1: head giới hạn luồng dữ liệu ở 120 dòng đầu tiên.
# Bước 2: tail trích xuất 20 dòng cuối cùng từ luồng dữ liệu đó.
head -n 120 metadata.tsv | tail -n 20
```
