---
layout: post
title: "Hệ thống DNS hoạt động như thế nào"
author: "vietcf"
categories: job
tags: ['Networking']
image: assets/img/2024/05/30/what_is_dns.png
---

Hệ thống phân giải tên miền DNS là một hệ thống quan trọng trong môi trường mạng hiện tại bất kể là internet hay là LAN. Bài viết hôm nay tập trung phân tích cách thức các truy vấn DNS thực tế hoạt động như thế nào để thực hiện việc phân giải một Domain -> IP. Đọc tới đây có thể có bạn thắc mắc nếu là bản ghi txt nó là text thì sao? đâu nhất thiết phải là IP????? Thực tế kể cả trả lại text đi chăng nữa thì quá trình hoạt động vẫn tương tự. Chỉ khác ở bước cuối cùng thông tin trả từ Authoritative Nameserver cho Client khác nhau mà thôi. Để đơn giản trong bài viết này tôi sẽ chỉ sử dụng trường hợp phân giải Domain -> IP cho dễ hình dung.

# Hiểu về các loại DNS server
Ta vẫn hiểu "sơ sơ" máy chủ DNS giúp ta phân giải một tên miền thành IP. Nghe thì có vẻ easy nhưng trong môi trường mạng thực tế việc này phức tạp hơn nhiều. Có đến 4 loại DNS server với các chức năng khác nhau phối hợp với nhau để thực hiện việc việc phân giải này. 

![DNS recursor]({{site.url}}/assets/img/2024/05/30/recursive-resolver.png)

-  **DNS recursor (Hoặc Recursive resolvers)**: Recurive tiếng việt là "Đệ quy". Đúng như cái tên nó **DNS recursor** là điểm dừng đầu tiên trong một truy vấn DNS. **DNS recursor** hoạt động như một trung gian giữa máy khách (Client) và các loại máy chủ DNS khác: Bao gồm gửi yêu cầu tới **Root nameserver** để lấy thông tin địa chỉ của **TLD (Top-Level Domain)** tương ứng với domain truy vấn. Hay **DNS recursor** thực hiện gửi yêu tới máy chủ **TLD (Top-Level Domain)** để lấy thông tin địa chỉ máy chủ **Authoritative nameserver**. **DNS recursor**  cũng thực hiện gửi yêu cầu tới **Authoritative nameserver** để lấy thông tin địa chỉ IP được yêu cầu. Cuối cùng **DNS recursor** tổng hợp các thông tin hoàn thiện gửi phản hồi tới Client.

*Thực tế hầu hết người dùng Internet sử dụng **DNS recursor** do nhà cung cấp dịch vụ Internet (ISP) cung cấp, dĩ nhiên ta cũng có những lựa chọn khác: Ví dụ như 1.1.1.1 của Cloudflare, hoặc 8.8.8.8 của GG.*

- **Root nameserver**: Trên thế giới có 13 máy chủ **Root nameserver**. Thực tế là ngay khi cài đặt địa chỉ của 13 Root nameserver này được fix luôn trong cấu hình của các DNS server. Một máy chủ **Root nameserver** sẽ nhận các truy vấn của các **DNS recursor** và phản hồi lại cho **DNS recursor** địa chỉ của **TLD (Top-Level Domain)** tương ứng dựa trên phần mở rộng của tên miền đó (.com, .net, .org, v.v.). Các máy chủ **Root nameserver** được giám sát bởi một tổ chức phi lợi nhuận gọi là Internet Corporation for Assigned Names and Numbers (ICANN).

*Lưu ý rằng mặc dù trên lý thuyết có 13 Root nameserver nhưng thực tế không có nghĩa là chỉ có 13 Root nameserver duy nhất trên hệ thống DNS. Ngoài 13 Root nameserver  có nhiều bản sao của mỗi Root nameserver trên khắp thế giới, chúng sử dụng định tuyến Anycast để cung cấp phản hồi nhanh chóng. Nếu cộng tất cả các phiên bản của các Root nameserver, bạn sẽ có hơn 600 máy chủ khác nhau.*


- **Level Domain nameserver (TLD)**: Top Level Domain Nameserver duy trì thông tin cho tất cả các tên miền có chung phần mở rộng tên miền, chẳng hạn như .com, .net, hoặc bất kỳ phần mở rộng nào sau dấu chấm cuối cùng trong một URL. Ví dụ, một máy chủ TLD nameserver .com chứa thông tin cho mọi trang web kết thúc bằng ‘.com’. Nếu người dùng tìm kiếm google.com, sau khi nhận được phản hồi từ một máy Root nameserver, DNS recursor sẽ gửi một truy vấn đến **Level Domain nameserver** .com. Máy chủ này sẽ phản hồi bằng cách trỏ tới máy chủ **Authoritative nameserver** cho tên miền đó.

