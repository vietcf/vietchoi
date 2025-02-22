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

# Inferentail/Blind SQLi

Inferentail SQL Injection (Có người còn gọi là Blind SQLi) là kỹ thuật tấn công SQL Injection trong đó payload vẫn được thực thi theo đúng ý Attacker nhưng Attacker không nhận được phản hồi trực tiếp từ hệ thống (Blind ~ mù) do kết quả của việc thực thi hoặc lỗi bị ẩn hoặc bị tắt Disabed. Tuy nhiên, bằng cách quan sát các thay đổi trong hành vi ứng dụng, kẻ tấn công vẫn có thể suy luận và trích xuất dữ liệu từ database.

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



## Time Base

# Outband SQLi


