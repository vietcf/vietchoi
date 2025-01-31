---
layout: post
title: "Pentest 101 - Giao thức HTTP những điểm cần lưu ý,"
categories: job
tags: ['Security']
image: assets/img/2025/01/31/image4.png
---

# 1.Cơ bản về giao thức HTTP

HTTP là giao thức quá đỗi quen thuộc mà hầu hết các bạn hoặc sẽ được học ở bậc đại học hoặc sẽ học trong “trường đời” khi đi làm. Vì vậy chắc ở đây ta không phải nói quá nhiều và quá chi tiết về nó (Nói dài nói dai thành nói dại). Bài viết này chỉ xin ghi chú lại các điểm chính yếu nhất mà thôi.

### HTTP hoạt động mô hình Client → Server

- HTTP hoạt động theo mô hình Client → Server: Với mô hình này Client hỏi (ta hay gọi là **Request**) và Server lắng nghe nhận câu hỏi và trả lời lại (ta hay gọi là **Response**). Nhấn mạnh lại rằng Server chỉ duy nhất giữ vai trò lắng nghe nhận câu hỏi và trả lời thôi, không thể chủ động tự mình liên lạc với Client để gửi các thông điệp (message) được. Trong HTTP cả khi Server có nhu cầu đó đi chăng nữa thì việc gửi gì đó tới Client nó vẫn phải hoàn toàn nằm đó chờ Client gửi request tới và phản hồi ý kiến của mình trong Response ngay sau đó.

![image.png]({{site.url}}/assets/img/2025/01/31/image.png)

> Trong thực tế để Server có thể chủ động liên lạc ngược lại Client, thực hiện giao tiếp 2 chiều ta phải sử dụng một giao thức hoàn toàn khác gọi là Web Socket.

### HTTP là giao thức dạng text base

- Nghĩa là các message truyền nhận giữa Client và Server đều ở dạng text và ta có thể dễ dàng đọc được.

![image1.png]({{site.url}}/assets/img/2025/01/31/image1.png)

### HTTP là Stateless

Stateless trong tiếng việt có thể hiểu là “Không nhớ trạng thái”. Điều này nghĩa là các cặp Request → Response của HTTP hoàn toàn là độc lập. Hỏi và trả lời là xong. Các cặp Request → Response hầu như không có liên hệ gì với nhau.

Nói tới đây nhiều bạn thắc mắc làm sao để Web server nhận biết được các Request đến từ cùng người dùng? vì rõ ràng trong thực tế, các ứng dụng web ta vẫn thấy các request của cùng một người dùng “có vẻ” như server nhận biết được. Bằng chứng là thực tế khi ta vào một ứng dụng web có tính năng đăng nhập, ta chỉ cần đăng nhập một lần. Các lần request sau ta hoàn toàn không phải đăng nhập lại nữa. Câu trả lời là trên nền tảng của HTTP nguyên bản người ta đã sử dụng thêm một số kỹ thuật khác để phân biệt phiên sử dụng của người dùng. Một số kỹ thuật phổ biến thường được sử dụng bao gồm:

**Sử dụng Cookie kết hợp với Session trên Server**

- **Cookie** là các dữ liệu nhỏ mà server gửi đến trình duyệt và được lưu trữ trên máy của người dùng. Khi người dùng gửi request tiếp theo, trình duyệt sẽ tự động gửi kèm cookie này.
- Sử dụng Session trên Server kết hợp với Cookie để nhận diện phiên người dùng

*Cách hoạt động:*

1. Khi người dùng truy cập lần đầu, server tạo ra Session ID mà một mã duy nhất cho người dùng để nhận diện người dùng. Session ID này được lưu vào Cookie (ví dụ: `PHPSESSID=abc123`) và gửi nó trong response cho người dùng
2. Trình duyệt lưu cookie chứa Session ID này và gửi nó kèm với các request tiếp theo

![image2.png]({{site.url}}/assets/img/2025/01/31/image2.png)

3. Server dùng session ID để nhận diện người dùng đồng thời tra cứu dữ liệu người dùng đã lưu trong bộ nhớ (RAM, database).

**Sử dụng Token (JWT - JSON Web Token)**

- Server cấp một **token** chứa thông tin định danh (được mã hóa và ký số) cho người dùng sau khi họ xác thực thành công.
- Token này được lưu trong trình duyệt (thường trong `localStorage` hoặc `sessionStorage`) và gửi kèm theo mỗi request, thường trong **Authorization** header.

*Cách hoạt động:*

