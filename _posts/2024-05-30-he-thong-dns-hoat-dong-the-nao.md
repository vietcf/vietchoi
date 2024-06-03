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

-  **DNS recursor (Hoặc Recursive resolvers)** - Recurive tiếng việt là "Đệ quy". Đúng như cái tên nó **DNS recursor** là điểm dừng đầu tiên trong một truy vấn DNS. **DNS recursor** hoạt động như một trung gian giữa máy khách (Client) và các loại máy chủ DNS khác. Sau khi nhận được truy vấn DNS từ một Client, **DNS recursor** sẽ tìm kiếm các thông tin đã lưu trữ trong bộ nhớ Cache của nó nếu có thì phản hồi trả lại thông tin IP cho Client luôn và kết thúc việc tìm kiếm. Trường hợp không tìm thấy trong Cache của mình, **DNS recursor** sẽ gửi yêu cầu tới **Root nameserver**, lúc này **Root nameserver** sẽ trả lại cho **DNS recursor** thông tin "chỉ dẫn" giúp nó có thể xác định được một **TLD (Top-Level Domain)** tương ứng với domain truy vấn. Ngay sau khi có thông tin về TLD thì **DNS recursor** sẽ thực hiện việc tiếp theo là gửi một yêu tới máy chủ TLD. Thông tin từ TLD trả lại sẽ là "chỉ dẫn" giúp **DNS recursor** có thể xác định địa chỉ máy chủ **Authoritative nameserver**. Ngay sau khi nhận được địa chỉ của **Authoritative nameserver** thì **DNS recursor** sẽ gửi một truy vấn tới **Authoritative nameserver**. Tới đây máy chủ **Authoritative nameserver** sẽ phản hồi thông tin cho **DNS recursor** địa chỉ IP được yêu cầu. Cuối cùng **DNS recursor** sẽ gửi phản hồi tới Client.

![DNS recursor]({{site.url}}/assets/img/2024/05/30/recursive-resolver.png)

Trong quá trình này, hãy lưu ý rằng **DNS recursor** sẽ lưu vào bộ nhớ cache thông tin nhận được từ máy chủ **Authoritative nameserver**. Khi một máy khách yêu cầu địa chỉ IP của domain đã được yêu cầu gần đây bởi một Client khác, **DNS recursor** có thể bỏ qua quá trình giao tiếp với các máy chủ DNS khác và phản hồi luôn thông tin có trong bộ nhớ cache của nó.

Thực tế hầu hết người dùng Internet sử dụng DNS recursor do nhà cung cấp dịch vụ Internet (ISP) cung cấp, dĩ nhiên ta cũng có những lựa chọn khác: Ví dụ như 1.1.1.1 của Cloudflare, hoặc 8.8.8.8 của GG.

- **Root nameserver** - Trên thế giới có 13 máy chủ **Root nameserver**. Thực tế là ngay khi cài đặt địa chỉ của 13 Root nameserver này được fix luôn trong cấu hình của các DNS server. Một máy chủ **Root nameserver** chấp nhận truy vấn của bộ giải đệ quy, bao gồm một tên miền, và máy chủ gốc sẽ phản hồi bằng cách hướng dẫn bộ giải đệ quy đến một máy chủ tên miền cấp cao (TLD) dựa trên phần mở rộng của tên miền đó (.com, .net, .org, v.v.). Các máy chủ tên miền gốc được giám sát bởi một tổ chức phi lợi nhuận gọi là Tập đoàn Internet về Tên miền và Số hiệu được cấp phát (ICANN).

Lưu ý rằng mặc dù có 13 máy chủ tên miền gốc, điều đó không có nghĩa là chỉ có 13 máy chủ trong hệ thống máy chủ tên miền gốc. Có 13 loại máy chủ tên miền gốc, nhưng có nhiều bản sao của mỗi loại trên khắp thế giới, sử dụng định tuyến Anycast để cung cấp phản hồi nhanh chóng. Nếu bạn cộng tất cả các phiên bản của các máy chủ tên miền gốc, bạn sẽ có hơn 600 máy chủ khác nhau.


- **TLD nameserver** - Top Level Domain Nameserver (TLD) duy trì thông tin cho tất cả các tên miền có chung phần mở rộng tên miền, chẳng hạn như .com, .net, hoặc bất kỳ phần mở rộng nào sau dấu chấm cuối cùng trong một URL. Ví dụ, một máy chủ TLD nameserver .com chứa thông tin cho mọi trang web kết thúc bằng ‘.com’. Nếu người dùng tìm kiếm google.com, sau khi nhận được phản hồi từ một máy Root nameserver, bộ giải đệ quy sẽ gửi một truy vấn đến máy chủ tên miền cấp cao nhất .com, và máy chủ này sẽ phản hồi bằng cách trỏ tới máy chủ Authoritative nameserver (xem bên dưới) cho tên miền đó.

