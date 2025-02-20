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

=> Rõ ràng Attacker không tạo ra mới mà chỉ lợi dụng quy tắc có sẵn để thao túng xử lý theo ý của attacker.

# Phân loại các kỹ thuật trong SQLi

Để khai thác lỗ hồng SQLi một số blogger tổng hợp lại các kỹ thuật chính như sau:

![sqli technical]({{site.url}}/assets/img/2025/02/12/1-sql-injection-technical.png)

>Tuy nhiên cần phải hiểu rằng các kỹ thuật này có thể sử dụng riêng lẻ hoặc kết hợp lẫn nhau. Không có một quy tắc nào cố định cả, tất cả tùy thuộc vào độ sáng tạo của người áp dụng.

# In-band SQLi

Như cái tên của nó In-band SQLi có nghĩa là Attacker sử dụng cùng một kênh giao tiếp để khai thác lỗ hổng và nhận kết quả. Ví dụ dễ hình dung nhất: Một trang web có lỗ hổng SQL Injection, nếu Attacker có thể thực hiện truy vấn độc hại (khai thác) và nhận dữ liệu trả về ngay trên chính trang đó thì trường hợp này gọi là In-band SQLi. Với In-band có 2 kỹ thuật hay được sử dụng để khai thác là **Error Base** và **UNION Base**

### Error Base

Lợi dụng thông báo lỗi để nhận diện lỗi hoặc khai thác thông tin trong CSDL (Lấy thông tin về loại DB, Kiểu DB, Tên DB, Tên bảng và dữ liệu trong DB ...) và không làm thay đổi dữ liệu trong DB.

Ví dụ:

* ***VD dùng Error Base để nhận diện lỗi***: Đã quá quen thuộc với hình bên dưới đối vơi những ai đã từng học PHP-MySQL. Đây là thông báo đặc trưng để phát hiện Ứng dụng chứa lỗi SQLi và Kiểu DB là MySQL.

![sqli technical error base1]({{site.url}}/assets/img/2025/02/12/2-sql-error-base1.png)

* ***VD dùng Error Base để khai thác/truy dữ liệu DB:*** Ứng dụng nhận đối số **input** từ user không làm sạch dữ liệu. Phía dưới được xử lý bằng câu lệnh truy vấn như sau:

```SELECT * FROM article WHERE title='$input';```

Với cấu trúc bảng:

```sql
    CREATE TABLE article (
        id INT AUTO_INCREMENT PRIMARY KEY,
        title VARCHAR(255),
        content TEXT
    ); 
```

Nếu Input nhận vào như sau ```' AND EXTRACTVALUE(1, CONCAT(0x7e, (SELECT database()))) -- ```. Kết quả trả về nếu lỗi được hiển thị trên page ta sẽ lấy được thông tin bảng: ```XPATH syntax error: '~mydb'```

Giải thích từng phần:
* ```' AND ... --``` → Bắt đầu SQL Injection và kết thúc truy vấn hợp lệ.

* ```EXTRACTVALUE(1, ...)``` → Gọi hàm EXTRACTVALUE() để cố tình gây lỗi hiện thị. Hàm này yêu cầu format trong dấu ngoặc là XML nên ta cung cấp text thông thường nó sẽ báo lỗi.

* ```CONCAT(0x7e, (SELECT database()))``` → 0x7e là ký tự ~ trong bảng mã hex, giúp làm rõ dữ liệu trả về. (SELECT database()) lấy tên database hiện tại.

* ```--```` → Comment phần còn lại của truy vấn để tránh lỗi cú pháp.```

Practice: Thử suy nghĩ xem muốn lấy danh sách các bảng trong DB và truy vấn thông tin trong một bảng thì làm thế nào :)

>Note: Với kỹ thuật này có 2 hàm rất hay được sử dụng khi khai thác Error base SQLi với MySQL là ExtractValue() hoặc UpdateXML()  ⇒ Ghi nhớ điều này sẽ sử dụng sau này.

### UNION Base

# Blind SQLi

# Outband SQLi


