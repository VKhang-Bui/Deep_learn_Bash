# Lệnh `join`

### 1. Bản chất thuật toán của lệnh `join`
`join` là một toán tử quan hệ (Relational Operator) hoạt động trên dòng lệnh. Nếu lệnh `paste` ghép nối hai tập tin một cách "mù quáng" theo số thứ tự dòng, thì `join` thực hiện phép ghép nối **có điều kiện**: Nó chỉ ráp nối hai bản ghi khi và chỉ khi chúng chia sẻ một giá trị khóa chung (Common Key). Cơ chế này tương đồng tuyệt đối với phép `JOIN` trong ngôn ngữ truy vấn cơ sở dữ liệu SQL.

**Cơ chế làm việc:**
Thuật toán `join` yêu cầu **hai tập tin đầu vào** đã được sắp xếp (sorted) theo cột khóa. Nó vận hành bằng kỹ thuật **Hợp nhất hai con trỏ (Two-pointer Merge)**:

1. Khởi tạo hai con trỏ đọc, mỗi con trỏ trỏ vào dòng đầu tiên của mỗi tập tin.
2. Đọc giá trị khóa tại vị trí con trỏ của cả hai tập tin.
3. **Nếu hai khóa trùng khớp:** Nối hai bản ghi lại thành một dòng duy nhất (ghép các trường của tập tin 1 với các trường của tập tin 2), xuất ra `stdout`. Tiến cả hai con trỏ.
4. **Nếu khóa tập tin 1 < khóa tập tin 2:** Bản ghi ở tập tin 1 không có cặp khớp. Mặc định, dòng này bị hủy bỏ (không xuất ra). Tiến con trỏ tập tin 1.
5. **Nếu khóa tập tin 1 > khóa tập tin 2:** Tương tự, bản ghi ở tập tin 2 bị hủy bỏ. Tiến con trỏ tập tin 2.
6. Lặp lại cho đến khi một trong hai con trỏ chạm tới EOF.

**Điều kiện tiên quyết bắt buộc:**
Cả hai tập tin đầu vào **PHẢI được sắp xếp trước** theo cùng một cột khóa, bằng cùng một thuật toán sắp xếp. Nếu vi phạm, `join` sẽ trả về kết quả thiếu bản ghi hoặc hoàn toàn sai lệch mà **không hề đưa ra cảnh báo lỗi**.

```bash
# Bước chuẩn bị bắt buộc trước khi chạy join:
sort -k1,1 file_A.txt > sorted_A.txt
sort -k1,1 file_B.txt > sorted_B.txt
join sorted_A.txt sorted_B.txt
```

### 2. Giao diện tham số kỹ thuật (Options/Flags)

**a. Chỉ định cột khóa (`-1` và `-2`):**
Mặc định, `join` sử dụng cột 1 (trường đầu tiên) của cả hai tập tin làm khóa đối sánh. Nếu cột khóa nằm ở vị trí khác, bắt buộc phải khai báo:
- **`-1 N`**: Chỉ định cột thứ `N` của **tập tin thứ nhất** làm khóa.
- **`-2 N`**: Chỉ định cột thứ `N` của **tập tin thứ hai** làm khóa.
```bash
# Đối sánh Cột 1 của file A với Cột 3 của file B
join -1 1 -2 3 sorted_A.txt sorted_B.txt
```

**b. Chỉ định ký tự phân cách trường (`-t`):**
Mặc định, `join` sử dụng khoảng trắng (Space/Tab) làm bộ phân cách. Cờ `-t` cho phép chỉ định ký tự phân cách tùy chỉnh. Ký tự này áp dụng đồng thời cho cả hai tập tin đầu vào lẫn luồng đầu ra.
```bash
# Đối sánh hai tập tin CSV phân cách bằng dấu phẩy
join -t ',' sorted_samples.csv sorted_phenotypes.csv
```