Các máy chủ TLD nameserver được quản lý bởi Internet Assigned Numbers Authority (IANA), một nhánh của ICANN. IANA phân chia các máy chủ TLD thành hai nhóm chính:

    + Generic top-level domains (Tên miền chung không thuộc quốc gia cụ thể): Đây là các tên miền không thuộc về quốc gia cụ thể, một số TLD chung được biết đến nhiều nhất bao gồm .com, .org, .net, .edu, và .gov.

    + Country code top-level domains (Tên miền theo mã quốc gia): Bao gồm bất kỳ tên miền nào thuộc về một quốc gia hoặc tiểu bang cụ thể. Ví dụ bao gồm .uk, .us, .ru, và .jp.

- **Authoritative nameserver**: Authoritative nameserver là máy chủ chứa thông tin ánh xạ domain - địa chỉ IP.  **Authoritative nameserver** là điểm dừng cuối cùng trong truy vấn domain -> IP.

# Một truy vấn DNS thực tế trải qua các bước như thế nào

Đối với hầu hết các tình huống, DNS query liên quan đến việc phân giải một tên miền thành địa chỉ IP tương ứng. Để hiểu cách thức hoạt động của quá trình này chúng ta hãy theo dõi con đường của một truy vấn DNS từ trình duyệt web, trải qua quá trình query các DNS servers và trở lại Client. Hãy cùng xem qua các bước sau.

Lưu ý: Thông tin tra cứu DNS thường được lưu trong bộ nhớ đệm (Cache). Cache này có thể là cục bộ bên trong máy tính truy vấn (Client) hoặc trong các máy chủ DNS khác nhau trong cơ sở hạ tầng DNS. Thông thường có 8 bước trong một truy vấn DNS. Khi thông tin DNS được lưu trong bộ nhớ đệm (Cache), ở bước nào đó thì kết quả sẽ được trả lại ngay và các bước sau đó sẽ được bỏ qua. Việc này giúp quá trình nhanh hơn. Ví dụ dưới đây trình bày toàn bộ 8 bước khi không có gì được lưu trong Cache.

![DNS Flow]({{site.url}}/assets/img/2024/05/30/full_dns_query.png)


1. Trình duyệt gửi yêu cầu DNS: Khi bạn nhập URL vào trình duyệt, trình duyệt sẽ kiểm tra bộ nhớ đệm của chính nó để xem liệu nó có bản ghi DNS cho tên miền cần truy vấn hay không.

2. Hệ điều hành kiểm tra bộ nhớ đệm: Nếu trình duyệt không có bản ghi trong bộ nhớ đệm, yêu cầu sẽ được gửi đến hệ điều hành, nơi cũng kiểm tra bộ nhớ đệm của nó.

3. Truy vấn đến **DNS recursor**: Nếu hệ điều hành không tìm thấy bản ghi trong bộ nhớ đệm, nó sẽ gửi truy vấn đến **DNS recursor** (thường là máy chủ DNS của ISP).

4. Truy vấn **Root nameserver**: **DNS recursor** gửi truy vấn đến một trong những máy chủ Root nameserver để tìm máy chủ Level Domain nameserver phù hợp.

5. Truy vấn máy chủ **Level Domain nameserver (TLD)**: Root nameserver trả lời với địa chỉ của máy chủ TLD. Ví dụ .com, và **DNS recursor** sẽ gửi truy vấn đến máy chủ TLD này.

6. Truy vấn máy chủ **Authoritative nameserver**: Máy chủ TLD trả lời với địa chỉ của máy chủ Authoritative nameserver cụ thể. Ví dụ google.com.

6. Nhận phản hồi từ máy chủ **Authoritative nameserver**: **DNS recursor** sau đó sẽ gửi truy vấn đến máy chủ Authoritative nameserve và nhận được địa chỉ IP của tên miền.

7. Trả kết quả về cho trình duyệt: **DNS recursor** trả lại địa chỉ IP cho hệ điều hành, hệ điều hành trả lại cho trình duyệt và cuối cùng trình duyệt sử dụng địa chỉ IP này để truy cập vào trang web.

Khi các bước này được thực hiện, các thông tin DNS sẽ được lưu trữ trong bộ nhớ đệm tại các mức khác nhau (trình duyệt, hệ điều hành, DNS recursor) để tối ưu hóa các lần truy cập sau.



