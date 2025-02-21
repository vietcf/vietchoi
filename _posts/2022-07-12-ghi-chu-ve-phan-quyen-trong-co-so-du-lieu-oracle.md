---
layout: post
title: "Ghi chú về phân quyền trong cơ sở dữ liệu Oracle,"
categories: job
tags: ['Security']
image: assets/img/2022/07/12/intro.png
---

Tôi không phải là một DBA chuyên nghiệp, kiến thức về Oracle DB của tôi còn rất hạn chế. Tuy nhiên, do yêu cầu công việc, tôi phải tìm hiểu những khái niệm cơ bản về Oracle, đặc biệt là cơ chế phân quyền của nó. Bài viết này sẽ tập trung làm rõ các thông tin cơ bản về Oracle và cơ chế phân quyền của nó cho một người mới bắt đầu như tôi. 

>P/S: Nhiều anh em cứ bảo rảnh thế có thời gian viết blog nhưng sự thật bận sml (vài tháng mới post được một bài). Chỉ là Note là thói quen tốn thời gian nhưng vẫn muốn giữ vì nhiều lúc đọc xong không note quên luôn sau cần lại mất công đọc lại. Notion Collection cá nhân còn rất nhiều thứ nhưng không có thời gian biên tập thành post trên blog. 

![Notion]( {{site.url}}/assets/img/2022/07/12/notion.png)

## Các khái niệm căn bản trong OracleDB

Để nói về Oracle tôi thấy người ta hay chia ra kiến trúc Process - MEM - File store. Cũng điểm qua để xem nó là cái chi. 

### Process trong Oracledb

Oracle chạy theo mô hình client -> Server. Trên client sẽ chạy 1 process gọi là **Client Process**. Khi người dùng kết nối tới server, client process này sẽ giao tiếp với **Server Process** thông qua các **Listener**. Quan hệ giữa client process và server process này có thể là 1 – 1 hoặc 1 –n  (tùy loại Dedicated Server hoặc Shared Server). Mỗi server process được cấp 1 không gian bộ nhớ trên server gọi là **PGA** (Program Global Area).

Trên server có cái gọi là các **Background Process** – các process xử lý nội tại của oracle bao gồm: DBWn, CKPT, LGWR, ARCn, .. với nhiều mục đích: có cái thì ghi dữ liệu từ Memory xuống đĩa cứng, có cái thì dọn dẹp bãi chiến trường mỗi khi Server process hoạt động xong, có cái thì đứng ngoài ghi chép lại các thay đổi do Server process sinh ra. Chúng âm thầm, liên tục hoạt động đằng sau Database, để giữ cho Database luôn vận hành trơn tru.
 
### Cấu trúc bộ nhớ (MEM)

Phần này không nói nhiều vì căn bản cũng không liên quan nhiều ở nội dung hôm nay, chỉ xin post cái hình - cơ bản nhìn cũng khá dễ hiểu khỏi cần giải thích nhiều. 

![Process]( {{site.url}}/assets/img/2022/07/12/process.png)

Trong đó: **SGA** - System Global Area (SGA)

### Các loại file trong OracleDB

Cơ sở dữ liệu dùng để lưu trữ => Cốt yếu dữ liệu chắc chắn sẽ cần phải percistance dưới dạng các file vật lý trên ổ đĩa. Oracle có một số loại file quan trọng sau sau:

* Datafiles: Các database datafiles chứa toàn bộ dữ liệu trong database.
* Redo Log Files: Chức năng chính của redo log là ghi lại tất cả các thay đổi đối với dữ liệu trong database. Redo log files được sử dụng để bảo vệ database khỏi những hỏng hóc do sự cố. Các thông tin trong redo log file chỉ được sử dụng để khôi phục lại database trong trường hợp hệ thống gặp sự cố và không cho phép viết trực tiếp dữ liệu trong database lên các datafiles trong database
* Control Files: Control file chứa các mục thông tin quy định cấu trúc vật lý của database như: Tên của database, Tên và nơi lưu trữ các datafiles hay redo log files, Time stamp (mốc thời gian) tạo lập database. 

### Cấu trúc logical OracleDB

Tôi tạm chia phần này thành 2 thứ gọi là cấu trúc lưu trữ và cấu trúc tổ chức dữ liệu. 

* Cấu trúc lưu trữ sẽ mô tả việc lưu trữ các thành phần logic thế nào – Là một phép link giữa các thành phần logic và lưu trữ trên ổ cứng. 
* Cấu trúc tổ chức sẽ mô tả việc tổ chức các thành phần/đối tượng được tổ chức thế nào – Cái nào bao cái nào. 

#### Nói về cấu trúc lưu trữ trước đi

Có cái hình này

![storage arch]( {{site.url}}/assets/img/2022/07/12/storagearch.png)
 
