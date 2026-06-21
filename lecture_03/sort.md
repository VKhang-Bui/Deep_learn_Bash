### 1. Bản chất cốt lõi
Lệnh `sort` làm nhiệm vụ đọc văn bản và sắp xếp lại toàn bộ các dòng theo thứ tự từ điển (A-Z).

**Bí mật bộ nhớ:**
Khác với `grep` hay `cut` là các công cụ Stream Processing (đọc và ném đi từng dòng một, cực tiết kiệm RAM), lệnh `sort` **BẮT BUỘC phải ngậm TOÀN BỘ dữ liệu vào RAM** rồi mới bắt đầu sort. Lấy ví dụ: Làm sao bạn biết ai cao nhất lớp nếu bạn chưa đo hết tất cả học sinh?

*Câu hỏi đặt ra:* Nếu bạn sắp xếp một file FASTQ 500GB trên máy tính chỉ có 8GB RAM, máy có sập không? 
*Trả lời:* Rất may là Không! Lệnh `sort` sở hữu một thuật toán gọi là **External Merge Sort**. Khi nó thấy RAM sắp đầy, nó tự động chặt dữ liệu đang giữ ra thành các file nhỏ, nhét tạm vào ổ cứng (vào thư mục `/tmp`), sắp xếp từng mẩu đó rồi gộp lại. 
*Hệ quả:* `sort` cực kỳ an toàn cho RAM nhưng lại **rất tốn thời gian chạy** và "ăn" tốc độ ổ cứng cực mạnh.

---

### 2. Các tùy chọn (Options)
Mặc định `sort` sắp xếp mọi thứ theo kiểu "Chữ cái" (Lexicographical). 

**A. `-n` (Numeric - Sắp xếp theo số học)**
*Cạm bẫy:* Mặc định, nó coi chuỗi `"10"` đứng trước chuỗi `"2"` (vì chữ số 1 đứng trước chữ số 2 trong bảng chữ cái). 
Nếu bạn muốn sort các con số tọa độ Gen thực sự, bạn BẮT BUỘC phải dùng `-n` để đánh thức tư duy toán học của `sort`.

**B. `-r` (Reverse - Đảo ngược chiều)**
Sắp xếp từ Z đến A, hoặc từ Cao xuống Thấp.
*Ví dụ thực chiến:* Đường ống `sort -nr | head` là combo kinh điển để tìm ra những con số lớn nhất xếp trên cùng.

**C. `-V` (Version)**

Khi bạn có danh sách Nhiễm sắc thể: `chr1, chr2, chr10, chr22`.
- Lệnh `sort` thường mặc định sẽ xếp: `chr1` $\rightarrow$ `chr10` $\rightarrow$ `chr2` $\rightarrow$ `chr22`. (Cực kỳ thảm họa khi đem vẽ biểu đồ hệ gen!).
- Lệnh `sort -V` sẽ nhận diện thông minh các số ở phần đuôi chữ để xếp chuẩn khoa học: `chr1` $\rightarrow$ `chr2` $\rightarrow$ `chr10` $\rightarrow$ `chr22`.

---

### 3. Sắp xếp theo Cột: Bộ đôi `-t` và `-k`
Đây là lúc `sort` bắt tay làm việc chung với dữ liệu dạng Bảng.

- **`-t` (Tương đương `-d` của `cut`):** Khai báo ký tự ranh giới cột (Ví dụ `-t ','` cho file CSV).
  *Khác biệt:* Mặc định của `sort` thông minh hơn `cut`! Nếu không có `-t`, `sort` tự động gom tất tần tật khoảng trắng (dấu cách/Tab liền nhau) thành một ranh giới cột duy nhất.
- **`-k` (Key - Chìa khóa sắp xếp):** Ra lệnh sắp xếp dựa trên cột nào?

**CÚ PHÁP `-k`:**
Rất nhiều chuyên gia nhầm lẫn khi gõ: `sort -k 2` để sắp xếp theo cột 2.
**Bản chất thực sự:** `sort -k 2` có nghĩa là *"Lấy từ cột 2 VÀ KÉO DÀI ĐẾN TẬN CUỐI DÒNG để làm chìa khóa sắp xếp"*. Nếu nội dung cột 2 bị trùng nhau, nó lấy luôn cột 3 ra đọ sức, làm hỏng hoàn toàn logic tính toán.
**Viết chuẩn mực chuyên nghiệp:** `sort -k 2,2` (Nghĩa là: Bắt đầu lấy chìa khóa từ cột 2, và DỪNG LẠI ở ngay cột 2).

*Bổ trợ:* Bash cho phép bạn nhét thẳng Option `-n` hay `-r` vào đuôi của khai báo Cột!
Ví dụ: `-k 2,2n` (Sắp xếp theo Cột 2, và ép Cột 2 phải đối xử như Số học).

---

### 4. Ứng dụng và Đường ống (Pipeline)

**Ví dụ 1: Sắp xếp file BED theo chuẩn của công cụ `bedtools`**
File BED luôn có Cột 1 (Tên NST) và Cột 2 (Tọa độ bắt đầu).
Hầu hết các phần mềm phân tích chuyên sâu yêu cầu file BED của bạn phải được sắp xếp chuẩn xác: Ưu tiên 1 là Tên NST (Dùng Version), Ưu tiên 2 là Tọa độ (Dùng Số học).
```bash
# -k 1,1V : Sắp xếp cột 1 theo Version (chr1, chr2, chr10...)
# -k 2,2n : Nếu cột 1 trùng nhau, tiếp tục phân định thắng thua bằng cột 2 theo Số học.
sort -k 1,1V -k 2,2n capture_regions.bed > sorted_regions.bed
```

**Ví dụ 2: Tìm 3 trình tự gen dài nhất**
Giả sử file `lengths.txt` chứa tên Gene ở Cột 1 và Chiều dài ở Cột 2 (phân cách bằng dấu cách). Làm sao để tìm ra Top 3 Gene dài nhất?
```bash
# Sắp xếp theo cột 2, bằng số (n), đảo ngược từ lớn đến bé (r)
# Sau đó đẩy luồng qua head để bóc lấy đúng 3 dòng đầu
sort -k 2,2nr lengths.txt | head -n 3
```

**Ví dụ 3: Cứu hộ máy chủ (Dành cho file khổng lồ)**
Nếu bạn biết file VCF của mình lớn tới 1 Terabyte, và bạn biết ổ `C:` hoặc thư mục `/tmp` của server sắp hết dung lượng. Bạn muốn ép `sort` đẩy các "file chặt nhỏ tạm thời" ra một thư mục ổ cứng khác rộng rãi hơn (như `/data/temp`) để tránh làm sập server:
```bash
# Tùy chọn -T (Temp) chỉ định nơi cất giấu dữ liệu tạm thời
sort -T /data/temp -k 1,1V variant_khong_lo.vcf > sorted.vcf
```
