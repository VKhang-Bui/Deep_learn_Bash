# Lệnh `paste`

### 1. Bản chất thuật toán của lệnh `paste`
`paste` là một toán tử ghép nối ngang vô điều kiện (Unconditional Horizontal Concatenator). Nếu `cat` nối các tập tin theo chiều dọc (hết file 1 rồi đến file 2), thì `paste` nối chúng theo **chiều ngang**: Dòng 1 của file A ghép với dòng 1 của file B, dòng 2 ghép với dòng 2, và tiếp tục cho đến hết.

**Cơ chế làm việc:**
Thuật toán `paste` duy trì đồng thời nhiều con trỏ đọc (mỗi con trỏ gắn với một tập tin đầu vào). Ở mỗi vòng lặp:

1. Đọc đồng thời một dòng từ mỗi tập tin đầu vào thông qua các con trỏ tương ứng.
2. Nối tất cả các dòng vừa đọc thành một bản ghi duy nhất, xen kẽ giữa chúng bằng ký tự phân cách (mặc định là Tab `\t`).
3. Xuất bản ghi đã nối ra luồng đầu ra (`stdout`).
4. Nếu một tập tin cạn kiệt dữ liệu (chạm EOF) trước các tập tin khác, thuật toán chèn trường rỗng (empty field) vào vị trí tương ứng và tiếp tục cho đến khi tập tin dài nhất kết thúc.

**Sự khác biệt cốt lõi giữa `paste` và `join`:**
- `paste` ghép nối **mù quáng** theo số thứ tự dòng. Nó không quan tâm đến nội dung hay giá trị khóa. Dòng 1 ghép với dòng 1, dòng 50 ghép với dòng 50, bất kể chúng có liên quan logic với nhau hay không.
- `join` ghép nối **có điều kiện** dựa trên giá trị khóa chung, có thể bỏ qua hoàn toàn các dòng không khớp.
- `paste` không yêu cầu dữ liệu phải được sắp xếp trước.

### 2. Giao diện tham số kỹ thuật (Options/Flags)

**a. Chế độ mặc định — Ghép nối đa tập tin:**
Không cần cờ tùy chọn. Cung cấp từ hai tập tin trở lên, thuật toán sẽ ráp nối theo chiều ngang với Tab làm bộ phân cách.
```bash
# File sample_ids.txt:       # File diagnosis.txt:
# PAT_001                    # Lung Cancer
# PAT_002                    # Healthy
# PAT_003                    # Breast Cancer

paste sample_ids.txt diagnosis.txt
# Output:
# PAT_001	Lung Cancer
# PAT_002	Healthy
# PAT_003	Breast Cancer
```

**b. Tùy chỉnh ký tự phân cách (`-d`, `--delimiters=LIST`):**
Cờ `-d` cho phép thay đổi ký tự phân cách mặc định (Tab) sang ký tự bất kỳ. Nếu cung cấp nhiều ký tự, `paste` sẽ luân phiên sử dụng chúng theo chu kỳ (cyclic rotation) giữa các cột.
```bash
# Sử dụng dấu phẩy làm bộ phân cách (xuất ra định dạng CSV)
paste -d ',' sample_ids.txt diagnosis.txt phenotype.txt
# Output:
# PAT_001,Lung Cancer,Stage III
# PAT_002,Healthy,N/A
# PAT_003,Breast Cancer,Stage I

# Luân phiên ký tự phân cách: Tab giữa cột 1-2, dấu phẩy giữa cột 2-3
paste -d '\t,' sample_ids.txt diagnosis.txt phenotype.txt
```

**c. Chế độ nối tuần tự từ một tập tin duy nhất (`-s`, `--serial`):**
Cờ `-s` đảo ngược hoàn toàn hướng nối dữ liệu. Thay vì ghép ngang nhiều tập tin theo từng dòng, nó nén toàn bộ nội dung của **một tập tin** thành **một dòng duy nhất** bằng cách thay thế tất cả ký tự ngắt dòng (`\n`) bằng ký tự phân cách.
```bash
# Nén toàn bộ danh sách Gene ID thành một chuỗi phẳng phân cách bằng dấu phẩy
paste -s -d ',' gene_list.txt
# Input:                  # Output:
# BRCA1                   # BRCA1,TP53,EGFR,KRAS,MYC
# TP53
# EGFR
# KRAS
# MYC
```
Kỹ thuật này cực kỳ hữu ích khi cần tạo chuỗi tham số cho các công cụ dòng lệnh yêu cầu danh sách đầu vào ở dạng "phẳng" (comma-separated list).

### 3. Ứng dụng trong đường ống Tin Sinh Học (Bioinformatics Pipelines)

**a. Lắp ráp ma trận metadata từ các tập tin cột đơn:**
Trong thực tế, dữ liệu metadata lâm sàng thường bị phân mảnh thành nhiều tập tin riêng lẻ (mỗi file chứa một thuộc tính). `paste` cho phép ráp nối nhanh chóng chúng thành một ma trận hoàn chỉnh.
```bash
# Lắp ráp 4 cột metadata thành một bảng TSV hoàn chỉnh
paste sample_ids.txt age.txt sex.txt ethnicity.txt > clinical_metadata.tsv
```

**b. Ghép cặp tên tập tin Paired-End (R1/R2):**
Nhiều công cụ alignment (như `bwa mem`) yêu cầu cung cấp đường dẫn R1 và R2 trên cùng một dòng. `paste` kết hợp hai danh sách đường dẫn riêng biệt thành các cặp song song.
```bash
# Tạo danh sách cặp file R1-R2 cho pipeline alignment
ls *_R1.fastq.gz > r1_list.txt
ls *_R2.fastq.gz > r2_list.txt
paste r1_list.txt r2_list.txt > paired_files.txt
# Output:
# sample_01_R1.fastq.gz	sample_01_R2.fastq.gz
# sample_02_R1.fastq.gz	sample_02_R2.fastq.gz
```

**c. Tạo chuỗi tham số phẳng cho lệnh downstream:**
Một số công cụ phân tích (như `samtools merge`) yêu cầu danh sách đầu vào ở dạng chuỗi phẳng phân cách bằng khoảng trắng thay vì từng dòng riêng biệt. Sử dụng `paste -s` để chuyển đổi.
```bash
# Biến danh sách file BAM (mỗi file trên một dòng) thành một chuỗi tham số trên một dòng duy nhất
bam_list=$(paste -s -d ' ' bam_files.txt)
samtools merge merged_output.bam $bam_list
```

**d. Kết hợp `paste` với toán tử thay thế tiến trình (Process Substitution):**
Khi không muốn tạo tập tin trung gian trên ổ cứng, có thể sử dụng `<(command)` để truyền luồng đầu ra của một lệnh trực tiếp vào `paste` như thể nó là một tập tin vật lý.
```bash
# Ghép ngang cột Chromosome (cắt từ file BED) với cột Gene Name (cắt từ file GFF)
# mà không tạo ra bất kỳ tập tin tạm thời nào
paste <(cut -f 1 regions.bed) <(cut -f 9 annotation.gff3)
```
