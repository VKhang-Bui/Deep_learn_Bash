# TỪ ĐIỂN THUẬT NGỮ (GLOSSARY)

File này là nơi lưu trữ và giải thích chuyên sâu các thuật ngữ cốt lõi (Jargon) thường gặp khi bạn học lập trình Bash Script, Linux và Tin Sinh Học.

---

### 1. RegEx (Regular Expression - Biểu thức chính quy)

**Khái niệm cơ bản:**
RegEx là một "ngôn ngữ ký hiệu" đặc biệt dùng để mô tả và nhận diện các khuôn mẫu (pattern) ẩn bên trong văn bản. Thay vì tìm kiếm một từ khóa cố định như `"Lỗi hệ thống"`, RegEx cho phép bạn tìm kiếm theo quy luật logic trừu tượng, ví dụ: *"Tìm dòng có chứa từ Lỗi, theo sau là 3 con số bất kỳ, và kết thúc bằng chữ A"*.

**Bản chất (Chiếc chìa khóa vạn năng):**
Nếu ví dữ liệu văn bản là những ổ khóa khổng lồ mang vô vàn hình thù, thì đoạn mã RegEx chính là bản vẽ "chế tạo chìa khóa vạn năng". Bạn sẽ lắp ráp các ký hiệu toán học logic (như `^`, `$`, `*`, `+`, `[A-Z]`) thành các rãnh chìa khóa vừa vặn, khớp chính xác vào khối lượng dữ liệu khổng lồ để lấy ra đúng thứ bạn cần.

*Ứng dụng trong Tin Sinh Học:* Đoạn mã `^ATG([ATGC]{3})*TAA$` chính là một chiếc "chìa khóa RegEx" được tạc ra để tự động dò tìm và bắt lấy các trình tự mã hóa gen (bắt đầu bằng mã bộ ba ATG, ở giữa là các bộ ba Nu bất kỳ, và kết thúc bằng mã dừng TAA).

**Quyền lực thực sự:**
Trong xử lý Dữ liệu lớn (Big Data), RegEx cho phép bạn gom hàng ngàn dòng code `if/else` kiểm tra từng ký tự lằng nhằng gom lại thành đúng **một dòng thần chú** ngắn gọn.

---

### 2. RegEx Engine (Động cơ Biểu thức chính quy)

**Khái niệm cơ bản:**
Bạn có thể đã nghe nhiều về "RegEx" (Biểu thức chính quy - tức là cú pháp dùng để tìm kiếm chuỗi). Tuy nhiên, **RegEx Engine** lại là một khái niệm sâu hơn. Nó chính là "BỘ NÃO" (phần mềm hoặc thuật toán ẩn bên trong các công cụ) làm nhiệm vụ đọc hiểu cái cú pháp RegEx của bạn, sau đó mang đi quét qua văn bản để tính toán xem có khớp hay không.

**Bản chất (Tại sao lại gọi là "Engine - Động cơ"?):**
Hãy tưởng tượng chuỗi RegEx (ví dụ: `^ATG.*TAA$`) giống như một "Bản vẽ thiết kế". Tự thân bản vẽ thì không thể tạo ra sản phẩm. Bạn cần một nhà máy (Động cơ) đọc hiểu bản vẽ đó và sản xuất ra thứ bạn muốn. 
Tương tự, chuỗi RegEx chỉ là những ký tự vô tri. **RegEx Engine** là đoạn mã lập trình thực sự bên dưới (dựa trên thuật toán Toán học gọi là Cỗ máy trạng thái - Automata) để chạy dọc theo từng chữ cái trong file dữ liệu của bạn.

**Tại sao bạn cần quan tâm đến nó?**
Bởi vì trên thế giới KHÔNG CHỈ CÓ MỘT "động cơ" duy nhất! 
Mỗi ngôn ngữ lập trình và phần mềm lại được trang bị một Động cơ khác nhau. Dẫn đến việc cùng một đoạn mã RegEx, nhưng mang qua công cụ khác thì lại chạy sai:
1. **Động cơ POSIX (BRE & ERE)**: Là động cơ nền tảng, cổ điển, chạy trên các công cụ có sẵn của Linux như `grep`, `sed`, `awk`. (Trong bài `grep` ta đã học sự khác nhau của BRE và ERE).
2. **Động cơ PCRE (Perl Compatible Regular Expressions)**: Là động cơ hiện đại, phổ biến và mạnh mẽ hơn rất nhiều. Nó hỗ trợ các "phép thuật" cao cấp như Lookahead (Nhìn trước), Lookbehind (Nhìn lùi). PCRE được sử dụng trong ngôn ngữ Perl, Python, Java, R, và khi bạn gõ lệnh `grep -P`.

**Bài học rút ra:** 
Ngày mai, nếu bạn copy một đoạn code RegEx cực xịn dùng để lọc chuỗi gen từ Python mang sang dán vào lệnh `sed` hoặc `grep` cơ bản mà thấy nó không hoạt động, thì hãy nhớ ngay đến khái niệm này. Lỗi không phải do bạn sai, mà do hai công cụ đó đang chạy bằng hai **RegEx Engine** có "hệ tư tưởng" khác nhau!

---
*(Các thuật ngữ tiếp theo sẽ được cập nhật thêm tại đây...)*