**c. Hiển thị các bản ghi không khớp (`-a`):**
Mặc định, `join` chỉ xuất ra các dòng có khóa trùng khớp ở cả hai tập tin (hành vi tương đương phép `INNER JOIN` trong SQL). Cờ `-a` mở rộng luồng đầu ra để bao gồm cả các bản ghi "mồ côi" (không tìm thấy cặp khớp):
- **`-a 1`**: In ra tất cả các bản ghi từ tập tin 1, kể cả những dòng không khớp với bất kỳ dòng nào trong tập tin 2 (tương đương `LEFT JOIN`).
- **`-a 2`**: Tương tự, nhưng cho tập tin 2 (tương đương `RIGHT JOIN`).
- **`-a 1 -a 2`**: In ra toàn bộ bản ghi từ cả hai tập tin, bất kể có khớp hay không (tương đương `FULL OUTER JOIN`).
```bash
# LEFT JOIN: Giữ lại toàn bộ mẫu trong file A, kể cả mẫu không có thông tin kiểu hình ở file B
join -a 1 sorted_samples.txt sorted_phenotypes.txt
```

**d. Định dạng luồng đầu ra (`-o`):**
Mặc định, `join` xuất ra dòng khóa chung theo sau là tất cả các trường còn lại của cả hai tập tin. Cờ `-o` cho phép kỹ sư kiểm soát chính xác trường nào được xuất ra và theo thứ tự nào. Cú pháp khai báo: `FILE_NUMBER.FIELD_NUMBER`.
```bash
# Chỉ xuất ra: Khóa chung (1.1), Cột 2 của file 1 (1.2), và Cột 4 của file 2 (2.4)
join -o 1.1,1.2,2.4 sorted_A.txt sorted_B.txt
```

**e. Giá trị thay thế cho trường rỗng (`-e`):**
Khi sử dụng `-a` (bao gồm bản ghi không khớp), các trường thuộc tập tin đối diện sẽ bị bỏ trống. Cờ `-e` cho phép chèn một chuỗi ký tự thay thế vào các vị trí trống đó. Tham số này **bắt buộc đi kèm** với `-o` để xác định cấu trúc cột đầu ra.
```bash
# FULL OUTER JOIN: Thay thế mọi ô trống bằng chuỗi "NA" (Not Available)
join -a 1 -a 2 -e "NA" -o 1.1,1.2,2.2 sorted_A.txt sorted_B.txt
```

### 3. Ứng dụng trong đường ống Tin Sinh Học (Bioinformatics Pipelines)

**a. Tích hợp dữ liệu từ nhiều nguồn độc lập (Data Integration):**
Trong phân tích hệ gen, dữ liệu thường bị phân mảnh thành nhiều tập tin riêng biệt: Một tập tin chứa danh sách Gene ID và mức biểu hiện (expression), một tập tin khác chứa Gene ID và chú thích chức năng (functional annotation). `join` cho phép hợp nhất chúng dựa trên cột Gene ID chung.
```bash
# Tập tin 1: gene_expression.tsv (Cột 1: GeneID, Cột 2: FPKM)
# Tập tin 2: gene_annotation.tsv (Cột 1: GeneID, Cột 2: Gene Name, Cột 3: Function)
# Yêu cầu: Ghép nối hai tập tin dựa trên GeneID

sort -k1,1 gene_expression.tsv > sorted_expr.tsv
sort -k1,1 gene_annotation.tsv > sorted_annot.tsv
join -t $'\t' sorted_expr.tsv sorted_annot.tsv > integrated_data.tsv
```

**b. Phát hiện mẫu thiếu dữ liệu (Missing Data Detection):**
Sử dụng kết hợp `-a` và `-e "NA"` để xác định nhanh các mẫu bệnh phẩm có dữ liệu giải trình tự nhưng thiếu thông tin kiểu hình lâm sàng (hoặc ngược lại).
```bash
# LEFT JOIN: Giữ lại toàn bộ danh sách mẫu, đánh dấu "MISSING" cho mẫu thiếu kiểu hình
sort -k1,1 sequenced_samples.txt > sorted_seq.txt
sort -k1,1 clinical_phenotypes.txt > sorted_pheno.txt
join -a 1 -e "MISSING" -o 1.1,1.2,2.2 -t $'\t' sorted_seq.txt sorted_pheno.txt
```

**c. So sánh hai danh sách kết quả (Set Comparison):**
Xác định các phần tử chung, chỉ có ở tập A, hoặc chỉ có ở tập B giữa hai danh sách Gene ID từ hai thí nghiệm khác nhau.
```bash
# INNER JOIN: Chỉ lấy các Gene ID xuất hiện ở CẢ HAI thí nghiệm
sort experiment_1_genes.txt > sorted_exp1.txt
sort experiment_2_genes.txt > sorted_exp2.txt
join sorted_exp1.txt sorted_exp2.txt > common_genes.txt
```
