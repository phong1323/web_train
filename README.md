# Web_train
**[Mở Task 1](Task1/README.md)**
- Dev 1 trang web có chức năng cơ bản là đăng ký, đăng nhập, hiển thị thông tin tài khoản (tên, giới tính, email, địa chỉ, sdt), search người dùng
- Phần client thì không cần quá đẹp
- Phần server thì chọn ngôn ngữ PHP và database MySQL
---

**[Mở Task 2](Task2/README.md)**
- Tìm hiểu chi tiết về lỗ hổng SQLI -> phân loại -> (Không bề lý thuyết nhiều quá mà chỉ nói theo ý hiểu) 
- Code web PHP (không cần CSS) chức năng đăng nhập đăng kí và kết nối với MySQL chứa lỗ hổng SQLI + khai thác để liệt kê được dữ liệu quan trọng trong MySQL đã dựng, áp dụng debug với x-debug để quan sát một số hàm filter như ‘mysqli_real_escape_string’ nếu có 
- Clear lab SQLI trên : https://portswigger.net/web-security/all-labs 
- Lưu ý: viết Write up đầy đủ các phần DEADLINE: 24h 1/6https://portswigger.net/web-security/all-labs
---

**[Mở Task 3](Task3/README.md)**
- Viết code python tự động khai thác cho 18 lab cũ
- Làm thêm 3 lab:
https://battle.cookiearena.org/challenges/web/baby-sql-injection-to-rcehttps://battle.cookiearena.org/challenges/web/simple-blind-sql-injectionhttps://battle.cookiearena.org/challenges/web/blind-logger-middleware
- Tìm và làm ít nhất 2 bài SQL Injection trên trang:
https://www.root-me.org/
- Viết wu đầy đủ
DEADLINE: 24h 6/6/2025https://battle.cookiearena.org/challenges/web/baby-sql-injection-to-rce
---

**[Mở Task 3.2](Task3.2/README.md)**
- Làm thêm 3 lab:
https://battle.cookiearena.org/challenges/web/baby-sql-injection-to-rcehttps://battle.cookiearena.org/challenges/web/simple-blind-sql-injectionhttps://battle.cookiearena.org/challenges/web/blind-logger-middleware
---

**[Mở Task 4](Task4/README.md)**
- Tìm hiểu tất cả mọi thứ có thể tìm hiểu được về command injection (khái niệm, cách nhận biết, hướng khai thác, cách phòng tránh, phân loại theo hướng tấn công)
- Dựng lại lỗ hổng này với php hoặc bất kỳ ngôn ngữ khác và viết write up khai thác web đã dựng
- Tìm hiểu một số kĩ thuật bypass command injection phổ biến và demo tích hợp với web ở trên
- Làm 7 bài sau:
https://battle.cookiearena.org/challenges/web/nslookup-level-1https://battle.cookiearena.org/challenges/web/nslookup-level-2https://battle.cookiearena.org/challenges/web/nslookup-level-3https://battle.cookiearena.org/challenges/web/ethical-ping-pong-clubhttps://battle.cookiearena.org/challenges/web/blind-command-injectionhttps://battle.cookiearena.org/skills-path/os-command-injection/challenge/command-limit-lengthhttps://battle.cookiearena.org/skills-path/os-command-injection/challenge/time
LƯU Ý: Viết write up đầy đủ
DEADLINE: 24h 25/6https://battle.cookiearena.org/challenges/web/nslookup-level-1
---

**[Mở Task 4.2](Task4.2/README.md)**
- Làm hết 2 lab trong này: https://drive.google.com/drive/folders/1X9re2TeaQkxAG3iKfa7Atqt69-6JHEqQ?usp=sharing
- Yêu cầu: dùng Ubuntu hoặc Kali, học cái build Dockerfile để làm
- Viết write up đầy đủ
- Wu tham khảo: https://hackmd.io/@duc193/SyEGJi8syl
Hạn: 23h 26/7https://drive.google.com/drive/folders/1X9re2TeaQkxAG3iKfa7Atqt69-6JHEqQ?usp=sharing
---

