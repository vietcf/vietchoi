---
layout: post
title: "Câu chuyện viết security baseline của tôi,"
categories: job
tags: ['Security']
image: assets/img/2025/02/13/0-securitybaseline-intro.png
---

Gần đây tôi được giao nhiệm vụ quản lý và cập nhật lại đống Security Baseline của cơ quan. Công việc này nói chung với tôi không phải là mới vì trước đây ở đơn vị cũ tôi cũng làm nhiều rồi. Tuy là vậy, thực tế mỗi lần làm đều gặp những trắc trở khác nhau cả về Technical và None Techincal cần phải vượt qua, chả cái nào giống cái nào :). Không sao cứ làm thôi khó ở đâu thì mình gỡ ở đó. Hôm nay muốn viết lại ghi chú này, cho tôi và cho cả những người cần nó. 

Security Baseline ngoài việc improve tính bảo mật hệ thống mang lại lợi ích nhìn thấy ngay. Đối với tổ chức lớn nó còn mang tính tuân thủ. Việc tuân thủ đảm bảo bạn có đủ tư cách để join vào cộng đồng chung dù bạn muốn hay không => Bắt buộc phải xây dựng.

# Nguyên tắc tôi sẽ tuân thủ
Tôi sẽ bắt đầu bằng các nguyên tắc tôi sẽ tuân thủ trong suốt thời gian tôi thực hiện:

+ Defense in the depth: Nguyên tắc này tương đối đơn giản và hẳn ai làm Security đều đã nghe. Việc bảo vệ cần được thực hiện ở nhiều layer khác nhau, lọt sàng xuống nia.

![image.png]({{site.url}}/assets/img/2025/02/13/1-security-baseline-dod.png)

+ Simple is the best: Simple thường với nghĩa là **đơn giản** ~ "làm ít" nhất có thể nhưng vẫn cần đảm bảo hiệu quả theo hướng chắt lọc những cái thực sự quan trọng mới làm (khoai nha).

Làm ít: Sẽ giảm số lượng công việc khi apply vào hệ thống thật hay maintain chính Baseline. Số lượng các đối tượng Apply security baseline trong tổ chức thường là rất lớn nếu làm nhiều thì sức đâu mà làm nỏi. Làm ít để giảm thiểu tác động xấu lên hệ thống hiện tại khi apply: Gây chết gián đoạn dịch vụ, ... nói chung có vô vàn lợi ích mà ngồi đây kể chả hết được. Nhưng nhắc lại Làm ít không có nghĩa là làm qua loa cho có, đã bỏ thời gian ra làm thì phải mang lại giá trị cho chính những gì mình bỏ ra và cho tổ chức.


# Hướng đi 

Bình thường: Tham khảo từ CIS, NIST, ISO, PCI/DSS
Tuy nhiên việc này không phải cứ vác về là dùng mà nó phải custom theo khẩu vị của từng tổ chức để đảm bảo apply vào mà hệ thống không bị chết.
Của tôi: 
+ Nghiên cứu về technical theo Mitre
+ Chắt lọc từ các nguồn tham khảo
=> Đưa ra phiên bản của mình.

# Chi tiết các phần của một Baseline theo quan điểm của tôi

# Vấn đề gặp phải và cách giải quyết

