---
layout: post
title: "MITRE ATT&CK FRAMEWORK- Hướng dẫn cho người mới bắt đầu tìm hiểu"
author: "vietcf"
categories: job
tags: ['Security']
image: assets/img/2024/04/25/0-intro-mitre.jpeg
---

>Ngoại trừ các câu hỏi liên quan tới bản thân như "Tối nay ăn gì" là những câu hỏi khó trả lời nhất, còn hầu hết trong công việc tôi nghĩ những gì ta làm thường thì đã có người làm rồi việc chỉ là tìm nguồn tham khảo và làm lại **theo** sao cho tốt hơn (~ có phần sáng tạo của mình trong đó) mà thôi! 

Anh em làm liên quan tới ATTT chắc đều đã nghe về MITRE ATT&CK Framework. Nhưng cá nhân khi vào trang chủ của Project này tôi cũng thấy khá rối, cái này link với cái kia luẩn qua luẩn quẩn không biết bắt đầu tìm hiểu từ đâu. Bài viết này này được thiết kế để người đọc có cái nhìn một cách hệ thống và toàn diện về MITRE ATT&CK Framework. Tôi nghĩ nó là một điểm khởi đầu tốt giúp bạn khám phá và áp dụng nó cho tổ chức của mình.

![Mitre 01]({{site.url}}/assets/img/2024/04/25/1-mitre.png)

# **1. Introduction**

Nhà tội phạm học nổi tiếng Edmond Locard đã áp dụng nguyên lý "Mỗi tiếp xúc đều để lại dấu vết", nguyên lý này cũng được áp dụng vào tội phạm mạng. Tương tự như bất kỳ tội phạm nào khác, Tội phạm mạng/kẻ tấn công (attacker) thế nào cũng chắc chắn sẽ để lại dấu vết sau mỗi cuộc tấn công mạng, và mỗi dấu vết này được gọi là **Indicator of Compromise** (IoC). Một IoC là một bằng chứng cho thấy một cuộc tấn công mạng đã xảy ra.

Các IoC cung cấp thông tin quý giá về những gì đã xảy ra, giúp cho bên phòng thủ chuẩn bị cho các cuộc tấn công tương lai. Có thể ngăn chặn & phát hiện và phản ứng lại các cuộc tấn công tương tự đã từng xảy ra. Tuy nhiên, các IoC không có cùng mức độ quan trọng (Level), một số loại quan trọng hơn nhiều so với các loại khác. Do sự khác biệt giữa các chỉ số đã dẫn đến nhu cầu về một hệ thống phân loại.

Một hệ thống phân loại IoC nổi tiếng, **The Pyramid of Pain** đã được giới thiệu vào năm 2013 bởi chuyên gia an ninh mạng David J Bianco. Bianco minh họa giá trị của mỗi loại chỉ số trong kim tự tháp này. Anh ấy đặt tên cho kim tự tháp là The Pyramid of Pain do mỗi cấp độ tương ứng với cảm giác “đau khổ” mà cả các chuyên gia an ninh cũng như kẻ thù (attacker) cảm thấy.

![Mitre 02]({{site.url}}/assets/img/2024/04/25/2-mitre.png)

Khi chúng ta leo lên các cấp độ cao hơn của kim tự tháp, việc thu thập và áp dụng các chỉ số trở nên ngày càng khó khăn hơn (đau đớn) đối với các chuyên gia an ninh (security professional). Tuy nhiên, cũng trở nên khó khăn hơn cho cả đối thủ (attacker) trong việc thay thế chúng bằng những thông tin khác (để ẩn danh/che dâu). Ví dụ, theo quan điểm của một security professional, việc thu thập và tích hợp các giá trị băm của các tệp tin độc hại vào các công cụ an ninh là dễ dàng, nhưng không dễ dàng để định nghĩa và áp dụng TTPs (Tactic, Technique, and Procedures) vào các công cụ an ninh. Từ góc nhìn của attacker, việc thay đổi giá trị băm của một tệp độc hại rất đơn giản, tuy nhiên việc thay đổi TTPs lại rất khó thực hiện. Theo quan điểm của người phòng thủ và kẻ tấn công, mỗi loại chỉ số được nêu trong bảng sau:

![Mitre 03]({{site.url}}/assets/img/2024/04/25/3-mitre.png)