**[Mở Task 5](Task5/README.md)**
- Tìm hiểu về xml file (cấu trúc, cú pháp cơ bản, thường dùng để làm gì?)
- Demo 1 chương trình web php dùng file xml để lưu trữ dữ liệu sản phẩm.
- XXE Injection, phân biệt entity, external DTD và internal DTD. Hậu quả của XXE Injection.
- Tìm hiểu chi tiết về CDATA và CDATA giúp ích gì trong XXE Injection, những ký hiệu CDATA không thể escape.
- Demo lại lỗ hổng này bằng php, cách ngăn chặn trong php(chi tiết với những hàm nào thì gây ra lỗi, hàm nào thì không gây ra lỗi, hàm nào được php viết để load file xml không xảy ra xxe).
- Các kỹ thuật bypass filter xxe thường gặp.
- Trình bày những dạng tấn công XXE qua những file thường gặp(docx, dtd, ...)
- Tìm hiểu về những dạng tấn công nhắm đến những phần khác nhau của xml có thể gặp, nguyên nhân, cách phòng tránh.
- CLEAR LAB: https://portswigger.net/web-security/all-labs#xml-external-entity-xxe-injection
---

**[Mở Task 5.2](Task5.2/README.md)**
- Làm hết các lab sau: https://dreamhack.io/wargame?category=web&page=1&difficulty=beginner&ordering=-cnt_solvers
- Root-me: SQL injection - Authentication SQL injection - Authentication - GBK SQL injection - String SQL injection - Numeric SQL Injection - Routed SQL Truncation SQL injection - Error SQL injection - Insert
DEADLINE: 23h 7/8
---

**[Mở Task 6](Task6.2/README.md)**
- Tìm hiểu về lỗ hổng File Inclusion, phân loại.
- Tìm hiểu cách hoạt động của các hàm include, require, include_once, require_once trong PHP.
- Tìm hiểu về các kĩ thuật bypass file inclusion sau: Null Byte, Double Encoding, UTF-8 Encoding, Path Truncation, Filter Bypass, Bypass allow_url_include.
- Phân biệt lỗ hổng File Inclusion và Path Traversal.
- Tìm hiểu cách hoạt động của một số php wrapper trong khai thác file inclusion
- Tìm nhiều nhất những case LFI2Rce -> demo.
- Phân biệt Reverse Shell và Bind Shell và sử dụng chúng trong bước cuối sau khi rce thành công.
- Cách ngăn chặn file inclusion
- Xây dựng lab tấn công demo cả 2 loại File Inclusion
- Làm root-me: Directory traversal, Local File Inclusion, Local File Inclusion - Double encoding, Remote File Inclusion
---

**[Mở Task 7](Task7/README.md)**
- Hậu quả của SSRF, những hàm nguy hiểu có thể gây ra ssrf (python, php, nodejs)
- Tìm hiểu về kỹ thuật ssrf dựa trên chuyển hướng.
- Phân loại ssrf, phân biệt chúng.
- Tìm hiểu về một số kỹ thuật bypass SSRF localhost, bypass black-list.
- Tìm hiểu về kỹ thuật curl Globbing trong ssrf.
- Có thể tham khảo nguồn: https://portswigger.net/web-security/ssrf, https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Server%20Side%20Request%20Forgery
- Tìm hiểu kỹ thuật DNS rebinding
- Clear lab SSRF portswigger
DEADLINE: 23h 11/9
---

**[Mở Task 8](Task8/README.md)**
- Tìm hiểu về SSTI (Khái niệm, nguyên nhân, tác hại + 1 số cách bypass, cách phòng tránh)
- Giải hết các bài SSTI trên rootme
DEADLINE:
23h 21/9
