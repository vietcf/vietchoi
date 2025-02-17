---
layout: post
title: "Tổng hợp tất cả các kỹ thuật scan trong Nmap,"
categories: job
tags: ['Security']
image: assets/img/2023/02/26/nmap_intro.jpg
---

Nmap (Network Mapper) là một công cụ quen thuộc với những ai làm trong lĩnh vực Network, System hay Security. Mặc dù miễn phí, Nmap cung cấp một loạt các tính năng mạnh mẽ, không hề thua kém bất kỳ sản phẩm thương mại nào. Tuy nhiên, chính sự đa dạng về tính năng và tùy chọn (option) đôi khi lại gây khó khăn cho người sử dụng, đặc biệt là những người mới bắt đầu. Vậy làm thế nào để có thể "thuần hóa" được công cụ mạnh mẽ này? Bài viết này sẽ chia sẻ những cách tiếp cận giúp bạn làm việc với Nmap hiệu quả hơn.

# Các tính năng của nmap

Trước khi đi sâu vào chi tiết, tôi muốn giới thiệu tới các bạn một hình ảnh minh họa về các tính năng chính của Nmap.

![nmap feature]({{site.url}}/assets/img/2023/02/26/nmap_feature.PNG)

Dựa vào hình ảnh, chúng ta có thể thấy rõ mindset design của Nmap là hỗ trợ cho từng bước của quá trình khám phá mạng (Discovery) thông tin mạng. Ban đầu, Nmap nhận đầu vào địa chỉ mạng (ví dụ: 192.168.1.0/24) hoặc địa chỉ của một host (IP hoặc domain). Sau đó, nó sẽ giúp ta xác định xem host nào trong mạng (nếu đầu vào là địa chỉ mạng) hoặc host được chỉ định (nếu đầu vào là địa chỉ host) còn hoạt động hay không. Nếu host còn sống (alive/online), Nmap sẽ tiếp tục xác định hệ điều hành (OS), các cổng (Port) đang mở, dịch vụ (Service) tương ứng với từng cổng và phiên bản OS, phần mềm (Software) đang chạy dựa trên các dấu hiệu riêng (Fingerprint) của chúng. Điểm đặc biệt nữa là Nmap còn cho phép người dùng tùy chỉnh và mở rộng chức năng thông qua các script, ví dụ như tự động thực hiện một tác vụ nào đó khi phát hiện một cổng đang mở. Cuối cùng, kết quả quét và thực thi script có thể được lưu trữ dưới nhiều định dạng file khác nhau.

Với mỗi nhiệm vụ thu thập thông tin (ta hay gọi là scan) Nmap cung cấp các chiến lược (tatics)/kỹ thuật (techniques) khác nhau. Lấy ví dụ để scan host còn hoạt động hay không, Nmap có thể sử dụng nhiều phương pháp khác nhau như: ARP scan, ICMP scan hoặc TCP/UDP scan. Tùy thuộc chiến lược được chọn chúng ta sẽ sử dụng các tùy chọn (option) khác nhau khi chạy lệnh ```nmap```. Việc lựa chọn chiến lược nào (hoặc cũng có thể kết hợp nhiều chiến lược nào với nhau) tùy thuộc vào từng tình huống cụ thể của người sử dụng. Chẳng hạn, ARP scan thường được sử dụng trong mạng LAN, trong khi một số chiến lược SYN scan có thể giúp vượt qua tường lửa (do một số loại tường lửa có cơ chế hạn chế khả năng bị Scan).

Sau khi đã nắm vững những tính năng cơ bản của Nmap, việc tiếp theo chúng ta cần làm là tìm hiểu cách sử dụng các tùy chọn (option) trong lệnh Nmap một cách hiệu quả. Việc này sẽ giúp chúng ta tận dụng tối đa sức mạnh của công cụ này. Cách tiếp cận thông thường của tôi để xác định sử dụng option nào là sẽ tham khảo từ cheat sheet [Link Cheat sheet NMAP]({{site.url}}/assets/img/2023/02/26/nmap_cheet_sheet_v7.pdf) (Đây là cheetsheet mà tôi thấy có vẻ ok nhất tìm được trên mạng). Trong trường hợp cheetsheet không có thì tìm google hoặc hỏi ChatGPT thôi (Lựa chọn mới). Anw, thời buổi hiện nay làm gì cũng được miễn là sử dụng từ khóa tìm kiếm phù hợp mà thôi.

>Quy ước: Từ đây trở về sau bất cứ khi nào tôi sử dụng sudo nghĩa là cần quyền root để có thể thực hiện scan thành công. Lý do là với mốt số chiến lược nmap cần sử dụng đặc quyền cao để thực hiện một số tác vụ mà ở user thường không được phép truy cập trong hệ điều hành (OS).

