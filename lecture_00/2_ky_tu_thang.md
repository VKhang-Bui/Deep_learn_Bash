# Giải mã Ký tự Thăng `#` trong Bash

Ký tự `#` (Pound/Hash) là một trong những ký hiệu đa nhiệm và có quyền lực bậc nhất trong thế giới Bash Script. Đa số người mới học chỉ nghĩ `#` dùng để viết ghi chú. Nhưng khi đi sâu vào bản chất, `#` còn là "thước đo chiều dài", "bác sĩ phẫu thuật chuỗi", và là "linh hồn" của mọi bản kịch bản (script).

---

### 1. Kẻ vô hình: Ghi chú (Comments)
Chức năng cơ bản nhất: Khi đứng tự do bên ngoài, mọi thứ nằm phía sau dấu `#` cho đến hết dòng sẽ bị Bash ngó lơ (bỏ qua) hoàn toàn.
```bash
echo "Khởi chạy Pipeline..." # Đoạn chữ đằng sau này máy tính không đọc
```
**Cạm bẫy:** Nếu `#` bị kẹp bên trong dấu ngoặc kép `" "` hoặc ngoặc đơn `' '`, nó mất đi quyền năng tàng hình và biến thành một ký tự văn bản bình thường.
```bash
echo "Ký tự # này sẽ được in ra màn hình bình thường"
```

---

### 2. Linh hồn của Script: Dấu Shebang `#!`
Khi bạn viết một file kịch bản hoàn chỉnh (ví dụ `pipeline.sh`), dòng **ĐẦU TIÊN** của file đó luôn luôn phải là:
```bash
#!/bin/bash
```
Đây là một **ngoại lệ siêu đặc biệt**. Khi hai ký tự `#!` đi liền nhau ở dòng số 1, nó được gọi là dấu **Shebang**. 
*Bản chất:* Mặc dù bản thân ngôn ngữ Bash coi đây là dòng ghi chú, nhưng **Hệ điều hành cốt lõi (Linux)** lại đọc dòng này đầu tiên để biết phải lấy "động cơ" nào ra để chạy đoạn code phía dưới (Ở đây là mượn động cơ `/bin/bash` ra chạy). Nếu bạn viết file Python, Shebang sẽ là `#!/usr/bin/env python3`.

---

### 3. Chiếc thước đo: Tính Kích thước & Số lượng
Khi `#` được kẹp vào bên trong ngoặc gọi biến `$`, nó biến hình thành một "chiếc thước đo" chiều dài. Cú pháp này vô cùng phổ biến trong các script Tin Sinh Học.

**A. Đếm độ dài của Chuỗi: `${#TÊN_BIẾN}`**
Giả sử bạn có một đoạn chuỗi DNA và muốn đếm xem trình tự này có bao nhiêu Nucleotide?
```bash
SEQ="ATGCATGC"
echo ${#SEQ}  # Cỗ máy sẽ đo và in ra số 8
```

**B. Đếm số phần tử trong Mảng (Array): `${#MANG[@]}`**
Bạn có một danh sách mảng chứa tên của 100 mẫu bệnh phẩm. Làm sao để đếm nhanh số lượng mẫu trong mảng?
```bash
SAMPLES=("Mẫu_A" "Mẫu_B" "Mẫu_C")
echo ${#SAMPLES[@]}  # Sẽ in ra con số 3
```

**C. Lấy Tổng số Tham số đầu vào: `$#`**
Khi bạn viết một phần mềm (ví dụ cách gọi lệnh là: `./chay_pipeline.sh sample_R1.fq sample_R2.fq`), làm sao phần mềm tự biết bạn đã cung cấp cho nó mấy file? Ký hiệu `$#` được sinh ra chính là để lưu trữ cái "số lượng file" đó.
```bash
# Lệnh kiểm tra đầu vào kinh điển:
if [ "$#" -ne 2 ]; then
    echo "LỖI: Bạn phải cung cấp đúng 2 file R1 và R2!"
    exit 1
fi
```

---

### 4. Bác sĩ phẫu thuật: Cắt bỏ Tiền tố (Prefix Trimming)
Đây là tính năng thuộc hàng "Master" ít người để ý. Dấu `#` có thể gọt bỏ một phần chuỗi chữ **tính từ bên trái sang**.

Ví dụ: Bạn có biến lưu đường dẫn file `DIR="/home/admin/data/sample.vcf"`. Bạn muốn tự động cắt đi chữ `/home`?
```bash
DIR="/home/admin/data/sample.vcf"
# Cấu trúc: ${Tên_Biến#Phần_cần_cắt_bỏ}
echo ${DIR#/*/}  
# Kết quả gọt được: admin/data/sample.vcf
```
*(Ghi chú: Để cắt phần đuôi (Suffix) từ bên phải sang như gọt bỏ `.vcf` thành `.fastq`, người ta dùng dấu `%`. Tuy nhiên ta sẽ nghiên cứu sâu kỹ thuật này ở một bài riêng về Phẫu thuật Chuỗi).*

---

### 5. Cạm bẫy với Dữ liệu thực tế (VCF, SAM)
Một lưu ý cực kỳ quan trọng: Dấu `#` mà bạn thấy dầy đặc trong các file định dạng chuẩn của Tin Sinh Học như **VCF** (Các dòng Header luôn bắt đầu bằng `##` hoặc `#CHROM`) **hoàn toàn không dính dáng gì đến lệnh ghi chú `#` của Bash**.
Đó đơn thuần chỉ là quy ước định dạng file do các nhà khoa học quy định. Do đó, khi bạn đọc file bằng lệnh `cat` hay `cut`, Bash không hề tự động ẩn các dòng đó đi. Muốn vứt bỏ các dòng Header có dấu `#` đó, bạn phải tự mình dùng lệnh `grep -v "^#"` để loại trừ chúng (giống như bài tập hôm trước bạn đã làm xuất sắc).
