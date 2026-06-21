# Lệnh `wc` (Word Count)

### 1. Bản chất thuật toán của lệnh `wc`
`wc` (Word Count) là một tiện ích phân tích luồng dữ liệu. Bản chất của lệnh này không phải là "đọc hiểu" văn bản, mà là một vòng lặp quét qua luồng dữ liệu tuần tự từng byte một từ điểm bắt đầu đến điểm kết thúc (độ phức tạp O(n)).

**Nó làm việc như thế nào?**
Bên trong bộ nhớ, thuật toán `wc` khởi tạo và duy trì đồng thời 3 biến đếm nội bộ: `byte_count`, `word_count`, và `newline_count`. Khi tiến hành nạp và quét qua từng ký tự trong luồng dữ liệu, nó kích hoạt logic đánh giá sau:

1. Cứ mỗi byte đọc được thành công, thuật toán cộng `byte_count` lên 1.
2. Nó sử dụng một Cỗ máy trạng thái (State Machine) để theo dõi loại ký tự. Nếu phát hiện sự chuyển đổi từ "ký tự phân cách" (khoảng trắng, Tab, ngắt dòng) sang một "ký tự hiển thị được", nó cộng `word_count` lên 1.
3. Nó thực hiện phép so sánh cấp thấp: Nếu byte vừa đọc khớp hoàn toàn với mã ASCII 10 (ký tự ngắt dòng `\n`), nó cộng `newline_count` lên 1. Nó tuyệt đối không phân tích cấu trúc đoạn văn, nó chỉ tìm và đếm dấu hiệu `\n`.

Khi chạm đến tín hiệu kết thúc tập tin (EOF - End of File), vòng lặp dừng lại. `wc` sẽ xuất 3 biến đếm này ra luồng chuẩn. Mặc định nó luôn in trọn bộ cả 3 thông số theo trật tự (Dòng - Từ - Byte).

### 2. Giao diện tham số kỹ thuật (Options/Flags)
Mặc định, khi không được cấp cờ tùy chọn, `wc` sẽ in ra màn hình toàn bộ 3 thông số nội bộ. Tuy nhiên, để phục vụ việc gán biến hệ thống và tự động hóa trong đường ống (pipeline), kỹ sư phải sử dụng các tham số để cô lập chính xác một trường dữ liệu duy nhất.

**a. Tham số định lượng trục dọc (Vertical Counting):**
- **`-l` (`--lines`)**: Yêu cầu thuật toán chỉ in ra giá trị của biến đếm `newline_count` (Tổng số mã ASCII 10).
  - *Tính ứng dụng:* Tham số xương sống để trích xuất kích thước dữ liệu theo chiều dọc. Nó là công cụ cơ bản nhất để ước lượng khối lượng bản ghi.
  - *Ví dụ kỹ thuật:* Đếm tổng số lượng dòng trong tập tin danh sách tên mẫu bệnh phẩm.
    ```bash
    wc -l sample_list.txt
    # Output: 150 sample_list.txt
    ```

**b. Tham số định lượng cấp độ chuỗi (String/Word Counting):**
- **`-w` (`--words`)**: Cô lập và in ra biến đếm `word_count`.
  - *Tính ứng dụng:* Tham số này hiếm khi được sử dụng để phân tích dữ liệu sinh học. Nó chủ yếu được dùng trong lập trình Bash để đếm số lượng tham số (arguments) truyền vào luồng.
  - *Ví dụ kỹ thuật:* Đếm số lượng từ phân lập trong một chuỗi ký tự bất kỳ.
    ```bash
    echo "Homo sapiens chromosome 1" | wc -w
    # Output: 4
    ```

**c. Tham số định lượng cấp độ vật lý (Physical/Memory Counting):**
- **`-c` (`--bytes`)**: In ra kích thước tập tin dựa trên hệ đếm byte vật lý (1 byte = 8 bit).
- **`-m` (`--chars`)**: In ra tổng số ký tự hiển thị thực tế trên màn hình.
  - *Sự khác biệt giữa `-c` và `-m`:* Khi thao tác với luồng chứa mã ASCII tiêu chuẩn (chuỗi nucleotide `A,T,G,C`), `-c` và `-m` cho kết quả y hệt nhau. Nếu tập tin chứa bảng mã UTF-8 (ký tự đa byte), `-c` sẽ lớn hơn `-m`. Dùng `-c` để đo lường RAM/ổ cứng, dùng `-m` để đo lường độ dài chuỗi sinh học.
  - *Ví dụ kỹ thuật:* 
    ```bash
    # Đo lường dung lượng byte vật lý của file tham chiếu hệ gen
    wc -c hg38.fasta
    # Output: 3234834661 hg38.fasta
    
    # Đo lường chính xác tổng số ký tự chữ cái
    wc -m hg38.fasta
    ```

**d. Tham số phân tích biên độ ngang (Horizontal Amplitude):**
- **`-L` (`--max-line-length`)**: Không trả về tổng số, mà phản hồi lại chiều dài (số lượng ký tự hiển thị) của dòng dài nhất xuất hiện trong luồng dữ liệu.
  - *Tính ứng dụng:* Hoạt động như một công cụ chẩn đoán giới hạn. Được dùng để kiểm tra độ dài của đoạn trình tự FASTA dài nhất, qua đó thiết lập kích thước cấp phát bộ đệm (Buffer Allocation) cho phần mềm hạ nguồn.
  - *Ví dụ kỹ thuật:* Tìm kích thước của chuỗi mã vạch tế bào (barcode) dài nhất.
    ```bash
    wc -L single_cell_barcodes.txt
    # Output: 16
    ```

### 3. Ứng dụng trong đường ống Tin Sinh Học (Bioinformatics Pipelines)

**a. Định lượng hệ dữ liệu giải trình tự (FASTQ):**
Mỗi bản ghi (read) trong kiến trúc FASTQ chiếm chính xác 4 dòng. Khai thác đặc tính này để kiểm kê tổng số luồng giải trình tự:
```bash
# Đếm tổng số dòng và thực hiện phép chia 4
reads_count=$(($(wc -l < sample.fastq) / 4))
echo "Tổng số bản ghi: $reads_count"
```
*(Kỹ thuật truyền luồng `<`: Lệnh `wc -l sample.fastq` sẽ in ra cả con số lẫn tên tập tin. Cấu trúc `wc -l < sample.fastq` cung cấp luồng dữ liệu vô danh, ép `wc` chỉ trả về con số thuần túy, đảm bảo an toàn tuyệt đối cho các phép gán biến toán học).*

**b. Kiểm định tính toàn vẹn của dữ liệu Paired-End:**
Dữ liệu Paired-End (R1 và R2) bắt buộc phải tương xứng tuyệt đối về số lượng bản ghi. `wc -l` đóng vai trò module kiểm định (sanity check) trước khi kích hoạt pipeline phân tích cấu trúc sâu.
```bash
lines_R1=$(zcat sample_R1.fastq.gz | wc -l)
lines_R2=$(zcat sample_R2.fastq.gz | wc -l)

if [ "$lines_R1" -ne "$lines_R2" ]; then
    echo "LỖI HỆ THỐNG: Mất đồng bộ số lượng bản ghi giữa R1 và R2."
    exit 1
fi
```

**c. Thống kê tập tin đầu ra:**
Kết hợp với bộ định tuyến `find` để đếm tổng số tập tin kết quả thỏa mãn điều kiện thực thi.
```bash
# Đếm tổng số tập tin VCF được tạo ra trong cấp thư mục hiện tại
find . -maxdepth 1 -name "*.vcf" | wc -l
```