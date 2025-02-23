---
layout: post
title: "Pentest 101 - Ghi chép về SQL Injection (SQLi),"
categories: job
tags: ['Security']
image: assets/img/2025/02/12/0-sql-injection-intro.png
---

SQL Injection là một kỹ thuật tấn công đã quá quen thuộc, không cần phải giới thiệu nhiều. Vì vậy, hôm nay chúng ta sẽ không đi sâu vào những khái niệm cơ bản mà sẽ tập trung vào việc điểm lại những khía cạnh quan trọng nhất của nó. 

# Injection - Lớp lỗ hổng quen thuộc,

SQL Injection (SQLi) là một dạng cụ thể của tấn công Injection. Các lỗ hổng Injection về cơ bản là chung một nguyên lý như sau:

>Lợi dụng các cú pháp, quy tắc “từ điển” có sẵn để thao túng hành vi của đối tượng xử lý instruction

=> Rõ ràng Attacker không tạo ra thứ gì mới lạ, mà chỉ lợi dụng quy tắc có sẵn để "thao túng" việc xử lý theo ý đồ mà Attacker mong muốn.

# Phân loại các kỹ thuật trong SQLi

Để khai thác lỗ hồng SQLi một số Blogger trên mạng tổng hợp lại các kỹ thuật chính hay được sử dụng như sau:

![sqli technical]({{site.url}}/assets/img/2025/02/12/1-sql-injection-technical.png)

>Tuy nhiên cần phải hiểu rằng các kỹ thuật này có thể sử dụng riêng lẻ hoặc kết hợp lẫn nhau. Cùng một vị trí mắc lỗi SQLi có thể có nhiều cách khai thác sử dụng các kỹ thuật khác nhau. Không có một quy tắc nào cố định cả, tất cả tùy thuộc vào độ sáng tạo của người áp dụng.

# In-band SQLi

Như cái tên của nó In-band SQLi có nghĩa là Attacker sử dụng cùng một kênh giao tiếp để khai thác lỗ hổng và nhận kết quả. Ví dụ dễ hình dung nhất: Một trang web có lỗ hổng SQL Injection, nếu Attacker có thể thực hiện truy vấn độc hại (khai thác) và nhận dữ liệu trả về trên chính trang web đó thì trường hợp này gọi là In-band SQLi. Với In-band có 2 kỹ thuật hay được sử dụng để khai thác là **Error Base** và **UNION Base**

## Error Base

Lợi dụng thông báo lỗi để nhận diện lỗi hoặc khai thác thông tin trong CSDL (Lấy thông tin về loại DB, Kiểu DB, Tên DB, Tên bảng và dữ liệu trong DB ...) và không làm thay đổi dữ liệu trong DB.

Ví dụ:

* ***VD dùng Error Base để nhận diện lỗi***: Hình anh bên dưới là thứ đã quá quen thuộc với những ai đã từng học PHP-MySQL. Đây là thông báo đặc trưng để phát hiện Ứng dụng chứa lỗi SQLi và Kiểu DB là MySQL.

![sqli technical error base1]({{site.url}}/assets/img/2025/02/12/2-sql-error-base1.png)

* ***VD dùng Error Base để khai thác/truy dữ liệu DB:*** Ứng dụng nhận đối số **input** từ user, không làm sạch dữ liệu. Phía dưới được xử lý bằng câu lệnh truy vấn như sau:

```SELECT * FROM article WHERE title='$input';```

Với cấu trúc bảng:

```sql
    CREATE TABLE article (
        id INT AUTO_INCREMENT PRIMARY KEY,
        title VARCHAR(255),
        content TEXT
    ); 
```

Nếu Input nhận vào Payload khai thác như sau ```' AND EXTRACTVALUE(1, CONCAT(0x7e, (SELECT database()))) -- ```. Nếu lỗi (Error) được hiển thị trên page sẽ là: ```XPATH syntax error: '~mydb'```

Giải thích từng phần của Payload:

![sqli technical error base2]({{site.url}}/assets/img/2025/02/12/3-sql-injection-error-base2.png)