Bắt đầu từ phần mềm Antivirus thế hệ đầu tiên, chúng ta phát hiện các IOCs dựa trên các thông tin truyền thống như hash values, IP addresses, và domain names. Tuy nhiên, tại thời điểm hiện tại tôi nghĩ chúng ta nên phải bắt đầu phát hiện dựa trên hành vi của attacker <=> Các Tactic, Technique, Procedures (TTPs) và các công cụ được sử dụng bởi Attacker. Ít nhất, chúng ta phải phát hiện các dấu vết (artifact) của họ trong mạng và máy chủ.

# **2. The MITRE ATT&CK Framework**

MITRE ATT&CK Framework  mô tả và tổ chức một cách có hệ thống các TTPs (Tactic, Technique, and Procedures). Đây là một public knowledge base (Dạng cơ sở dữ liệu mở công khai) có dễ dàng tiếp cận và được đóng góp bởi cộng đồng. Nó đã trở thành một ngôn ngữ chung giữa các security teams để mô tả các TTPs.

![Mitre 04]({{site.url}}/assets/img/2024/04/25/4-mitre.png)

Cần hiểu rằng ATT&CK framework bao gồm hành vi của các (nhóm) attacker đã quan sát đuọc (đã biết). Vì vậy không thể mong đợi nó sẽ bao gồm mọi hành vi của Attacker.

## **2.1. Giới thiệu ATT&CK Matrix**

Truy cập các matrix ở đây: [https://attack.mitre.org/](https://attack.mitre.org/)

Có 3 loại ATT&CK Matrix bao gồm:

- Enterprise: Đối tượng là môi trường Enterprise (Tổ chức thông thường)
- Mobile: Đối tượng là môi trường Mobile
- Industrial Control Systems (ICS): Đối tượng là các hệ thống điều khiển công nghiệp

![Mitre 05]({{site.url}}/assets/img/2024/04/25/5-mitre.png)

Nói chung tùy môi trường mạng đang sử dụng mà ta sử dụng các matrix tương ứng. Ở đây ta thường hay làm với Enterprise nên sẽ tập trung vào Enterprise Matrix, phần sau cũng sẽ chỉ tập trung vào nội dung này.

![Mitre 06]({{site.url}}/assets/img/2024/04/25/6-mitre.png)

Trong ma trận này, mỗi cột đại diện cho một **Tactic** ~ các mục tiêu kỹ thuật của đối thủ. Để đạt được những mục tiêu này kẻ thù sử dụng các **Technique** khác nhau. Nói đến đây nếu mà chưa hiểu hoặc chưa nhớ **Tactic, Technique** là gì cũng không sao. Chỉ cần nhớ các cột trong Matrix là các **Tactic**, mỗi ô là một **Technique.**

MITRE ATT&CK không phải là tĩnh và nó liên tục được cập nhật (đương nhiên rồi). Ví dụ, trong phiên bản 11.3 được phát hành vào tháng 4 năm 2022, 2 **Technique** mới và 10 **sub**-**Technique**  mới đã được thêm vào Enterprise Matrix.

Từ đầu hay nói về Tactic, Technique, và Procedures trong MITRE ATT&CK, các khái niệm này gọi là các đối tượng (object) và ngoài TTP ra thì còn nhiều các đối tượng khác. Các đối tượng này cũng có  mối quan hệ với nhau và được mô tả đầy đủ ở hình sau 

*Chú ý ta cần biết các khái niệm này là gì thì mới khai thác được MITRE ATT&CK nên cái hình này và đoạn sau từ đây trở đi thì cần phải nhớ!*

![Mitre 07]({{site.url}}/assets/img/2024/04/25/7-mitre.png)

## **2.2. Tactics (TA)**

*Tiếng việt có thể hiểu Tactics là các chiến thuật :) nhưng thôi từ giờ cứ dùng tiếng anh đi vì dịch sang tiếng việt có thể không sát nghĩa cho lắm*

Các Tactics trả lời câu hỏi sau: "Những mục tiêu nào mà attacker đang cố gắng đạt được?". 

Ví dụ: Để thực hiện việc Execution một tệp tin độc hại trên máy tính, attacker đã sử dụng một phần mềm độc hại (malware) được gửi qua email để xâm nhập vào một hệ thống máy tính trong mạng nội bộ của một công ty. Khi tệp đính kèm được mở, phần mềm độc hại đã thực thi và mở một command-line interface (CLI) trên máy tính nạn nhân ⇒ Việc mong muốn Execution một tệp tin độc hại trên máy tính chính là Tactic và command-line interface (CLI) là một kỹ thuật ~ Techinique để thực hiện việc này.

