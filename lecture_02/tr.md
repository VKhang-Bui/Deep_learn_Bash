# Lệnh `tr` (Translate)

### 1. Bản chất thuật toán của lệnh `tr`
`tr` là một bộ ánh xạ ký tự (Character-level Mapping Engine). Khác với mọi lệnh xử lý văn bản khác trong Bash (như `sed`, `awk` thao tác trên chuỗi hoặc dòng), `tr` vận hành ở tầng thấp nhất: **từng byte đơn lẻ**.

**Cơ chế làm việc:**
Khi tiếp nhận luồng dữ liệu, thuật toán `tr` khởi tạo một **bảng tra cứu ánh xạ (Lookup Table)** có kích thước cố định 256 phần tử (tương ứng với 256 mã ASCII). Bảng này được nạp sẵn từ hai tập hợp ký tự (SET1 và SET2) do người dùng cung cấp. Sau đó, thuật toán quét qua luồng dữ liệu từng byte một:

1. Đọc 1 byte từ luồng đầu vào (`stdin`).
2. Tra cứu byte đó trong bảng ánh xạ.
3. Nếu byte khớp với một phần tử trong SET1, thuật toán thay thế nó bằng phần tử tương ứng ở cùng vị trí (index) trong SET2, rồi xuất byte mới ra `stdout`.
4. Nếu byte không khớp, xuất nguyên bản ra `stdout`.

**Ràng buộc kiến trúc quan trọng:**
- `tr` **KHÔNG đọc tập tin trực tiếp**. Nó chỉ tiếp nhận dữ liệu thông qua luồng đầu vào chuẩn (`stdin`). Mọi thao tác bắt buộc phải truyền dữ liệu qua đường ống (`|`) hoặc toán tử chuyển hướng (`<`).
- `tr` **KHÔNG xử lý chuỗi đa ký tự**. Nó chỉ ánh xạ 1 ký tự -> 1 ký tự. Nếu cần thay thế chuỗi (Ví dụ: `chr` -> `chromosome`), bắt buộc phải sử dụng lệnh `sed`.

### 2. Giao diện tham số kỹ thuật (Options/Flags)

**a. Chế độ mặc định — Phiên dịch (Translate):**
Không cần cờ tùy chọn. Chỉ cần cung cấp SET1 và SET2. Thuật toán ánh xạ từng ký tự trong SET1 sang ký tự ở vị trí tương ứng trong SET2.
```bash
# Chuyển đổi toàn bộ nucleotide viết thường sang viết hoa
echo "atcgatcg" | tr 'atcg' 'ATCG'
# Output: ATCGATCG

# Chuyển đổi trình tự DNA sang chuỗi bổ sung (Complementary strand)
echo "ATCGATCG" | tr 'ATCG' 'TAGCTAGC'
# Output: TAGCTAGC
```

**b. Chế độ xóa — Loại bỏ ký tự (`-d`, `--delete`):**
Khi kích hoạt cờ `-d`, thuật toán chuyển sang chế độ lọc. Bất kỳ byte nào khớp với SET1 sẽ bị hủy bỏ hoàn toàn khỏi luồng đầu ra. Cú pháp này không yêu cầu khai báo SET2.
```bash
# Loại bỏ hoàn toàn ký tự ngắt dòng rác của hệ thống Windows (\r) ra khỏi kịch bản script
tr -d '\r' < script_from_windows.sh > clean_script.sh

# Lọc bỏ toàn bộ chữ số khỏi định danh Header tập tin FASTA
echo ">Gene_01_BRCA1" | tr -d '0-9'
# Output: >Gene__BRCA
```

**c. Chế độ nén — Gộp ký tự lặp liên tiếp (`-s`, `--squeeze-repeats`):**
Cờ `-s` yêu cầu thuật toán rà soát luồng dữ liệu và gộp (collapse) mọi khối ký tự lặp liên tiếp trùng khớp với SET1 thành một ký tự duy nhất.
```bash
# Chuẩn hóa cấu trúc phân cách hỗn loạn (nhiều space liên tiếp) thành 1 khoảng trắng duy nhất
echo "gene1    gene2       gene3" | tr -s ' '
# Output: gene1 gene2 gene3

# Hợp nhất các dòng trống liên tiếp thừa thãi thành một dòng duy nhất
tr -s '\n' < messy_output.txt > clean_output.txt
```

**d. Chế độ phần bù — Thao tác nghịch đảo (`-c`, `--complement`):**
Cờ `-c` tiến hành đảo ngược tập hợp SET1. Thuật toán sẽ không nhắm vào các ký tự nằm trong SET1, mà tác động lên **toàn bộ các ký tự còn lại** của bảng mã ASCII không thuộc SET1.
```bash
# Xóa bỏ mọi ký tự KHÔNG phải là nucleotide chuẩn (A,T,G,C,a,t,g,c) và ký tự ngắt dòng (\n)
tr -cd 'ATGCatgc\n' < raw_sequence.txt > pure_nucleotides.txt
```

### 3. Cú pháp khai báo tập hợp ký tự (POSIX Character Classes)
Để gia tăng độ tương thích (portability) của lệnh trên các hệ điều hành dựa trên UNIX, `tr` cung cấp bộ cú pháp tiêu chuẩn nhằm định nghĩa các nhóm ký tự:

| Cú pháp POSIX | Nhóm ký tự quy chiếu |
|---|---|
| `[:upper:]` | Toàn bộ các chữ cái in hoa (A-Z) |
| `[:lower:]` | Toàn bộ các chữ cái in thường (a-z) |
| `[:digit:]` | Nhóm ký tự số học (0-9) |
| `[:alpha:]` | Trọn bộ chữ cái Alphabet (a-z, A-Z) |
| `[:space:]` | Tập hợp các ký tự tạo khoảng trống (Space, Tab, Newline...) |

```bash
# Tiêu chuẩn hóa chuỗi dữ liệu sang chữ hoa bằng cấu trúc POSIX
echo "atcgatcg" | tr '[:lower:]' '[:upper:]'
# Output: ATCGATCG
```

### 4. Ứng dụng trong đường ống Tin Sinh Học (Bioinformatics Pipelines)

**a. Chuyển đổi định dạng phân mảnh (Delimiter Transformation):**
Biến đổi tập tin metadata cấu trúc CSV (phân cách bằng dấu phẩy) sang TSV (phân cách bằng Tab) nhằm phục vụ các phần mềm xử lý hạ nguồn chuyên biệt.
```bash
# Ánh xạ toàn bộ dấu phẩy ',' thành ký tự Tab '\t'
tr ',' '\t' < metadata.csv > metadata.tsv
```

**b. Tái cấu trúc chuỗi trình tự (Sequence Flattening):**
Bằng cách kết hợp `grep` (lọc bỏ dòng tiêu đề bắt đầu bằng `>`) và `tr` (tiêu diệt ký tự ngắt dòng `\n`), hệ thống sẽ nén toàn bộ luồng trình tự thành một chuỗi DNA liên tục trên một dòng duy nhất.
```bash
grep -v "^>" gene_sequence.fasta | tr -d '\n' > single_line_sequence.txt
```