Practice: Thử suy nghĩ xem muốn lấy danh sách các bảng trong DB và truy vấn thông tin trong một bảng thì làm thế nào :)

>Note: Với kỹ thuật này có 2 hàm rất hay được sử dụng khi khai thác Error base SQLi với MySQL là ExtractValue() hoặc UpdateXML()  ⇒ Ghi nhớ điều này sẽ sử dụng sau này.

## UNION Base

Câu lệnh UNION là thứ không quá "xa lạ" với ai đã từng làm việc với SQL

![sqli technical union]({{site.url}}/assets/img/2025/02/12/3-sql-injection-union.png)

Kỹ thuật UNION Base sử dụng toán tử SQL UNION kết hợp với SELECT để trả về dữ liệu bổ sung trên trang. Đây là phương pháp phổ biến nhất để trích xuất dữ liệu trái phép qua lỗ hổng SQL Injection.

Quay lại với ví dụ: Ứng dụng nhận đối số **input** từ user, không làm sạch dữ liệu. Phía dưới được xử lý bằng câu lệnh truy vấn như sau:

```select * from article where id = '$input';``` => Hiển thị nội dung chi tiết của một bài viết trên trang.

![sqli technical error base3]({{site.url}}/assets/img/2025/02/12/3-sql-error-base2.png)

Với cấu trúc bảng trong CSDL:

```sql
    CREATE TABLE article (
        id INT AUTO_INCREMENT PRIMARY KEY,
        title VARCHAR(255),
        content TEXT
    ); 
```
Giả sử web vẫn bật tính năng hiển thị lỗi (Error). Do vậy khi query với câu lệnh sai cú pháp (Số cột lấy từ lệnh SELECT sau UNION không bằng số cột lấy từ lệnh trước UNION) thì lỗi vẫn hiển thị.

Payload: ```1 UNION SELECT 1```

![sqli technical error base4]({{site.url}}/assets/img/2025/02/12/3-sqli-error-base4.png)

Lỗi chỉ hết khi số lượng cột sau UNION bằng số cột trong câu lệnh SELECT ban đầu để câu lệnh SQL không bị lỗi cú pháp => Ta sẽ xác định được số cột cần thiết sau UNION.

Payload: ```1 UNION SELECT 1,2,3```

![sqli technical error base5]({{site.url}}/assets/img/2025/02/12/3-sql-injection-error-base5.png)

Cố gắng xác định vị trí hiển thị của dữ liệu lấy từ UNION

Payload: ```0 UNION SELECT 1,2,3```

![sqli technical error base6]({{site.url}}/assets/img/2025/02/12/3-sql-injection-error-base6.png)

=> Ta có thể xác định vị trí hiển thị của từng phần của Payload với các giá trị 1,2,3.

Tiếp tục lấy tên của CSDL

Payload: ```0 UNION SELECT 1,2,database()```

![sqli technical error base7]({{site.url}}/assets/img/2025/02/12/3-sql-injection-error-base7.png)

Tiếp tục không khó để lấy tên các bảng của CSDL bằng một số Query sau:

```sql
0 UNION SELECT 1,2,group_concat(table_name) FROM information_schema.tables WHERE table_schema = 'sqli_one'
0 UNION SELECT 1,2,group_concat(column_name) FROM information_schema.columns WHERE table_name = 'staff_users'
0 UNION SELECT 1,2,group_concat(username,':',password SEPARATOR '<br>') FROM staff_users
```

Quá lợi hại phải không các bạn.

# Inferential/Blind SQLi

Inferential SQL Injection (Có người còn gọi là Blind SQLi) là kỹ thuật tấn công SQL Injection trong đó payload vẫn được thực thi theo đúng ý Attacker nhưng Attacker không nhận được phản hồi trực tiếp từ hệ thống (Blind ~ mù) do kết quả của việc thực thi hoặc lỗi bị ẩn hoặc bị tắt Disabed. Tuy nhiên, bằng cách quan sát các thay đổi trong hành vi ứng dụng, kẻ tấn công vẫn có thể suy luận và trích xuất dữ liệu từ database.

