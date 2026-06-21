# Lệnh `uniq` (Unique)

### 1. Bản chất thuật toán của lệnh `uniq`
`uniq` là một bộ lọc trùng lặp tuyến tính (Linear Deduplication Filter). Bản chất của lệnh này không phải là tìm kiếm và loại bỏ mọi bản ghi trùng lặp trong toàn bộ tập tin, mà là một thuật toán so sánh **kề cận (Adjacent Comparison)**.

**Cơ chế làm việc:**
Thuật toán `uniq` duy trì trong bộ nhớ đệm duy nhất một biến tạm gọi là `previous_line` (dòng trước đó). Khi quét qua luồng dữ liệu, nó thực thi vòng lặp sau:

1. Nạp dòng hiện tại (`current_line`) từ luồng đầu vào.
2. So sánh chuỗi byte của `current_line` với `previous_line`.
3. Nếu hai dòng **KHÔNG trùng khớp**: Xuất `current_line` ra luồng đầu ra (`stdout`), đồng thời cập nhật `previous_line = current_line`.
4. Nếu hai dòng **trùng khớp hoàn toàn**: Hủy bỏ `current_line` (không xuất ra `stdout`), giữ nguyên `previous_line`.
5. Lặp lại cho đến khi chạm tín hiệu kết thúc tập tin (EOF).

**Hệ quả kỹ thuật sống còn:**
Do cơ chế chỉ so sánh dòng hiện tại với dòng liền kề ngay trước nó, `uniq` **TUYỆT ĐỐI KHÔNG CÓ KHẢ NĂNG** phát hiện các bản ghi trùng lặp nằm rải rác ở các vị trí xa nhau trong tập tin. Nếu dữ liệu đầu vào chưa được sắp xếp (unsorted), các bản ghi trùng lặp không kề nhau sẽ lọt qua bộ lọc mà không bị phát hiện.

Đây là lý do **`sort` và `uniq` luôn vận hành theo cặp bắt buộc** trong mọi đường ống kỹ thuật:
```bash
# Quy trình chuẩn: sort gom nhóm các dòng giống nhau lại kề cận -> uniq loại bỏ bản sao
sort raw_data.txt | uniq
```

### 2. Giao diện tham số kỹ thuật (Options/Flags)

**a. Chế độ mặc định — Khử trùng lặp (Deduplication):**
Không cần cờ tùy chọn. Thuật toán sẽ loại bỏ mọi dòng trùng lặp kề cận và chỉ giữ lại một bản ghi duy nhất (đại diện) cho mỗi nhóm.
```bash
# Trích xuất danh sách tên nhiễm sắc thể không trùng lặp từ cột 1 của tập tin BED
cut -f 1 regions.bed | sort | uniq
```

**b. Chế độ đếm tần suất (`-c`, `--count`):**
Cờ `-c` bổ sung một cột đếm (count) ở đầu mỗi dòng đầu ra, phản ánh chính xác số lần bản ghi đó xuất hiện liên tiếp trong luồng dữ liệu đã sắp xếp. Đây là tham số được sử dụng nhiều nhất của `uniq`, biến nó thành một công cụ phân tích tần suất (Frequency Analysis).
```bash
# Thống kê số lượng biến thể (variant) trên từng nhiễm sắc thể trong tập tin VCF
cut -f 1 mutations.vcf | sort | uniq -c
# Output mẫu:
#    1247 chr1
#     983 chr2
#     541 chr3
#     ...
```

Kết hợp thêm `sort -nr` phía sau để xếp hạng từ nhiều đến ít:
```bash
cut -f 1 mutations.vcf | sort | uniq -c | sort -nr
# Output: Nhiễm sắc thể chứa nhiều đột biến nhất sẽ hiển thị ở dòng đầu tiên.
```

**c. Chế độ lọc nghịch — Chỉ in bản ghi trùng lặp (`-d`, `--repeated`):**
Cờ `-d` đảo ngược hành vi mặc định. Thay vì giữ lại các dòng duy nhất, thuật toán chỉ xuất ra những dòng **có ít nhất 2 bản sao kề cận trở lên**. Các dòng chỉ xuất hiện đúng 1 lần sẽ bị loại bỏ hoàn toàn khỏi luồng đầu ra.
```bash
# Phát hiện các mã Gene ID bị trùng lặp trong danh sách metadata (lỗi nhập liệu)
sort gene_ids.txt | uniq -d
# Output: Chỉ hiển thị những Gene ID xuất hiện từ 2 lần trở lên.
```