# Một số option chung

Nmap có một số tùy chọn (option) chung mà ta có thể sử dụng ở tất cả các loại scan. Cụ thể các tùy chọn chung có thể xem ở phần đầu của cheat sheet bên trên

![common_options]({{site.url}}/assets/img/2023/02/26/common_options.PNG)

Áp dụng: Lấy ví dụ thay bằng việc phải gõ vào từng subnet/host trong console khi chạy command ta có thể đặt chúng trong 1 file tên là **targets.txt** rồi sử option `-iL` để load file này vào nmap scan, với mỗi host/subnet là một dòng (Dĩ nhiên scan cái gì ta sẽ kết hợp thêm các options khác nữa, các option khác sẽ nói sau). Để minh họa tôi thử scan host alive bằng ARP(-PR) không scan port (-sn) trong mạng LAN với lệnh sau:

`sudo nmap -PR -sn -iL targets.txt`

# Nmap live host Discovery ~ Xác định host còn sống (alive/online) trong mạng

Khi cần xác định các host trong mạng hoặc một host còn sống (alive/online) ta có thể sử dụng một số chiến lược/kỹ thuật như sau.

![host alive overview]({{site.url}}/assets/img/2023/02/26/host_alive_overview.PNG)

Chú ý: Do chỉ cần xác định host alive không quan tâm tới port, dịch vụ nên ta thường xuyên sử dụng option `-sn`: Bỏ qua scan port chỉ scan host sống hay không để giảm thời gian scan.

Có cái nhìn tổng quan rồi, bây giờ giả sử muốn sử dụng chiến lược dùng ARP scan ta đi hỏi chat gpt thôi.

![chat gpt]({{site.url}}/assets/img/2023/02/26/host_alive_arp.PNG)

Thêm option load from file nào

![chat gpt arp and load list]({{site.url}}/assets/img/2023/02/26/combine_list_arp.PNG)

Với các chiến lược còn lại ta làm tương tự thôi. Cái này easy quá phải không

# Nmap Port scan ~ Xác định các port được mở trên host

Trước khi bắt đầu ta cần hiểu một số trạng thái của cổng (port) mà nmap có thể trả lại khi ta thực hiện scan. Các trạng thái có thể xảy ra bao gồm:

+ `Open` (mở): Điều này có nghĩa là một ứng dụng đang lắng nghe kết nối trên cổng đó.
+ `Closed` (đóng): Điều này có nghĩa là cổng không có ứng dụng nào lắng nghe kết nối.
+ `Filtered` (được lọc): Điều này có nghĩa là cổng đó đang bị chặn bởi tường lửa hoặc bởi các thiết bị bảo mật khác, khiến Nmap không thể xác định trạng thái thực sự.
+ `Unfiltered` (không được lọc): Điều này có nghĩa là Nmap có thể tiếp cận cổng nhưng không thể xác định chính xác trạng thái của nó.
+ `Open|Filtered` (mở hoặc được lọc): Điều này có nghĩa là Nmap không thể xác định chắc chắn liệu cổng đó có được mở hay không do bị chặn bởi tường lửa hoặc các thiết bị bảo mật khác.

Khi cần xác định một/các port được mở trên 1 host ta có thể sử dụng một số chiến lược/kỹ thuật scan như sau:

![Port scan]({{site.url}}/assets/img/2023/02/26/port_scan.png)

TCP là giao thức truyền tải hướng kết nối. Có quá trình bắt tay 3 bước để khởi tạo kết nối và thông báo khi đóng kết nối mô tả bằng hình bên dưới (Cái này quá quen rồi).

![TCP flow]({{site.url}}/assets/img/2023/02/26/tcp_full.jpg)

Các chiến lược TCP scan hầu hết thực hiện bằng việc gửi một hoặc 1 số gói tới đích cần scan và **nghe ngóng** kết quả phàn hồi để xác định port mở hay đóng. Chi tiết việc **gửi** và **ngóng** được mô tả khá dễ hiểu bằng các hình sau trong cheetsheet:

![Tacstic Port scan]({{site.url}}/assets/img/2023/02/26/tacstic_scan.PNG)

Còn với UDP thì khác, không thực hiện bắt tay khi mở kết nối nên xác định trạng thái port có thể phức tạp hơn và đôi khi là mất nhiều thời gian hơn. Chiến lược scan của nmap trong trường hợp này thường là cố gắng gửi một hoặc một số gói tin UDP tới các cổng của mục tiêu cần nhắm tới. Với hầu hết các cổng các gói tin gửi đi này thường không có nội dung (payload) ngoại trừ một số cổng thông thường thì có thể gửi thêm payload. Sau đó tcpdump sẽ chờ phản hồi từ phía target. Dựa vào việc phản hồi hay không mà nó xác định trạng thái của cổng.