Với Blind SQLi có 2 kỹ thuật hay được sử dụng là: Authentication Bypass và Boolean Based.

## Authentication Bypass

Authentication Bypass là một case cơ bản của Blind SQLi. Với case này đơn giản chỉ cần bypass được form login để vượt qua cơ chế xác thực mà chả cần khai thác thông tin gì của DB cả. 

Thông thường user cần cấp username và password để dưới ứng dụng build lên query sau:

```select * from users where username='%username%' and password='%password%' LIMIT 1;```

Nếu query trả lại bản ghi khác rỗng quá trình logon thành công ngược lại thất bại. Với case này thay payload vào ô password với nội dung: ```' OR 1=1;--``` để câu lệnh truy vấn check password thành:

```select * from users where username='' and password='' OR 1=1;```

Vì ```1=1`` là một điều kiện luôn TRUE và chúng ta đã sử dụng toán tử OR nên làm biểu thức logic sau WHERE luôn là TRUE. Truy vấn sẽ luôn trả về giá trị khác rỗng, thỏa mãn logic của ứng dụng web rằng database đã tìm thấy một cặp username/password hợp lệ, dẫn đến việc cho phép truy cập.

Khá dễ hiểu phải không, đây cũng là ví dụ kinh điển hay gặp mà các blogger hay sử dụng để giới thiệu về SQLi.

## Boolean Base

Boolean SQLi được sử dụng khi chỉ có 2 giá trị trả lại. Hai giá trị có thể là True - False, 0 - 1, yes - no, Thành công - thất bại, thậm trí có lỗi - không có lỗi cũng được coi là một dạng boolean (Nhìn về tổng quát error based cũng có thể sử dụng như dạng biến thể của Boolean), ...

Kẻ tấn công gửi một truy vấn SQL đến cơ sở dữ liệu, buộc ứng dụng trả về kết quả khác nhau dựa trên điều kiện đúng hoặc sai từ đó dò dần dữ liệu. (Ai đã từng học thuật toán tìm kiếm chia đôi thì sẽ thấy nó tương tự như vậy).

**Ví dụ:** Giả sử có ứng dụng có hàm kiểm tra sự tồn tại của một username với username nhận vào từ người dùng. Dưới ứng dụng xử lý bằng câu lệnh sql như sau:

```sql
SELECT * FROM users WHERE username = '%username%' LIMIT 1;
```

* Nếu username tồn tại câu lệnh trả lại true trên ứng dụng web. Ngược lại trả lại false.

![sqli technical boolean]({{site.url}}/assets/img/2025/02/12/4-sql-injection-boolean1.png)

* Thử với payload ```' OR 1=1 --``` thì trả lại true

* Thử với payload ```' OR 0=1 --``` thì trả lại false

=> Ta thấy rằng ứng dụng trả lại hai giá trị **true** và **flase**. Giá trị này được quyết định bởi biểu thức logic sau mệnh đề WHERE trong câu truy vấn SQL. Cụ thể 

* Nếu biểu thức logic sau WHERE có giá trị TRUE, ứng dụng trả về **true**.

* Nếu biểu thức logic sau WHERE có giá trị FALSE, ứng dụng trả về **false**.

Hiện tượng này cho thấy ứng dụng có thể tồn tại lỗ hổng SQLi. Đặc biệt, chúng ta có thể khai thác lỗ hổng này bằng kỹ thuật Boolean-based SQL injection. Lý do: Kỹ thuật Boolean-based SQL injection cho phép chúng ta kiểm soát giá trị logic của biểu thức sau WHERE thông qua dữ liệu đầu vào được cung cấp cho biến %username%. Dựa vào kết quả true hoặc false trả về, chúng ta có thể suy luận và từng bước thu thập thông tin về cơ sở dữ liệu.

*Quay lại với ứng dụng bên trên, áp dung kỹ thuật Boolean ta thực hiện như sau:*

Thử chạy biểu thức logic để xác định độ dài của password