- Hiện tại với môi trường Enterprise có khoảng 14 Tactics như sau

![Mitre 08]({{site.url}}/assets/img/2024/04/25/8-mitre.png)

- ATT&CK framework không được thiết kế để đọc các tactic theo cách tuyến tính (linear ), và attacker không cần phải tiến qua các Tactics từ trái sang phải. Attacker cũng không nhất thiết phải sử dụng tất cả các chiến thuật ATT&CK.
- Trên trang chủ của Mitre ATT&CK khi click vô  link chi tiết của một Tactic. Ta sẽ thấy thông tin meta data của một Tactic như sau:

![Mitre 09]({{site.url}}/assets/img/2024/04/25/9-mitre.png)

*Dễ thấy mỗi Tactic có một ID. Ví dụ, ID của chiến thuật Credential Access là TA0006. MITRE ATT&CK cũng cung cấp một mô tả ngắn gọn cho mỗi chiến thuật. Và thông tin ngày sửa đổi.*

- Trong mỗi Tactic sẽ chứa một danh sách **Technique** đã từng được sử dụng bởi các Attacker (hoặc Attacker Groupo) trong các cuộc tấn mạng thực tế (threat actor).

![Mitre 10]({{site.url}}/assets/img/2024/04/25/10-mitre.png)

## 2.3. **Technique** (T)

Các **Technique (kỹ thuật)** mô tả "cách thức" mà một attacker hoàn thành một mục tiêu Tactic thông qua một hành động (action ) hoặc một loạt các hành động (series of actions). Ví dụ, một kẻ tấn công có thể sử dụng **Technique** OS Credential Dumping để đạt được Tactic Credential Access. Vì vậy, một **Technique** là một hành vi cụ thể của attacker được sử dụng để đạt được một mục tiêu.

Dĩ nhiên các Attacker có thể sử dụng một hoặc nhiều Technique kết hợp để đạt được mục tiêu của họ.

Một Technique cũng có thể được sử dụng để đạt được nhiều mục tiêu. Vì vậy, một Technique có thể được phân loại dưới nhiều Tactic. Ví dụ, Technique T1078 Valid Accounts được phân loại trong bốn Tactic sau: Defense Evasion, Persistence, Privilege Escalation, Initial Access.

MITRE ATT&CK cung cấp thông tin sau về mỗi Technique

- Metadata
- Description
- Sub-techniques
- Procedure examples
- Mitigations
- Detections

Đoạn sau sẽ mô tả chi tiết về từng nội dung này.

### **2.3.1. Metadata**

Đây là mục Metadata của OS Credential Dumping technique

![Mitre 11]({{site.url}}/assets/img/2024/04/25/11-mitre.png)

Một vài trường thường có trong meta data bao gồm:

