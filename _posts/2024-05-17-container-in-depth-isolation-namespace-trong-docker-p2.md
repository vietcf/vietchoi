---
layout: post
title: "Container in depth - Cô lập tiến trình, tính năng Namespace trên linux - điểm khởi nguồn của container, P2"
categories: job
tags: ['Container']
image: assets/img/2024/05/17/0-intro-isolation-container.png
---

Khi sử dụng Container, đặc biệt khi attach một console vào ta cảm giác như container giống y hệt các máy ảo độc lập chạy công nghệ ảo hóa truyền thống ta hay sử dụng. Độc lập với nghĩa là một Container hầu như chúng không thể nhìn/can thiệp vào các Container khác cũng như máy chủ Host (Kể từ đây khi nhắc tới Host hiểu là máy chủ Linux gốc chạy Container bên trên đó). Trong bài viết này, chúng ta sẽ tìm hiểu cách công nghệ Container thực hiện việc việc cô lập (Isolation) này. 

Để thực hiện việc này, Container Linux không chỉ sử dụng một mà là kết hợp một số cơ chế/layer khác nhau để thực hiện việc Isolation. Các layer này sẽ được mô tả ngắn gọn ở hình bên dưới (mô tả chi tiết ở sẽ phân tích ở từng bài viết sau). Chúng đều là các tính năng hoàn toàn độc lập nhau và có sẵn trên linux nên ta có thể sử dụng chúng một cách độc lập với nhau và đương nhiên là độc lập với Container (Theo nghĩa không nhất thiết phải sử dụng container mới có thể sử dụng được chúng). 

![isolation level]({{site.url}}/assets/img/2024/05/17/0-isolation-level.png)


So với các máy ảo, một trong những điểm mạnh mẽ hơn của việc Isolation sử dụng Container Linux là nó cung cấp sự linh hoạt trong việc kiểm soát mức độ Isolation. Tuy nhiên, điều này cũng có thể dẫn đến những điểm yếu về mặt bảo mật. Khi chúng ta hiểu thêm về cách hoạt động của các layer Isolation trong container, chúng ta sẽ thấy rằng chúng hoàn toàn có thể được điều chỉnh để phù hợp với các tình huống khác nhau. Trong bài viết này chúng ta cũng sẽ sử dụng các công cụ của Linux để tương tác với các layer này và phân tích các vấn đề bảo mật liên quan đến chúng.

Bài viết này sẽ tập trung vào cơ chế đầu tiên, **Linux namespaces** là cơ chế quan trọng nhất và hầu như được sử dụng ở tất cả các công nghệ Container hiện tại. 

# Namespaces

Linux namespaces: Cho phép hệ điều hành cung cấp cho một tiến trình (process) có "cái nhìn" (~ View) cô lập về một hoặc nhiều tài nguyên hệ thống. Hiện tại, Linux hỗ trợ tám namespaces:

- Mount
- PID
- Network
- Cgroup
- IPC
- Time
- UTS
- User

Namespaces là một phần quan trọng trong việc bảo mật container, vì chúng giới hạn View của process bên trong container đối với phần còn lại của máy chủ (Quyết định một process trong container có thể nhìn/can thiệp vào tài nguyên nào). Hiểu cách hoạt động của các Namespaces cũng có thể hữu ích để bảo mật container và khắc phục sự cố. Namespaces khá linh hoạt, vì chúng có thể được áp dụng một cách riêng lẻ hoặc theo nhóm cho một hoặc nhiều processes. Ta cũng có thể sử dụng các công cụ tiêu chuẩn (tool) của Linux để tương tác với chúng, mở ra một số khả năng thú vị cho việc gỡ lỗi (troubleshoot) Container hoặc thực hiện các cuộc điều tra bảo mật trên các phiên bản Container đang chạy.

Chúng ta có thể sử dụng lệnh `lsns` để xem các Namespaces trên máy chủ như được thể hiện ở hình bên dưới. Tool này đi kèm với gói **util-linux** trên hầu hết các bản phân phối Linux.