1. Người dùng đăng nhập → server cấp token.
2. Các request tiếp theo của người dùng sẽ gửi token:

    ![image3.png]({{site.url}}/assets/img/2025/01/31/image3.png)

3. Server dựa vào thông tin định danh trong token để nhận diện người dùng

**Sử dụng IP và User-Agent**

- Server có thể dựa vào **địa chỉ IP** và thông tin **User-Agent** (trình duyệt, hệ điều hành) để tạm nhận diện người dùng.
- Tuy nhiên, cách này **không chính xác hoàn toàn**:
  - Địa chỉ IP có thể bị thay đổi (do NAT hoặc ISP).
  - Nhiều người dùng có thể dùng chung IP.

**Kết hợp các phương pháp**

Trong thực tế, server thường kết hợp nhiều phương pháp trên để đạt độ chính xác cao:

1. Sử dụng **Cookies** hoặc **Token** để nhận diện người dùng.
2. Kết hợp với IP hoặc thông tin thiết bị để tăng tính bảo mật.

# Các công nghệ web PHP, Java, Dotnet … bản chất/nguyên tắc hoạt động bên dưới là gì?

Ta hay nghe có một số loại Web server như apache, nginx, lighhttpd, IIS, … và các ngôn ngữ lập trình Web như PHP, Java, Python, …. Vậy bản chất chúng là gì và cơ chế hoạt động như thế nào?

![image4.png]({{site.url}}/assets/img/2025/01/31/image4.png)

Trong giao thức HTTP ta sẽ thấy các message Request và Response chỉ ở dạng HTML text file không có gì hơn, bất kể phía server site ta sử dụng php, java, dotnet đi chăng nữa. Nội dung HTML text file bao gồm 3 cấu trúc cú pháp chính định nghĩa nên: html tag, css, js. Trình đọc HTML text file này để build lên giao diện người dùng. Để kiểm chứng ta chuột phải chọn View Source sẽ thấy.

![image5.png]({{site.url}}/assets/img/2025/01/31/image5.png)

- Vậy các ngôn ngữ như PHP, Java, Dotnet kia để làm gì?

Thời gian đầu các trang web là các trang HTML cố định không thay đổi, người ta cũng gọi là công nghệ web tĩnh. Sau này để có khả năng hiển thị theo ngữ cảnh, cá nhân hóa từng người dùng người ta phát triển ra công nghệ web động. Với công nghệ web động các đoạn mã HTML thường không có sẵn trên web server mà nội dung HTML được sinh ra động theo ngữ cảnh. Ví dụ như khi ta login với User A website sẽ hiển thị một kiểu, khi login với User B sẽ hiển thị một kiểu không giống nhau. Các ngôn ngữ như PHP, Java, Dotnet, … sinh ra để phục vụ công nghệ web động.

Để đơn giản việc mô tả công nghệ web động tôi sẽ lấy ví dụ của nginx - Một web server khá phổ biến hoạt động kết hợp với PHP. Dĩ nhiên với các web server còn lại nó cũng hỗ trợ các cơ chế tương tự.

![image6.png]({{site.url}}/assets/img/2025/01/31/image6.png)

Với Nginx khi người dùng request các tài nguyên với phần mở rộng là `.html, .js, .css` thì Nginx sẽ tìm kiếm các file tĩnh tương ứng ngay thư mục Document root của mình trả lại cho máy tính người dùng luôn do khi trả lại dạng text này browser sẽ hiểu. Còn khi request một file với phần mở rộng là `.php` do browser không thể hiểu được luôn nên Nginx sẽ forward nội dung file .php tới cho PHP Handler để xử lý file theo cú pháp của php. PHP Handler đọc mã `.php` này để xử lý rồi trả lại cho Nginx theo một cú pháp nhất định để Nginx có thể hiểu. Sau khi nhận được dữ liệu này Nginx chuyển toàn bộ nội dung qua HTML rồi trả lại cho người dùng.

Việc giao tiếp giữa Nginx và PHP Handler là giao tiếp liên tiến trình (process) thông qua file socket hoặc TCP Socket (tùy do người cấu hình lựa chọn) thông qua giao thức FastCGI. Dễ tìm thấy trong nginx đoạn cấu hình sau của Nginx.

![image7.png]({{site.url}}/assets/img/2025/01/31/image7.png)

Bức tranh tổng thể sẽ như sau:

![image8.png]({{site.url}}/assets/img/2025/01/31/image8.png)

Bài viết cũng khá dài rồi, tạm thời viết tới đây, có gì hay ho tôi sẽ cập nhật tiếp sau.