- ID: ID nhận diện của Technique. **Thường bắt đầu bằng T**.
- Sub-techniques: Danh sách ID của Technichque con (Trong trường hợp là technique cha)
- Sub-technique of: Link sang technique cha (Trong trường hợp là technique con)
- Platforms: Nền tảng có thể sử dụng Technichque
- Permission Required: Quyền yêu cầu để thực hiện technique
- Data Sources: Nơi có thể thu thập IoC để phát hiện việc sử dụng kỹ thuật
- Defense Bypassed: Quy thuật bypass tránh bị phát hiện
- Impact Type: Impact trong t/h kỹ thuật được sử dụng. VD: ảnh hưởng đến **availability** hay **integrity**
- CAPEC ID: ID định danh duy nhất của một mẫu tấn công trong CAPEC. Về CAPEC là một dự án khác của Mitre mô tả về các lỗi cách khai thác và khắc phục theo từng ngôn ngữ. Xem chi tiết [tại đây]([https://cwe.mitre.org/index.html](https://cwe.mitre.org/index.html))

*Một số trường mở rộng khác như: Contributors, Version, Created, Last Modified*

### **2.3.2. Description**

Luôn có mô tả ở phần đầu của Technique

![Mitre 12]({{site.url}}/assets/img/2024/04/25/12-mitre.png)

### **2.3.3. Sub-techniques**

Một technique cũng có thể có các sub-technique

![Mitre 13]({{site.url}}/assets/img/2024/04/25/13-mitre.png)

### **2.3.4. Procedure Examples (G/S)**

Phần **Procedure Examples** mô tả chi tiết về kỹ thuật hoặc cũng có thể là các thông tin liên quan "không phải" kỹ thuật liên quan tới Technique. 

Ví dụ: 'APT1 đã được biết đến sử dụng kỹ thuật lấy thông tin đăng nhập bằng cách sử dụng Mimikatz.' Vì vậy, các Procedure Examples có thể hỗ trợ trong việc xác định threat actor (Attacker Group) nào thực hiện kỹ thuật, cách nó áp dụng, và công cụ nào nó sử dụng. Với các ký hiệu ID bắt đầu bằng G ~ Attacker Group; S ~ Công cụ/phần mềm sử dụng. Sẽ nói chi tiết về Group và Software tiếp ở phần sau.

![Mitre 14]({{site.url}}/assets/img/2024/04/25/14-mitre.png)

Thông tin này có thể có giá trị để tái tạo một sự cố bằng cách mô phỏng hành vi của kẻ tấn công, cũng như cụ thể về cách phát hiện trường hợp khi nó được sử dụng. Tuy nhiên, cá nhân thấy trong hầu hết các Procedure được cung cấp trong Technique thường là chung chung, khó để thực hành mô phỏng hành vi của kẻ tấn công, lúc này ta sẽ tìm tới sự trợ giúp của GG thần trưởng ^^

### **2.3.5. Mitigation (M)**

Phần này mô tả phương án giảm thiểu (Hardening) để hạn chế hệ thống bị khai thác bằng Technique này.

![Mitre 15]({{site.url}}/assets/img/2024/04/25/15-mitre.png)

### **2.3.6. Detection (DS)**

Cung cấp các thông tin về phương án phát hiện (Detect) Technique này. Hãy chú ý vào phần Source ~ nguồn mà ta có thể lấy thông tin Detect

![Mitre 16]({{site.url}}/assets/img/2024/04/25/16-mitre.png)

# **3. Groups (G)**

Attacker thường hoạt động theo tổ chức do vậy được phân nhóm (dựa trên một số các dấu hiệu riêng biệt VD: Technique, Signature trong Software (Mailware/Tools) sử dụng, ...) và được theo dõi bằng một tên chung trong cộng đồng an ninh mạng. Hiện tại, có 133 nhóm trong Framework MITRE ATT&CK

![Mitre 17]({{site.url}}/assets/img/2024/04/25/17-mitre.png)

Với từng Group cũng liệt kê Meta data của Group. Các Technique, Software hay sử dụng bởi group.

# **4. Software**

Software được phân loại thành phần malware và Tool trong Framework MITRE ATT&CK. 

Cũng như các đối tượng khác. Software cũng có các thông tin Metadata và các Technique được sử dụng kết hợp với Software.

Đồng thời cung cấp đường đẫn tới trang chủ hoặc link download Software

![Mitre 18]({{site.url}}/assets/img/2024/04/25/18-mitre.png)

# **5. Khác MITRE ATT&CK**

- Mitre còn  một dự án khác khá hay ho là CAPEC  [https://cwe.mitre.org/index.html](https://cwe.mitre.org/index.html) (CAPEC là viết tắt của "Common Attack Pattern Enumeration and Classification" (Danh sách và Phân loại Các Mẫu Tấn Công Phổ Biến). Đây là một cơ sở dữ liệu chứa các mô hình tấn công phổ biến được phân loại và mô tả. CAPEC cung cấp một phương tiện để hiểu các mô hình tấn công, giúp người dùng nắm bắt cách thức mà kẻ tấn công có thể tận dụng lỗ hổng hoặc điểm yếu để thực hiện các cuộc tấn công. Được phát triển bởi MITRE Corporation, CAPEC cung cấp một khung nhìn tổng thể về cách các mối đe dọa hoạt động và là một công cụ hữu ích cho các chuyên gia bảo mật trong việc phân tích và phòng ngừa các cuộc tấn công mạng.) đáng để tham khảo.
- Nagivator là một công cụ hỗ trợ đắc lực trong việc tra cứu MITRE ATT&CK: [https://mitre-attack.github.io/attack-navigator/](https://mitre-attack.github.io/attack-navigator/)

# **6. Tham khảo**

- 1. [https://www.picussecurity.com/mitre-attack-framework-beginners-guide](https://www.picussecurity.com/mitre-attack-framework-beginners-guide)