![lsns level]({{site.url}}/assets/img/2024/05/17/1-namespace-list.png)

Trường "NPROCS" cho thấy rằng có 261 process đang sử dụng Namespaces đầu tiên trên máy chủ này. Ta cũng có thể thấy rằng một số process đã được gán cho các Namespace riêng của chúng (thường là mnt hoặc uts). Những process này hoàn toàn không được khởi tạo bởi Docker, tuy nhiên chúng đang sử dụng các Namespace nhất định để cách ly tài nguyên của riêng chúng với các process khác trên hệ thống.

Sau khi sử dụng Docker để khởi động một container mới bằng lệnh ```docker run -d nginx```, chạy lại sudo lsns sẽ hiển thị các Namespaces mới cho các process NGINX. Theo mặc định, Docker sử dụng các Namespace  mnt, uts, ipc, pid, và net khi tạo một container.

![lsns nginx]({{site.url}}/assets/img/2024/05/17/1-namespace-list-nginx.png)

Mỗi namespace này cung cấp một mức độ cô lập khác nhau và có thể được sử dụng độc lập hoặc kết hợp để đáp ứng các yêu cầu cụ thể của hệ thống và ứng dụng.

### Mount namespace

Mount Namespaces (mnt) cung cấp cho một process một View cô lập về hệ thống tập tin (filesystem ). Điều này để đảm bảo rằng một (/nhóm) process sẽ được sở hữu một filesystem riêng nó và không can thiệp vào các filesystem thuộc về các tiến trình khác cũng như hệ thống tệp tin của máy chủ Host. Khi sử dụng mnt Namespaces, một tập hợp các filesystem mounts point mới được cung cấp cho process thay cho các filesystem mounts point mà nó nhận được theo mặc định.

Ta có thể xem các Mount Namespaces nào được sử dụng bởi process bằng xem trong thư mục /proc/[PID]; thông tin này nằm trong /proc/[PID]/mountinfo. 

![proc nginx]({{site.url}}/assets/img/2024/05/17/2-namespace-mnt-1.png)

Hoặc ta cũng có thể sử dụng một công cụ như findmnt, công cụ này sẽ cung cấp một phiên bản được định dễ nhìn hơn rất nhiều. Khi sử dụng các công cụ này, trước tiên ta cần tìm ID tiến trình của container. Một cách để làm điều này là sử dụng lệnh inspect của Docker. ```docker inspect -f '{{.State.Pid}}' [CONTAINER]``` sẽ trả về thông tin PID. Sau đó ta chạy lệnh findmnt -N [PID] để lấy thông tin các mount point.

![findmnt nginx]({{site.url}}/assets/img/2024/05/17/2-namespace-mnt-0.png)

Trong ảnh chụp màn hình phía trên, ta có thể thấy rằng Container của ta có thư mục gốc / được mount vào một thư mục nằm trong /var/lib/docker - nơi Docker lưu trữ tất cả các image và container filesystem.

Một điểm quan trọng liên quan đến bảo mật cần nhớ là tất cả các hệ thống tập tin gốc / được sử dụng bởi các Container trên máy chủ sẽ nằm trong một thư mục được quản lý runtime process gốc của container (/var/lib/docker/ theo mặc định). Vì vậy, cần đảm bảo rằng quyền truy cập vào thư mục này được kiểm soát chặt chẽ và nó đang được giám sát để phát hiện các truy cập trái phép.

### PID namespace

Chắc hẳn chúng đều biết mỗi process trên linux có một *process ID* (PID) như một định danh cho một process. Nhắc lại ở bài viết trước ta thấy rằng các PID này đươc liên kết với một thư mục trong ```/proc``` virtual filesystem.

**PID** **namespace** cho phép một (/nhóm) process có thể được isolate về mặt “không gian” (space) Process ID (PID) khác nhau. Việc isolate này có thể hiểu như sau:

*PID namespaces isolate the process ID number space, meaning that processes in different PID namespaces can have the same PID ~ Đoạn này tạm hiểu là các process ở các namespace khác nhau hoàn toàn có cùng PID*

Việc có “không gian” PID khác nhau là quan trọng vì trên một linux Host thông thường ta biết rằng hai process không thể có cùng một PID. Nếu chỉ xem xét một hệ thống đơn lẻ tất nhiên không thể xảy ra xung đột về PID vì hệ thống liên tục tăng số ID của tiến trình lên và không bao giờ gán cùng một số PID hai cho 2 process. Tuy nhiên khi xử lý các container trên nhiều host, vấn đề này trở nên phức tạp hơn. Như mô tả trong man page:

*PID namespaces allow containers to provide functionality such as suspending/resuming the set of processes in the container and migrating the container to a new host while the processes inside the container maintain the same PIDs. ~ Khi một container được migrate giữa 2 Host muốn bảo toàn PID và không gây xung đột với PID trên hệ thống đích thì rõ ràng cần một PID namespace riêng biệt.*

Ngoài ý nghĩa kể trên, việc isolation các PID namespaces giúp cho các tiến trình trong một Namespace có cơ chế hoạt động tương tự như PID của Host ~ Nói chung các nhà phát triển linux họ thích vậy, cái project namespace là họ muốn tái hiện một space cho một (/nhóm) process với cấu trúc tương tự Host chủ. Các PID bên trong namespace mới tạo ra sẽ bắt đầu từ 1 và nó là PID của process đầu tiên ~ process init. Dĩ nhiên với từng namespace các process init này luôn có PID=1 và phải hoàn toàn độc lập với nhau cũng như độc lập với PID init process của Host. Và để làm được điều này rõ ràng là cần phải có một cơ chế như mô tả của PID namespace nhắc tới ở trên rồi.

Cần chú ý rằng là là bất kỳ tiến trình nào có PID=1 đều quan trọng đối với sự tồn tại của namespace. Nếu Process có PID=1 bị exit vì bất kỳ lý do nào, kernel sẽ gửi một tín hiệu **SIGKILL** đến tất cả các tiến trình còn lại trong namespace đó, kết quả là  namespace bị shutting down.

Để kiểm chứng điều này ta có thể sử dụng **nsenter** để hiển thị danh sách các process đang chạy bên trong một Container. Ta cần có một Container image có binary của trương trình (lệnh) ps. Sau đó truyền vào PID và mnt namespace của container cần chạy lệnh ps. Ở đây tôi sử dụng busybox image chạy như một Container ở background với lệnh ```docker run --name busyback -d busybox top``` (Chạy lệnh top trong container để nó không bị Exit). Sau đó, ta sẽ sử dụng ```docker inspect``` để lấy PID của Container của ta, sau đó sử dụng nsenter để kiểm tra danh sách tiến trình bên trong Container. Kết quả được hiển thị dưới đây. Kết quả cho phép ta nhìn thấy tiến trình top của ta đang chạy.

![pid ns 0]({{site.url}}/assets/img/2024/05/17/3-pid-namespace-0.png)

Một cách khác để minh họa PID namespace là sử dụng công cụ unshare của Linux để chạy một tập các namespaces. Để thực hiện việc này ta chạy lệnh ```sudo unshare --pid --fork --mount-proc /bin/bash``` . Kết quả của việc thực thi lệnh này là ta sẽ tạo ra một bash shell process trong một PID namespace mới.

![pid ns 1]({{site.url}}/assets/img/2024/05/17/3-pid-namespace-1.png)

Một điểm thú vị là khi hiểu về PID namespace cũng có thể cho phép một Container xem các tiến trình đang chạy trong một Container khác. Tùy chọn --pid trên lệnh docker run cho phép ta start một Container cho mục đích troubleshoot trong PID namespace của một container khác.