**d. Chế độ lọc thuận — Chỉ in bản ghi xuất hiện duy nhất (`-u`, `--unique`):**
Cờ `-u` thực hiện phép lọc nghiêm ngặt nhất: Chỉ xuất ra những dòng xuất hiện **đúng 1 lần duy nhất** trong toàn bộ tập hợp đã sắp xếp. Mọi bản ghi có từ 2 bản sao trở lên đều bị hủy bỏ.
```bash
# Trích xuất các mẫu bệnh phẩm chỉ xuất hiện duy nhất trong một danh sách (không bị lặp)
sort sample_list.txt | uniq -u
```

**e. Bỏ qua trường so sánh (`-f N`, `--skip-fields=N`):**
Cờ `-f N` yêu cầu thuật toán bỏ qua (ignore) `N` trường (cột, phân cách bằng khoảng trắng) đầu tiên trước khi tiến hành phép so sánh chuỗi. Điều này cho phép kiểm tra tính trùng lặp dựa trên nội dung phía sau, bất kể giá trị cột đầu khác nhau.
```bash
# Bỏ qua cột số thứ tự (Cột 1) và chỉ kiểm tra trùng lặp dựa trên nội dung từ Cột 2 trở đi
sort -k2 data.tsv | uniq -f 1
```

**f. Bỏ qua ký tự đầu dòng (`-s N`, `--skip-chars=N`):**
Cờ `-s N` tương tự cờ `-f`, nhưng thay vì bỏ qua theo trường, nó bỏ qua `N` ký tự (byte) đầu tiên trên mỗi dòng trước khi thực hiện phép đối sánh.
```bash
# Bỏ qua 5 ký tự đầu dòng (Ví dụ: mã prefix "SRR01") và so trùng phần còn lại
sort data.txt | uniq -s 5
```

### 3. Ứng dụng trong đường ống Tin Sinh Học (Bioinformatics Pipelines)

**a. Bảng phân bố tần suất hoàn chỉnh (Frequency Distribution Table):**
Đây là kỹ thuật đường ống kinh điển nhất của `uniq`, kết hợp 4 lệnh thành một chuỗi logic liền mạch: Cắt cột -> Sắp xếp -> Đếm tần suất -> Xếp hạng.
```bash
# Câu hỏi: Trong tập tin annotation GFF3, loại đặc trưng (feature type) nào xuất hiện nhiều nhất?
cut -f 3 annotation.gff3 | sort | uniq -c | sort -nr | head -n 10
# Output mẫu:
#   45231 exon
#   32104 CDS
#   18776 mRNA
#   12003 gene
#    ...
```

**b. Kiểm tra tính duy nhất của định danh mẫu (Sample ID Validation):**
Trước khi khởi chạy pipeline phân tích trên hàng trăm mẫu, bước kiểm định bắt buộc là đảm bảo không có mã mẫu nào bị trùng lặp trong tập tin danh sách đầu vào (sample sheet). Nếu trùng, pipeline có thể ghi đè dữ liệu đầu ra lên nhau.
```bash
# Đếm tổng số mã mẫu và tổng số mã mẫu sau khi khử trùng
total=$(wc -l < sample_ids.txt)
unique=$(sort sample_ids.txt | uniq | wc -l)

if [ "$total" -ne "$unique" ]; then
    echo "CẢNH BÁO: Phát hiện $((total - unique)) mã mẫu bị trùng lặp."
    sort sample_ids.txt | uniq -d    # In ra các mã bị trùng để rà soát
    exit 1
fi
```

**c. Loại bỏ bản ghi trùng trong kết quả BLAST:**
Sau khi chạy BLAST, danh sách kết quả (hit list) thường chứa nhiều bản ghi trùng lặp (cùng một query khớp nhiều vùng trên cùng một subject). Kết hợp `sort` và `uniq` để cô lập danh sách các subject duy nhất.
```bash
# Cột 2 của BLAST output (outfmt 6) chứa Subject ID
cut -f 2 blast_results.txt | sort | uniq > unique_hits.txt
```
