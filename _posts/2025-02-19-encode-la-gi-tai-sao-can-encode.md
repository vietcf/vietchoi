---
layout: post
title: "Encoding là gì? Tại sao cần Encoding dữ liệu?"
categories: job
tags: ['Security']
image: assets/img/2025/02/19/0-encoding-intro.png
---

Trong quá trình làm việc ta thường hay gặp cụm từ Encoding(n)/Encode(v). Ví dụ:  Base64 Encode, URL Encode, … vậy Encoding là gì và tại sao cần Encode?

# Encode là gì?

Search một hồi trên mạng tôi thấy định nghĩa *“Encoding is the process of converting data into a specific format that can be easily transmitted or stored”* tạm dịch sang tiếng việt “Encoding là quá trình chuyển đổi dữ liệu sang một định dạng cụ thể để có thể dễ dàng truyền tải hoặc lưu trữ.”

Đọc nghe khá rối rắm đúng không? 

# Tại sao lại cần Encode?

> Mọi cái sinh ra đều có lý do của nó phải không?
> 

Tại sao cần Encode? Để trả lời cho câu hỏi này không có cách nào hay hơn là đưa ra các vấn đề mà encode có thể giải quyết trong thực tế! Một số tình huống cụ thể có thể kể đến như sau:

- Đầu tiên là base64 encoding:  Đây là một trong số những kiểu encode dùng để chuyển dữ liệu từ dạng mã nhị phân sang dạng text. Tưởng tượng như ta đang có nhu cầu gửi email có một bức ảnh chẳng hạn, ảnh thì bản chất nó ở dạng binary. Vì giao thức SMTP như chúng ta biết nó là text base nên thông thường email không thể truyền data dạng binary đi được rồi. Vậy nên ta cần phải chuyển từ dữ liệu binary sang dạng text để có thể gửi ảnh qua SMTP ⇒  Ta cần phải dùng encode base64 để chuyển ảnh binary nó sang dạng text. Đây là ví dụ thực tế cho việc dùng Encode phục vụ việc truyền tải dữ liệu giữa 2 bên. Điều này cũng xảy ra tương tự với trường hợp cần lưu (store) ảnh vào cơ sở dữ liệu và phương án thực hiện cũng tương tự là Encoding.
- Một kịch bản khác thường hay gặp hơn, đó là url encoding: Như ta đã biết, url chỉ có thể chứa những kí tự trong bảng mã [ASCII](http://www.w3schools.com/charsets/ref_html_ascii.asp). Tuy nhiên, trong thực tế, các đường dẫn url của ta thường xuyên cần phải chứa những kí tự đặc biệt nằm ngoài bảng mã này (Như khi biến GET truyền lên có chứa ký tự dấu của tiếng việt chả hạn), thế nên, ta cần sử dụng encoding để chuyển nó về định dạng chấp nhận được ở đây là chỉ bao gồm các ký tự trong bảng mã ASCII ⇒ Để giải quyết vấn đê này ta cũng cần một phương án Mapping dữ liệu đặc biệt về các ký tự trong ASCII. Cụ thể, các thuật toán url encode, thường sẽ thay thế các kí tự đặc biệt bằng một chuỗi bắt đầu với `%` gắn với 2 giá trị số hex.

⇒ Tóm lại Encoding là quan trọng trong lĩnh vực khoa học máy tính.

# Quá trình ngược của Encode là Decode

Cái này dĩ nhiên rồi, có Encode để chuyển đổi dữ liệu sang định dạng đích thì phải có Decode để khôi phục dữ liệu về dạng ban đầu. 

![encode and decode]({{site.url}}/assets/img/2025/02/19/1-encoding-and-decoding-strings-problem.png)

Thông thường các thuật toán Encode và Decode  theo một số chuẩn nhất định và các chuẩn này cũng được công khai. Ai có đoạn mã đã được encode thì có thể decode đoạn mã đó một cách dễ dàng mà không cần thêm thông tin “bí mật gì” miễn là chọn đúng thuật toán mà thôi.

# Tổng hợp lại

- Encode là để đảm bảo **khả năng sử dụng (usability) của dữ liệu**. Khả năng sử dụng này bao gồm việc phục vụ việc truyền tải (transmit) hoặc lưu trữ (store) dữ liệu.
- Encode nhiều khi người ta cũng gọi là “Mã hóa”, tuy nhiên **encoding không liên quan và cũng không giúp cải thiện việc bảo mật của dữ liệu**