Để minh họa điều này, ta sẽ start một container máy chủ web bằng cách chạy lệnh ```docker run -d --name=webserver nginx```. Sau đó ta sẽ bắt đầu một debug container bằng cách chạy lệnh ```docker run -it --name=debug --pid=container:webserver raesene/alpine-containertools /bin/bash```. Nếu sau đó ta chạy lệnh ```ps -ef``` từ container debug vừa được start ta có thể thấy các tiến trình từ container máy chủ web ban đầu cũng như các tiến trình chạy trong debug container.


![pid ns 2]({{site.url}}/assets/img/2024/05/17/3-pid-namespace-2.png)

Chia sẻ PID namespace giữa các container cũng có thể được thực hiện trong các cụm Kubernetes, việc này có thể hữu ích để troubleshoot vì cơ bản cơ chế của chúng cũng tương tự ở trên mà thôi. 

### Network namespaces

Tiếp theo trong danh sách có một namespace khác gọi là **net namespaces**. Đây là thành phần chịu trách nhiệm cung cấp network “space” (không gian độc lập về network) cho một (/nhóm) Process (interfaces, routing, etc.) ~ mỗi **net namespaces** hoàn toàn có thể có một IP, một bảng route riêng. Điều này là hữu ích để đảm bảo rằng các Process trong các net namspace khác nhau có thể liên kết với các port mà chúng cần kết nối mà không gây cản trở lẫn nhau và các lưu lượng mạng có thể được định tuyến đến các ứng dụng cụ thể mà không xảy ra conflict lẫn nhau. 

*Để lấy ví dụ hãy hình dung nếu không netnamspace ta rất khó để thực hiện 2 ứng dụng http cùng chạy port 80 trên một máy chủ host. Tuy nhiên thực tế nếu dùng container ta có thể chạy 2 container http sử dụng port 80 trên một máy chủ một cách dễ dàng. Điều này có được là nhờ net namespace.*

Tương tự như các namespace đã được đề cập trước đó, bạn có thể tương tác với **net namespaces** bằng cách sử dụng các công cụ Linux tiêu chuẩn như ```nsenter```. Bước đầu tiên là lấy PID của Container sau đó có thể sử dụng nsenter để xem các thông tin trong  **net namespaces** của Container ví dụ như IP hoặc bảng route một cách dễ dàng.

Một điểm quan trọng ở đây là lệnh ip ta đang chạy là của Host và không cần phải tồn tại bên trong Container. Điều này thuật hữu ích trong trường hợp cần troubleshoot các vấn đề mạng cho các Container không có cài đặt nhiều tool/toy (Do bình thường các image thường được đóng gói cố gắng minimun nhất có thể).

Một phần khác của các công cụ Linux có thể được sử dụng để tương tác với **net namespaces** là lệnh ip thông qua lệnh netns. Lệnh phụ này thường cho phép bạn tương tác với các không gian tên mạng khác nhau trên một hệ thống. Tuy nhiên, lưu ý rằng nó không hoạt động trong Docker vì các liên kết mềm mà netns phụ thuộc vào không tồn tại.

Ta cũng có thể sử dụng Docker để chia sẻ **net namespaces**, tương tự như cách chia sẻ **PID namespaces**. Ta có thể khởi chạy một troubleshoot container, troubeshoot Container này có thể có các công cụ như tcpdump đã được cài đặt và kết nối nó với Net của Container đang chạy.

Chạy docker run -it --name=debug-network --network=container:webserver raesene/alpine-containertools /bin/bash sẽ cho phép ta kết nối vào mạng của một container hiện có được gọi là "webserver." Khi nó được khởi chạy, ta có thể chạy netstat -tunap để xem các cổng lắng nghe, và nó sẽ hiển thị máy chủ web đang chạy trên cổng 80 từ container khá


### Cgroup namespace

### IPC namespace

### UTS namespace

### Time namespace

### User namespace

# Tham khảo

[https://securitylabs.datadoghq.com/articles/container-security-fundamentals-part-2/](https://securitylabs.datadoghq.com/articles/container-security-fundamentals-part-2/)

[https://www.redhat.com/sysadmin/pid-namespace](https://www.redhat.com/sysadmin/pid-namespace)