Một database có thể được phân chia về mặt logic thành các đơn vị gọi là các **Tablespaces**.  Tablespaces thường bao gồm một nhóm các thành phần có quan hệ logic với nhau. Khi tạo Tablespace sẽ cho ta lựa chọn nơi lưu trữ các **Data file**, có thể chọn lưu trữ một hoặc nhiều data file. Mỗi table trong csdl sẽ được lưu trữ trong 1 **Segment** nằm trong 1 tables space. Mỗi row trong table được lưu trữ  trong một hoặc nhiều **Block**. Mỗi block thuộc các extends. Nói cho có chứ cũng không cần quan tâm chi tiết tới block, extend, segment làm gì vì phần này căn bản oracledb care cho ta hết rồi chỉ cần quan tâm tới mức Tablespace và Datafile thôi.

#### Nói về cấu trúc tổ chức có một số khái niệm như sau

* User: Người dùng trên oracle. Mỗi user có một username.
* Schema: Là tập các đối tượng được (tables, views, …) mà thuộc về một user.

Khi tạo user bằng lệnh create user  oracle sẽ tự động tạo một schema “rỗng” cho user tên schema mặc định là username luôn. Sau này ta sử dụng lệnh grant để thêm các đối tượng vào schema của user.

Nói hoài về database. Vậy database là gì? Database là thứ bao gồm tất cả users và dữ liệu của users (table, view, index, …). 

Từ các định nghĩa trên thấy mối quan hệ giữa các đối tượng đó: Database là thứ bao gồm tất cả. Một database có nhiều user mỗi user gắn với một schema là wrapper của các đối tượng user được phép sử dụng. Database có nhiều table, view, procedure, view, index, … và đương nhiên schema cũng có thể có nhiều table, view, procedure, view, index theo định nghĩa trên – Nghe na ná nhau nhưng qua mô tả trên rõ ràng là khác nhau mà. P/S: Là người đi từ mySQL sang tôi rất hay confuse giữa database và schema :v 

## Nói về vấn đề phân quyền trên oracle

Đây là phần tôi nghĩ là quan trọng ít nhất với những người như tôi người mà không phải DBA chỉ là system security culi.

### Nói về user

Một user sẽ có 1 username và phương thức xác thực nhất định (password hoặc key). User cũng có thể được gán một Profile định nghĩa các chính sách áp dụng cho một user: VD Độ dài và độ phức tạp password, lockout account, … thậm trí là giới hạn bộ nhớ sử dụng cho user.
Oracle có tạo sẵn một số user khi cài xong bao gồm:
 
### Nói về quyền

Quyền (privilege/Permission) là sự cho phép thực hiện 1 câu lệnh SQL nào đó hoặc được phép truy xuất đến một đối tượng nào đó (vd: quyền tạo bảng CREATE TABLE, quyền connect đến cơ sở dữ liệu CREATE SESSION, quyền truy vấn SELECT,…).

Quyền có thể gán trực tiếp cho user hoặc gián tiếp thông qua **Role** - nói sau (User được gán vào role, role định nghĩa tập quyền).

Có 2 loại quyền cơ bản là quyền hệ thống và quyền đối tượng.

![sys perm]({{site.url}}/assets/img/2022/07/12/overviewperm.png)

* Quyền hệ thống (System Privilege): là quyền thực hiện một tác vụ CSDL cụ thể hoặc quyền thực hiện một loại hành động trên tất cả những đối tượng trong schema của hệ thống. Vd: ALTER SYSTEM, CREATE TABLE, DELETE ANY TABLE,… User có thể cấp 1 quyền hệ thống nếu có một trong các điều kiện sau:
  - User đã được cấp quyền hệ thống với tùy chọn WITH ADMIN OPTION.
  - User có quyền GRANT ANY PRIVILEGE.

  ![sys perm]({{site.url}}/assets/img/2022/07/12/sysperm.png)

  - Oracle có một số quyền quản trị hệ thống (administrative privileges) đặc biệt cần biết:

  ![special perm]( {{site.url}}/assets/img/2022/07/12/specialrole.png) 

* Quyền đối tượng Quyền đối tượng (Object Privilege): là quyền thực hiện một hành động cụ thể trên một đối tượng cụ thể, dùng để quản lý việc truy xuất đến các đối tượng của schema cụ thể nào đó.User có thể cấp 1 quyền đối tượng nếu có một trong các điều kiện sau:
  - User có thể cấp bất kỳ quyền đối tượng trên bất kỳ đối tượng nào thuộc sở hữu của mình cho user khác.
  - User có quyền GRANT ANY OBJECT PRIVILEGE.
  -	User được cấp quyền đối tượng đó với tùy chọn WITH GRANT OPTION.
	![object perm]( {{site.url}}/assets/img/2022/07/12/objectperm.png) 

### Nói về Role

Oracle cũng hỗ trợ RBAC tạo ra các role định nghĩa một tập các quyền. User khi được gán vào role nào thì sẽ có quyền tương ứng nằm trong tập quyền của role

Oracle cũng định nghĩa sẵn một số "build-in" role
 
![buildin role]( {{site.url}}/assets/img/2022/07/12/buildinrole.png) 

Sẽ không nói về grant quyền là thế nào vì đó chỉ là vấn đề search google khi đã hiểu cơ bản những nội dung trên.