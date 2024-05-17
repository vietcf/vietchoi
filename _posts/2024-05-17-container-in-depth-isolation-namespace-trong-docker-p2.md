---
layout: post
title: "Container in depth - Cô lập tiến trình, tính năng Namespace trên linux - điểm khởi nguồn của container, P2"
author: "vietcf"
categories: job
tags: ['Container']
image: assets/img/2024/05/17/0-intro-isolation-container.png
---

Khi sử dụng Container, đặc biệt khi attach một console vào ta cảm giác như container giống y hệt các máy ảo độc lập chạy công nghệ ảo hóa truyền thống ta hay sử dụng. Độc lập với nghĩa là một Container hầu như chúng không thể nhìn/can thiệp vào các Container cũng như máy chủ Host. Trong bài viết này, chúng ta sẽ tìm hiểu cách công nghệ Container thực hiện việc việc cô lập (Isolation) này. 

Để thực hiện việc này, Container Linux không chỉ sử dụng một mà là kết hợp một số cơ chế/layer khác nhau để thực hiện việc Isolation. Các layer này sẽ được mô tả ngắn gọn ở hình bên dưới (mô tả chi tiết ở sẽ phân tích ở từng bài viết sau). Chúng đều là các tính năng hoàn toàn độc lập nhau và có sẵn trên linux nên ta có thể sử dụng chúng độc lập với nhau và độc lập với Container (Không nhất thiết phải sử dụng container mới có thể sử dụng được chúng). 

![isolation level]({{site.url}}/assets/img/2024/05/17/0-isolation-level.png)


So với các máy ảo, một trong những điểm mạnh mẽ hơn của việc Isolation sử dụng Container Linux là nó cung cấp sự linh hoạt trong việc kiểm soát mức độ Isolation. Tuy nhiên, điều này cũng có thể dẫn đến những điểm yếu về mặt bảo mật. Khi chúng ta hiểu thêm về cách hoạt động của các layer Isolation trong container, chúng ta sẽ thấy rằng chúng hoàn toàn có thể được điều chỉnh để phù hợp với các tình huống khác nhau. Trong bài viết này chúng ta cũng sẽ sử dụng các công cụ của Linux để tương tác với các layer này và phân tích các vấn đề bảo mật liên quan đến chúng.

Bài viết này sẽ tập trung vào cơ chế đầu tiên, **Linux namespaces** là cơ chế quan trọng nhất và hầu như được sử dụng ở tất cả các công nghệ Container hiện tại. 

# Namespaces

Linux namespaces: cho phép hệ điều hành cung cấp cho một tiến trình một "cái nhìn" (~ View) cô lập về một hoặc nhiều tài nguyên hệ thống. Hiện tại, Linux hỗ trợ tám namespaces:

- Mount
- PID
- Network
- Cgroup
- IPC
- Time
- UTS
- User

Namespaces là một phần quan trọng trong việc bảo mật container, vì chúng giới hạn View của process bên trong container đối với phần còn lại của máy chủ (Quyết định một process trong container có thể nhìn/can thiệp vào tài nguyên nào). Hiểu cách hoạt động của các Namespaces cũng có thể hữu ích để bảo mật container và khắc phục sự cố. Namespaces khá linh hoạt, vì chúng có thể được áp dụng một cách riêng lẻ hoặc theo nhóm cho một hoặc nhiều processes. Ta cũng có thể sử dụng các công cụ tiêu chuẩn (tool) của Linux để tương tác với chúng, mở ra một số khả năng thú vị cho việc gỡ lỗi container và thực hiện các cuộc điều tra bảo mật trên các phiên bản container đang chạy.

Chúng ta có thể sử dụng lệnh `lsns` để xem các Namespaces trên máy chủ, như được thể hiện ở hình bên dưới. Tool này đi kèm với gói **util-linux** trên hầu hết các bản phân phối Linux.

![lsns level]({{site.url}}/assets/img/2024/05/17/1-namespace-list.png)

Trường "NPROCS" cho thấy rằng có 261 process đang sử dụng Namespaces đầu tiên trên máy chủ này. Chúng ta cũng có thể thấy rằng một số process đã được gán cho các Namespace riêng của chúng (thường là mnt hoặc uts). Những process này hoàn toàn không được khởi tạo bởi Docker, tuy nhiên chúng đang sử dụng các Namespace nhất định để cách ly tài nguyên của riêng chúng với các process khác trên hệ thống.

Sau khi sử dụng Docker để khởi động một container mới bằng lệnh ```docker run -d nginx```, chạy lại sudo lsns sẽ hiển thị các Namespaces mới cho các process NGINX. Theo mặc định, Docker sử dụng các Namespace  mnt, uts, ipc, pid, và net khi tạo một container.

![lsns nginx]({{site.url}}/assets/img/2024/05/17/1-namespace-list-nginx.png)

Mỗi không gian tên này cung cấp một mức độ cô lập khác nhau và có thể được sử dụng độc lập hoặc kết hợp để đáp ứng các yêu cầu cụ thể của hệ thống và ứng dụng.

## Mount namespace

Mount Namespaces (mnt) cung cấp cho một process một View cô lập về hệ thống tập tin (filesystem ). Điều này để đảm bảo rằng một/tập process sẽ được sở hữu một filesystem riêng nó và không can thiệp vào các filesystem thuộc về các tiến trình khác cũng như hệ thống tệp tin của máy chủ Host. Khi sử dụng mnt Namespaces, một tập hợp các filesystem mounts point mới được cung cấp cho process thay cho các filesystem mounts point mà nó nhận được theo mặc định.

Chúng ta có thể xem các Mount Namespaces nào được sử dụng bởi process bằng cách nhìn vào hệ thống tập tin /proc; thông tin này nằm trong /proc/[PID]/mountinfo. 

![proc nginx]({{site.url}}/assets/img/2024/05/17/2-namespace-mnt-1.png)

Hoặc chúng ta cũng có thể sử dụng một công cụ như findmnt, công cụ này sẽ cung cấp một phiên bản được định dễ nhìn hơn rất nhiều. Khi sử dụng các công cụ này, trước tiên chúng ta cần tìm ID tiến trình của container. Một cách để làm điều này là sử dụng lệnh inspect của Docker. ```docker inspect -f '{{.State.Pid}}' [CONTAINER]``` sẽ trả về thông tin PID, cho phép chúng ta chạy findmnt -N [PID] để lấy thông tin mount.

![findmnt nginx]({{site.url}}/assets/img/2024/05/17/2-namespace-mnt-0.png)



## PID namespace


## Network namespace

## Cgroup namespace

## IPC namespace

## UTS namespace

## Time namespace

## User namespace