Việc quản lý các máy chủ TLD nameserver được xử lý bởi Internet Assigned Numbers Authority (IANA), một nhánh của ICANN. IANA phân chia các máy chủ TLD thành hai nhóm chính:

    Generic top-level domains (Tên miền chung không thuộc quốc gia cụ thể): Đây là các tên miền không thuộc về quốc gia cụ thể, một số TLD chung được biết đến nhiều nhất bao gồm .com, .org, .net, .edu, và .gov.

    Country code top-level domains (Tên miền theo mã quốc gia): Bao gồm bất kỳ tên miền nào thuộc về một quốc gia hoặc tiểu bang cụ thể. Ví dụ bao gồm .uk, .us, .ru, và .jp.

Thực tế còn có một loại thứ ba dành cho các tên miền hạ tầng, nhưng hầu như không bao giờ được sử dụng. Loại này được tạo ra cho tên miền .arpa, vốn là một tên miền chuyển tiếp được sử dụng trong việc tạo ra DNS hiện đại; ý nghĩa của nó ngày nay chủ yếu mang tính lịch sử.

- Authoritative nameserver - Authoritative nameserver là máy chủ chứa thông tin ánh xạ domain - địa chỉ.  Authoritative nameserver là điểm dừng cuối cùng trong truy vấn domain.

> Bạn có tự hỏi tại sao phải chia ra nhiều loại DNS Server thế này không :) tôi cũng không hiểu đâu.

# Một truy vấn DNS thực tế trải qua các bước như thế nào

Đối với hầu hết các tình huống, DNS query liên quan đến việc phân giải một tên miền thành địa chỉ IP tương ứng. Để hiểu cách thức hoạt động của quá trình này, chúng ta hãy theo dõi con đường của một truy vấn DNS từ trình duyệt web, qua quá trình query các DNS servers và trở lại. Hãy cùng xem qua các bước sau.

Lưu ý: Thông tin tra cứu DNS thường được lưu trong bộ nhớ đệm, hoặc cục bộ bên trong máy tính truy vấn hoặc từ xa trong cơ sở hạ tầng DNS. Thông thường có 8 bước trong một truy vấn DNS. Khi thông tin DNS được lưu trong bộ nhớ đệm, các bước sẽ được bỏ qua, giúp quá trình nhanh hơn. Ví dụ dưới đây trình bày toàn bộ 8 bước khi không có gì được lưu trong bộ nhớ đệm.

![DNS recursor]({{site.url}}/assets/img/2024/05/30/full_dns_query.png)


1. Trình duyệt gửi yêu cầu DNS: Khi bạn nhập URL vào trình duyệt, trình duyệt sẽ kiểm tra bộ nhớ đệm của chính nó để xem liệu nó có bản ghi DNS cho tên miền đó hay không.

2. Hệ điều hành kiểm tra bộ nhớ đệm: Nếu trình duyệt không có bản ghi, yêu cầu sẽ được gửi đến hệ điều hành, nơi cũng kiểm tra bộ nhớ đệm của nó.

3. Truy vấn đến bộ giải đệ quy DNS: Nếu hệ điều hành không tìm thấy bản ghi, nó sẽ gửi truy vấn đến bộ giải đệ quy DNS (thường là máy chủ DNS của ISP).

4. Truy vấn máy chủ gốc DNS: Bộ giải đệ quy gửi truy vấn đến một trong những máy chủ gốc DNS để tìm máy chủ TLD phù hợp.

5. Truy vấn máy chủ TLD DNS: Máy chủ gốc trả lời với địa chỉ của máy chủ TLD, ví dụ .com, và bộ giải đệ quy sẽ gửi truy vấn đến máy chủ TLD này.

6. Truy vấn máy chủ DNS có thẩm quyền: Máy chủ TLD trả lời với địa chỉ của máy chủ có thẩm quyền cho tên miền cụ thể, ví dụ google.com.

6.  Nhận phản hồi từ máy chủ có thẩm quyền: Bộ giải đệ quy sau đó sẽ gửi truy vấn đến máy chủ có thẩm quyền và nhận được địa chỉ IP của tên miền.

7.  Trả kết quả về cho trình duyệt: Bộ giải đệ quy trả lại địa chỉ IP cho hệ điều hành, hệ điều hành trả lại cho trình duyệt, và cuối cùng trình duyệt sử dụng địa chỉ IP này để truy cập vào trang web.

Khi các bước này được thực hiện, các thông tin DNS sẽ được lưu trữ trong bộ nhớ đệm tại các mức khác nhau (trình duyệt, hệ điều hành, bộ giải đệ quy DNS) để tối ưu hóa các lần truy cập sau.