![Udp Port scan]({{site.url}}/assets/img/2023/02/26/udp_response.PNG)

Tới đây ta vẫn tiếp tục áp dụng phương án sử dụng chat gpt để tìm hiểu chi tiết khi đã biết được tổng quan về các chiến lược scan như trên.

# Scan OS, service và sử dụng NSE (Nmap Scripting Engine)

### Scan OS

Nmap cũng hỗ trợ ta xác định OS của target dựa trên fingerprint của OS. Có thể sử dụng cú pháp sau để sử dụng tính năng này

`nmap -O <target>` 

### Scan service

Ngoài nmap cũng hỗ trợ ta xác định các dịch vụ (Dịch vụ là gì VD: http,https,ftp,...) và thông tin liên quan của dịch vụ (version) đang sử dụng port open trên target. Sử dụng option `-sV` cho tính năng này

`sudo nmap -sV --version-intensity 9 <target>` 

Trong đó: `-sV`: Là option chỉ ra rằng nmap sẽ scan detect service .`--version-intensity [0-9]` là option để tăng mức độ chi tiết của thông tin liên quan tới dịch vụ ta thực hiện scan. Chỉ số càng cao mức độ chi thông tin chi tiết càng nhiều. 

### NSE script trong nmap

Script trong Nmap là các chương trình viết bằng ngôn ngữ Lua, được sử dụng để tự động hóa các nhiệm vụ trong quá trình quét mạng. Các script này được sử dụng để thực hiện các tác vụ như phát hiện lỗ hổng bảo mật, thu thập thông tin từ các dịch vụ mạng và xác định các thiết bị trong mạng

nmap có rất nhiều script mặc định lưu tại vị trí sau `/usr/share/nmap/scripts/` . Để gọi toàn bộ script tại vị trí này ta sử dụng option `-sC` lúc này nmap sẽ tự động load và sử dụng toàn bộ các script. Ví dụ: `nmap -sC 192.168.1.1 -p 80`

Ngoài ra ta cũng có thể tự gọi script cụ thể bằng option sau

`nmap -sV -script=<script name> <target>` . Ví dụ: `nmap -sV -script=http-title,http-headers 192.168.1.1 -p 80`


# Lưu kết quả nmap ra file

Mặc định nmap sẽ đẩy kết quả scan ra màn hình console. Ta có thể đẩy kết quả này ra file với một số định dạng như sau:

![OUtput Port scan]({{site.url}}/assets/img/2023/02/26/output.PNG)

# Xào nấu?

Phía trên chúng ta nói rất nhiều về các option riêng rẽ của nmap. Tới đây nói về một chuyện thú vị hơn đó là việc kết hợp nhiều option để thực hiện việc scan. Việc kết hợp này hoàn toàn tuân theo quy luật sử dụng kết hợp các option của command trên linux.

`sudo nmap kenh14.vn -sT -sV -sC -O`  => Đây là ví dụ việc kết hợp các option scan TCP connect ở pharse host discovery (Ta có thể thay đổi các cơ chế host discovery khác bằng các option ở phía trên), sau đó nếu host đã sống (alive/online) ta sẽ detect OS, detect Service rồi chạy toàn bộ các script mặc định tới target là địa chỉ kenh14.vn

Nếu ta không chỉ định port thì mặc định nmap sẽ scan 1000 ports phổ biến. Giờ muốn scan một port thì ta chỉ thêm option về port vào command `-p80`.

`sudo nmap kenh14.vn -sT -sV -sC -O -p 80`

Còn nếu muốn scan toàn bộ 65535 port TCP thì ta sử dụng option -p-

`sudo nmap kenh14.vn -sT -sV -sC -O -p-`

OK ở đây tôi chỉ lấy một ví dụ để minh họa thế thôi, thực tế nếu bạn có nhu cầu nào thì các bạn tùy biến cho phù hợp. 

# Cuối cùng hãy nhớ!

Bài viết khá dài và nhiều nội dung nhưng tôi nghĩ điều quan trọng là bạn nắm được các tính năng của nmap một cách có hệ thống thôi, còn lại chi tiết hãy để Google và chat GPT lo. Cái đầu của chúng ta quá nhỏ bé để nhớ tất cả mọi thứ phải không.

Thks bạn đã đọc bài viết của tôi.