---
layout: post
title: "Active Directory Security - Quay lại các khái niệm cơ bản, P1"
author: "vietcf"
categories: job
tags: ['Active Directory']
image: assets/img/2024/04/04/intro.png
---

Như đã giới thiệu, để bắt đầu tôi sẽ nhắc qua một số khái niệm "theo tôi nghĩ" là quan trọng trước khi nói về AD Security. Các khái niệm này có thể đã quen thuộc với mọi người, đặc biệt là những ai từng làm việc với AD. Tuy nhiên tôi vẫn muốn nhắc lại vì đôi khi có nhiều cái quen quá đâm ra chúng ta lại không để ý hoặc nhầm lẫn, vậy thôi cứ làm rõ ở đây với nhau đi.

## MS sinh ra Active Directory làm gì?

Để "hình dung" AD là gì xin lấy cái hình mô tả sơ qua về nó như sau:

![What is active Directory]( {{site.url}}/assets/img/2024/04/10/0-active-directory.png)

Qua cái hình ta có thể thấy AD nó quá quan trọng phải không. Là nơi quản lý hầu hết các tài sản IT của tổ chức. Từ Personal Computer/Workstation/Server sau đó là User. Chỉ nói riêng về AD User hay được sử dụng là nguồn xác thực cho hầu hết các hệ thống trong một tổ chức. Các hệ thống này rất đa dạng: từ Network (Switch, Router,...); Hạ tầng (Email, Monitor, Chat, Ảo hóa, ...); Bảo mật (Firewall,IPS/IDS, VPN...) cho đến các hệ thống nghiệp vụ (ERP/Core, ...). Do vậy chén được AD thì có kha khá thứ hay ho để làm đó.

Nói về AD là gì thì dài dòng vậy, nhưng theo ý kiến cá nhân tôi MS làm ra AD để giải quyết 2 vấn đề chính nhất dưới đây:

+ Quản lý định danh tập trung trong tổ chức. Định danh này là các đối tượng cụ thể như User, Group, Computer, Máy in, ...

+ Quản lý chính sách tập trung áp dụng lên định danh. Ví dụ như Chính sách về mật khẩu, truy cập, chính sách log audit, ...

## Một chút về các đối tượng trên Active Directory

Khi làm với AD ta đã quá quen thuộc với các khái niệm: Domain, User, Group, OU, Computer, Printer, ... các đối tượng này trên AD được gọi chung là  **Security principals**. Thực tế AD lưu trữ các đối tượng này sử dụng cấu trúc theo chuẩn LDAP (Hầu như tất cả lưu theo cấu trúc này). Do vậy mỗi đối tượng này đều có tập các **Thuộc tính - Giá trị** (key-value) của riêng nó và được lưu trong một Ldap Object.

![Ldap Object attributes]( {{site.url}}/assets/img/2024/04/10/01_ldap_object.png)

Nói thì nghe hài hước nhưng thỉnh thoảng vẫn có một số khái niệm lẫn lộn cần nhắc lại ở đây.

Domain vs Domain Controller: Một Domain có các Domain Controllers là các các máy chủ (Đối tượng Computer) có vai trò đặc biệt để vận hành hoạt động của Domain. Các máy chủ này mặc định được bố trí vào một OU riêng tách biệt không lẫn lộn với các các đối tượng khác. Cứ join thêm một Domain Controller (Bao gồm cả Read Only Domain Controller - RODC) thì nó auto được nhét vào đây.

![Domain Controller OU]( {{site.url}}/assets/img/2024/04/10/02-domain-controllers.png)

Ngoài ra có 2 đối tượng Group và OU (Organizational Unit) ai mới làm AD cũng hay bị nhẫm lần giữa chúng. Hãy nhớ rằng Group sinh ra để nhóm các đối tượng quản lý và phân quyền truy cập tới tài nguyên (File/Thư mục) hoặc chính sách gửi mail trong khi OU tập trung vào việc nhóm các đối tượng lại với nhau để thiết lập các chính sách (VD: Chính sách mật khẩu, Audit log, ...). 

Trong AD cũng định nghĩa sẵn một số Group/User có sẵn (built-in User/Group). Các User/Group này một số là các đối tượng với *Đặc quyền** đặc biệt trên hệ thống ~ Privilege User/Group trên hệ thống Active Directory. Nhóm này cần được tập trung tối đa trong việc quản lý/sử dụng vì mất một user trong nhóm này là vô cùng nghiêm trọng. Tôi chia ra làm 2 nhóm với Level khác nhau.

Nhóm "Đặc biệt" quan trọng (Nhóm này ở dạng động vật quý hiếm cần bảo vệ tối đa) bao gồm:

![Privileage Group]( {{site.url}}/assets/img/2024/04/10/03-built-in-privilege-user-group.png)

Nhóm "Quan trọng" (Nhóm cần bảo vệ) bao gồm:

Đọc nhiều hơn bạn có thể đọc ở [Link này](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/plan/security-best-practices/appendix-b--privileged-accounts-and-groups-in-active-directory#domain-admins). Nhưng tôi nghĩ là không nên vì đặc điểm của MS là viết dai viết dài đâm ra thành viết dại chả ai đọc, các nội dung chính tôi đã tóm tắt ở trên rồi.

Group Policy: Một Group Policy là một tập các chính sách (Policy) áp dụng lên các đối tượng trên AD. Nói lý thuyết là vậy nhưng tôi thấy chủ yếu các Policy được "áp" lên 2 nhóm đối tượng chính và quan trọng nhất là Computer và User. Vì rõ là khi ta show ra chi tiết sẽ nhìn thấy 2 phần riêng biệt rõ ràng Computer và User.

![Computer with User]( {{site.url}}/assets/img/2024/04/10/04-computer-with-user.png)

Tôi nhấn mạnh chỗ này để bạn hiểu rằng nếu mà muốn cấu hình lên các Workstation/Member server một Policy nào đó ở phần Computer thì phải nhét chúng vào OU có Group policy thiết lập Policy đó. Tương tự nếu muốn cấu hình Policy ốp ở phần User lên một User thì phải nhét User vào đúng OU có chứa Group Policy được thiết lập. Có nhiều bạn thắc mắc sao tôi cấu hình Policy mà không ăn thì hóa ra là cấu hình ở phần Computer nhưng lại không nhét Workstation/Member server vào OU thiết lập Policy lại đi nhét User vào đó - cái này không ăn là phải.


## Các cơ chế xác thực trên AD điều cần phải biết,

### Kerberos Authentication

### NetNTLM 

## Sự kết nối giữa nhiều Domain trên AD





