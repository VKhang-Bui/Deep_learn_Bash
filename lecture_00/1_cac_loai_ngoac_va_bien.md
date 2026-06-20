# LECTURE 0: "NGỮ PHÁP" BÍ MẬT CỦA BASH SCRIPT

Trước khi học các "Động từ" (như `cat`, `grep`, `cut`), bạn phải nắm được "Ngữ pháp" cốt lõi của Bash. Đây chính là các ký tự phép thuật `()`, `[]`, `{}` và hệ thống Biến. Không hiểu chúng, bạn sẽ không bao giờ hiểu tại sao có lúc gõ `()` lúc lại `$(())`.

---

### 1. Biến (Variables) và Dấu Ngoặc Nhọn `{}`

**A. Biến thông thường**
- **Cú pháp gán:** `TÊN_BIEN="Giá trị"` (Tối kỵ: TUYỆT ĐỐI KHÔNG được có dấu cách ở hai bên dấu bằng `=`. Gõ `A = 5` là lỗi ngay!).
- **Cú pháp gọi:** Sử dụng dấu `$` ở trước. 
```bash
SAMPLE="Patient_01"
echo "Đang phân tích mẫu $SAMPLE"
```

**B. Sự lợi hại của Dấu ngoặc nhọn `{}` trong gọi Biến**
Giả sử bạn muốn tạo tên file là `Patient_01_R1.fastq` từ biến `$SAMPLE` ở trên.
```bash
# Lỗi: Bash sẽ đi tìm cái biến tên là $SAMPLE_R1 (không tồn tại)
echo "$SAMPLE_R1.fastq" 

# Đúng: Dấu {} "bảo vệ" giới hạn của tên biến
echo "${SAMPLE}_R1.fastq"
```
*Bài học:* Khi gọi biến mà có dính liền với các chữ cái khác, hãy luôn bọc nó trong `${}`.

---

### 2. Biến Mảng (Arrays)

Trong Tin sinh học, bạn hiếm khi chạy 1 mẫu, mà thường chạy 100 mẫu. Mảng sinh ra để chứa danh sách đó.
- **Khai báo mảng:** Dùng ngoặc tròn `()`, các phần tử cách nhau bằng dấu cách.
```bash
SAMPLES=("Mẫu_A" "Mẫu_B" "Mẫu_C")
```
- **Truy cập phần tử:** (Lưu ý: Bash bắt đầu đếm từ `0`)
```bash
echo ${SAMPLES[0]}  # In ra: Mẫu_A
echo ${SAMPLES[1]}  # In ra: Mẫu_B
```
- **Lấy TOÀN BỘ mảng (Cực kỳ quan trọng để viết vòng lặp):** Dùng ký hiệu `@`
```bash
echo ${SAMPLES[@]}  # In ra: Mẫu_A Mẫu_B Mẫu_C
```

---

### 3. "Bắt" lệnh bỏ vào túi: Dấu `$()` (Command Substitution)

Bình thường bạn gõ `date`, máy in giờ ra màn hình rồi biến mất. Làm sao để "hứng" cái giờ đó lưu vào một biến?
Dùng dấu `$()` (Bên trong là một câu lệnh):
```bash
HOM_NAY=$(date)
echo "Hôm nay là: $HOM_NAY"

# Đây chính là tuyệt chiêu ở Bài kiểm tra số 1:
SO_DONG=$(wc -l < file.txt)
```

---

### 4. Cỗ máy Làm Toán: Dấu `$(( ))` (Arithmetic Expansion)

Bạn có nhận ra điểm khác biệt không? Cỗ máy chạy lệnh là `$()`, còn cỗ máy làm toán có tới 2 dấu ngoặc tròn `$(( ))`.

Bash bản chất là một kẻ "mù chữ toán học". Nếu bạn gõ:
```bash
A=5
B=10
echo $A + $B
# Kết quả in ra màn hình: 5 + 10 (Nó coi dấu cộng là một ký tự chữ bình thường!)
```
Để ép Bash phải giải bài toán, bạn BẮT BUỘC phải ném nó vào cỗ máy `$(( ))`:
```bash
echo $((A + B))
# Kết quả in ra: 15
```
*(Đây chính là bí mật đằng sau phép tính đếm reads siêu tốc `reads=$(($(wc -l < sample.fq) / 4))` ở Bài 1. Tôi dùng `$()` để chạy lệnh đếm ra con số, rồi đưa con số đó vào `$(( ... / 4 ))` để làm phép chia!)*

---

### 5. Dấu Ngoặc Vuông `[ ]` và `[[ ]]` (Điều kiện Test)

Bạn chỉ dùng dấu ngoặc vuông khi viết câu lệnh điều kiện `if/else`. Bản chất ngoặc vuông là lệnh "Hỏi/Kiểm tra".
```bash
if [ "$SO_DONG" -eq 100 ]; then
    echo "Đúng 100 dòng"
fi
```
**Luật sinh tử:** BẮT BUỘC phải có dấu cách ở bên trong hai lề của ngoặc vuông. 
- Gõ `["$A" == "$B"]` $\rightarrow$ LỖI NGAY.
- Gõ `[ "$A" == "$B" ]` $\rightarrow$ CHUẨN.

*(Ghi chú: `[[ ]]` là phiên bản nâng cấp hiện đại hơn của `[ ]`, hỗ trợ so sánh RegEx và chống lỗi rỗng tốt hơn. Trong Bash, luôn ưu tiên dùng `[[ ]]` thay vì `[ ]`).*

---

### 6. Trình Sinh Sôi Nảy Nở: Brace Expansion `{}`

Đừng nhầm với `${}` (gọi biến) ở phần 1. Dấu ngoặc nhọn đứng một mình có khả năng tự động sinh chuỗi hàng loạt cực kỳ ma thuật. Rất hay dùng để tạo file hoặc thư mục test.
- **Sinh chuỗi số liên tiếp:**
```bash
echo Sample_{1..5}
# Kết quả: Sample_1 Sample_2 Sample_3 Sample_4 Sample_5
```
- **Sinh theo danh sách:**
```bash
touch data_{R1,R2}.fastq
# Sẽ tạo ra 2 file: data_R1.fastq và data_R2.fastq
```
- **Tạo cấu trúc thư mục phức tạp trong 1 nốt nhạc:**
```bash
mkdir -p Project/{data,scripts,results}/{fastqc,mapped}
# Sẽ tạo ra hàng loạt thư mục con chéo nhau!
```

### Tóm tắt "Từ vựng Ngoặc":
1. `$VAR` hoặc `${VAR}` : Gọi Biến.
2. `$(lệnh)` : Chạy câu lệnh và bắt lấy kết quả chữ.
3. `$((toán))` : Làm phép tính cộng trừ nhân chia.
4. `[ điều kiện ]` : So sánh đúng sai cho vòng lặp If.
5. `{a,b,c}` : Sinh chuỗi hàng loạt (a b c).
