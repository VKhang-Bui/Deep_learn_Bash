# Lệnh `sort`

### 1. Bản chất thuật toán của lệnh `sort`
`sort` là một tiện ích định tuyến và tổ chức dữ liệu. Bản chất của lệnh này không đơn thuần là đảo dòng văn bản, mà là một thuật toán sắp xếp quy mô lớn (chủ yếu dựa trên thuật toán External Merge Sort). Cốt lõi sức mạnh của `sort` là khả năng xử lý các tập tin khổng lồ vượt quá dung lượng RAM của hệ thống (Out-of-core sorting).

**Nó làm việc như thế nào?**
Khi tiếp nhận luồng dữ liệu, thuật toán `sort` vận hành qua 3 giai đoạn:
1. **Nạp và Phân mảnh (Read & Split):** Thuật toán nạp dữ liệu vào bộ nhớ đệm (buffer) dưới dạng mảng các dòng. Nếu dung lượng tập tin vượt giới hạn RAM, nó tự động băm tập tin thành nhiều mảnh (chunks), sắp xếp từng mảnh độc lập trên RAM, sau đó ghi các mảnh tạm thời (tmp files) xuống ổ đĩa cứng.
2. **Đối sánh (Comparison):** Hệ thống lấy ra hai bản ghi bất kỳ và đưa qua hàm đánh giá để quyết định vị trí tương đối. Mặc định, hàm này sẽ so sánh mã nhị phân của từng byte một từ trái sang phải theo chuẩn bảng mã ASCII (So sánh từ điển).
3. **Trộn và Xuất (Merge & Output):** Sau khi toàn bộ các mảnh đã được sắp xếp, thuật toán thực hiện bước trộn (Merge) cuối cùng, ráp nối chúng lại và bơm luồng dữ liệu đã được định tuyến ra luồng đầu ra chuẩn (`stdout`).

### 2. Giao diện tham số kỹ thuật (Options/Flags)
Mặc định thuật toán so sánh từ điển của `sort` sẽ gây ra lỗi logic đối với các con số (Ví dụ: Số `10` sẽ bị xếp trước số `2` do ký tự `1` có mã ASCII nhỏ hơn `2`). Để vô hiệu hóa hành vi này và thiết lập logic ma trận, ta phải sử dụng các tham số điều khiển:

**a. Tham số can thiệp cơ chế so sánh:**
- **`-n` (`--numeric-sort`)**: Ép thuật toán từ bỏ việc so sánh chuỗi ASCII. Thay vào đó, nó ép kiểu (type casting) chuỗi ký tự thành giá trị toán học thực sự để đánh giá.
  - *Ví dụ kỹ thuật:* Sắp xếp danh sách điểm chất lượng (Quality Score) hoặc P-value từ thấp đến cao.
    ```bash
    sort -n p_values.txt
    ```
- **`-r` (`--reverse`)**: Đảo ngược hoàn toàn vector xuất dữ liệu của thuật toán (Từ Lớn đến Bé, hoặc từ Z về A).
  - *Ví dụ kỹ thuật:* Liệt kê các gen có mức độ biểu hiện cao nhất xuống thấp nhất.
    ```bash
    sort -n -r expression_levels.txt
    # Lập trình viên Bash thường viết gộp tham số: sort -nr expression_levels.txt
    ```

**b. Tham số can thiệp cấu trúc ma trận (Trường/Khóa):**
- **`-t` (`--field-separator`)**: Chỉ định ký tự phân cách các trường (cột). Nếu không khai báo, `sort` mặc định coi nhóm khoảng trắng (Space) và Tab là bộ phân cách.
  - *Ví dụ kỹ thuật:* Sắp xếp cấu trúc dữ liệu CSV được phân cách bằng dấu phẩy.
    ```bash
    sort -t ',' -k2,2 database.csv
    ```
- **`-k` (`--key=POS`)**: Định nghĩa giới hạn không gian của bộ nhớ đệm so sánh. Cấu trúc chuẩn là `-k[Cột_bắt_đầu],[Cột_kết_thúc]`. 
  - *Lỗ hổng kỹ thuật:* Nếu chỉ viết `-k2`, thuật toán sẽ nạp toàn bộ chuỗi từ cột 2 kéo dài đến hết dòng để làm khóa đối sánh, gây lãng phí chu kỳ CPU và sai lệch kết quả. Phải luôn thiết lập điểm dừng `-k2,2` để cô lập khóa duy nhất tại cột 2.
  - *Ví dụ kỹ thuật:* Sắp xếp tập tin BED dựa trên cột 2 (Tọa độ bắt đầu) theo toán học.
    ```bash
    sort -k2,2n target_regions.bed
    ```

### 3. Ứng dụng trong đường ống Tin Sinh Học (Bioinformatics Pipelines)

**a. Khóa đa cấp (Multi-level Sorting) cho hệ tọa độ:**
Các tập tin không gian hệ gen (như BED, VCF) luôn yêu cầu sắp xếp nghiêm ngặt theo 2 cấp độ: Cấp 1 là Nhiễm sắc thể (Cột 1, kiểu chuỗi), Cấp 2 là Tọa độ (Cột 2, kiểu số). Đây là điều kiện bắt buộc trước khi đưa vào các gói phần mềm hạ nguồn như `bedtools`.
```bash
# Cấp 1 (-k1,1): Gom nhóm theo nhiễm sắc thể (Ví dụ: chr1, chr2...)
# Cấp 2 (-k2,2n): Bên trong mỗi nhiễm sắc thể, định tuyến tọa độ toán học từ nhỏ đến lớn.
sort -k1,1 -k2,2n annotation.vcf > sorted_annotation.vcf
```

**b. Tối ưu hóa hiệu năng I/O cho luồng dữ liệu khổng lồ:**
Khi xử lý các ma trận hệ gen hàng chục Gigabyte, thuật toán `sort` sẽ sinh ra hàng ngàn tập tin tạm tại thư mục lõi `/tmp`. Nếu thư mục hệ thống này cạn kiệt dung lượng, tiến trình sẽ sập (Crash). Giải pháp kỹ thuật là dùng tham số `-T` để chuyển hướng thư mục tạm sang ổ đĩa cứng lưu trữ, và `--parallel` để huy động đa luồng vi xử lý.
```bash
# Huy động 8 lõi CPU xử lý song song và dời thư mục bộ đệm tạm sang phân vùng lưu trữ lớn
sort --parallel=8 -T /mnt/data/temp_dir -k1,1 -k2,2n huge_database.bed > sorted.bed
```
