
### 1. Bản chất cốt lõi (Căn bệnh đãng trí của `uniq`)
Nhiệm vụ của `uniq` là phát hiện và loại bỏ các dòng bị trùng lặp. Tuy nhiên, nó có một **cơ chế hoạt động (Bản chất)** cực kỳ quan trọng mà bạn phải khắc cốt ghi tâm:

Cỗ máy `uniq` mắc một căn bệnh **"mất trí nhớ ngắn hạn"**.
Khác với `sort` phải nạp toàn bộ file 100GB vào RAM để nhìn bức tranh toàn cảnh, `uniq` là công cụ xử lý luồng Stream siêu tốc. Nó chỉ có khả năng nhớ được đúng **MỘT DÒNG DUY NHẤT** (chính là cái dòng nó vừa đọc trước đó). Nó dùng dòng nhớ đó để so sánh với dòng tiếp theo.

**⚠️ LUẬT SINH TỬ:** 
Bởi vì "bệnh đãng trí" này, `uniq` **CHỈ PHÁT HIỆN ĐƯỢC CÁC DÒNG TRÙNG NHAU NẾU CHÚNG ĐỨNG SÁT NHAU**. 
Nếu chữ "Gen_A" nằm ở dòng số 1 và chữ "Gen_A" lặp lại ở dòng số 1000, `uniq` sẽ coi đó là 2 gene khác biệt!
$\rightarrow$ **Quy tắc vàng:** Bạn **BẮT BUỘC phải dùng lệnh `sort`** trước để đẩy tất cả các dòng giống nhau về đứng thành từng cụm sát nhau, rồi sau đó mới được cho dữ liệu chảy qua ống `uniq`. (Ngoại trừ trường hợp hiếm hoi bạn cố tình muốn đếm chuỗi lặp lại liên tiếp kiểu "K-mer").

---

### 2. Các tùy chọn (Options) "Làm nên tên tuổi"
Nếu chạy lệnh `uniq` trần trụi không có tùy chọn, nó đơn giản là nén các dòng trùng lặp đứng cạnh nhau thành 1 dòng duy nhất. Nhưng quyền lực thực sự của nó nằm ở các cờ sau:

**A. `-c` (Count - Bộ đếm tần suất)**
Đây là tùy chọn được dùng nhiều nhất! Nó không chỉ gom các dòng trùng lại, mà còn tự động sinh ra một con số **đếm số lần xuất hiện** đính kèm ngay vào đầu mỗi dòng.
*Ứng dụng:* Đếm xem một đột biến xuất hiện bao nhiêu lần, đếm số lượng read mapping vào từng NST.

**B. `-d` (Duplicate - Lưới lọc nhân bản)**
Bình thường `uniq` sẽ in ra mọi thứ (cả dòng xuất hiện 1 lần và dòng đã được gom trùng). Nếu bạn kẹp cờ `-d`, nó sẽ **vứt bỏ** các dòng đơn lẻ, và **chỉ in ra** những dòng có "anh em sinh đôi" (bị trùng lặp từ 2 lần trở lên).
*Ứng dụng:* Bắt lỗi dữ liệu, tìm xem có bệnh nhân nào bị nhập dữ liệu xét nghiệm 2 lần không.

**C. `-u` (Unique - Kẻ độc cô cầu bại)**
Ngược lại hoàn toàn với `-d`, tùy chọn `-u` sẽ **vứt bỏ thẳng tay** bất cứ dòng nào có dấu hiệu trùng lặp. Nó chỉ ưu ái giữ lại những dòng xuất hiện đúng 1 lần duy nhất trong toàn bộ file.

---

### 3. Đường ống Thực chiến (Pipeline) trong Tin Sinh Học

**Ví dụ 1: Pipeline tìm Top 3 Barcode (Mã vạch) xuất hiện nhiều nhất**
Giả sử bạn đã dùng `grep` và `cut` bóc ra được file `barcodes.txt` chứa hàng triệu đoạn mã Barcode từ máy giải trình tự. Bạn muốn biết 3 Barcode nào dồi dào nhất.
Đây là **Đường ống 3 khúc kinh điển** mà bạn sẽ dùng đi dùng lại hàng ngàn lần:
```bash
# Khúc 1: sort để gom các barcode giống nhau tụ lại một chỗ.
# Khúc 2: uniq -c để đếm số lần (Nó sẽ tạo ra 2 cột: Cột 1 là Số đếm, Cột 2 là Barcode).
# Khúc 3: Đẩy lại vào sort -nr để sắp xếp theo Cột 1 (n) từ cao xuống thấp (r).
sort barcodes.txt | uniq -c | sort -nr | head -n 3
```

**Ví dụ 2: Bắt lỗi trùng lặp tọa độ trong file BED**
Bạn có file `regions.bed` và bạn nghi ngờ công cụ gọi biến dị (Variant Caller) bị lỗi khiến các vùng gen bị báo cáo lặp lại 2 lần. Làm sao để lôi cổ các vùng trùng lặp đó ra kiểm tra?
```bash
# Ta sort chuẩn file BED trước để gom dòng.
# Sau đó cho chảy qua uniq -d để chỉ lọc ra các dòng vi phạm luật lặp lại.
sort -k 1,1V -k 2,2n regions.bed | uniq -d
```

**Ví dụ 3: Định danh các Gene hoàn toàn dị biệt**
Bạn có danh sách các gen (đã lấy ra từ cột của file TSV). Bạn muốn tìm ra những gene đóng vai trò "Độc bản" - tức là chỉ xuất hiện ĐÚNG 1 LẦN duy nhất trong toàn bộ quần thể, những gene nào xuất hiện từ 2 cá thể trở lên sẽ bị vứt bỏ.
```bash
sort gen_list.txt | uniq -u
```