```' OR (SELECT LENGTH(password) FROM users WHERE username='admin') = 8 --```

Do vừa kết luận ở trên là nếu biểu thức logic sau WHERE là TRUE thì ứng dụng trả lại true ngược lại sẽ trả lại false => Nếu câu lệnh query là đúng <=> ứng dụng trả lại là true thì ta có thể xác định được độ dài password là 8. Ngược lại ứng dụng trả lại là false thì độ dài password khác 8. Nhưng mà khi đã có nhận định này ta chỉ cần chạy loop 1 vòng lặp từ 1-100 là dễ dàng xác định được độ dài password mà :) (Khi kết quả trả lại true).

Sau đó tiếp tục kiểm tra xem ký tự đầu tiên của password có phải là a hay không bằng biểu thức logic sau:

```' OR (SELECT SUBSTRING(password,1,1) FROM users WHERE username='admin') = 'a' --```

Thử với các giá trị 'b', 'c', ... cho đến khi tìm được ký tự đúng.
Lặp lại với các vị trí 2, 3, ... cho đến khi lấy được toàn bộ mật khẩu.

>Tóm lại, với Boolean-based SQL Injection, chúng ta sẽ dần dần dò tìm thông tin bằng cách thay đổi giá trị của biểu thức logic trong truy vấn SQL. Dựa vào phản hồi của hệ thống (đúng hoặc sai), kẻ tấn công có thể suy luận và trích xuất dữ liệu một cách có hệ thống.

## Time Base

Một trong những kỹ thuật phổ biến được sử dụng trong Blind SQLi là dựa trên thời gian (time-based). Attacker sẽ gửi một truy vấn SQL đến cơ sở dữ liệu. Truy vấn này được thiết kế để gây ra một độ trễ nhất định trong phản hồi của ứng dụng nếu một điều kiện cụ thể là đúng. Bằng cách đo thời gian phản hồi, Attacker có thể suy luận xem điều kiện đó là đúng hay sai.

Với kỹ thuật này thông thường Attacker sử dụng hàm SLEEP(x) để thực hiện việc này.

Ví dụ: Giả sử khi truy cập vào dường dẫn https://website.thm/analytics?referrer=tryhackme.com ứng dụng sẽ lấy toàn bộ thông tin log tracking liên quan tới domain tryhackme.com bằng câu lệnh truy vấn sau để trả lại (Dĩ nhiên không làm sạch giá trị của %domain%)

```select * from analytics_referrers where domain='tryhackme.com' LIMIT 1```

Thử một số payload thay vào %domain% như bên dưới và thời gian phản hồi tương ứng có thay đổi

![sqli technical time base]({{site.url}}/assets/img/2025/02/12/5-sql-injection-time-base.png)

Lý giải cho sự khác biệt về thời gian thực thi như sau: Ta có thể nhận ra rằng nếu câu lệnh Query đúng cú pháp thì trang web sẽ trả lời hơi trễ ~sau 5s (Do câu lệnh SLEEP được thực thi) còn lại gần như trả lời tức thì => Ta có thể xác định được số cột của bảng analytics_referrers. Hay đấy nhưng bây giờ làm gì hay ho hơn chút đi.

```sql
admin123' UNION SELECT SLEEP(5),2 where database() like 'u%';
admin123' UNION SELECT SLEEP(5),2 where database() like 'sqli_f%';--
```

# Outband SQLi

Out-of-Band SQLi là kỹ thuật không phổ biến vì nó phụ thuộc vào việc bật các tính năng cụ thể trên máy chủ cơ sở dữ liệu hoặc vào logic của ứng dụng web để có thể thực hiện một lời gọi ra  mạng bên ngoài dựa trên kết quả truy vấn SQL.

Loại tấn công này sử dụng hai kênh giao tiếp riêng biệt: một để thực hiện tấn công (ví dụ: gửi yêu cầu web) và một để thu thập dữ liệu (theo dõi các yêu cầu HTTP/DNS gửi đến dịch vụ do kẻ tấn công kiểm soát).

