---
layout: post
title: "Kubernetes In Depth  - Giới thiệu về Kubernetes API server, P1"
author: "vietcf"
categories: job
tags: ['Active Directory']
image: assets/img/2024/04/22/0-intro-kubernetes.png
---

Bữa đang xoạn cái tiêu chuẩn thiết lập an toàn thông tin cho Kubernetes/EKS, mà làm việc này thì không thể không "depth" về cơ chế Authen&Author trong EKS. Nhân tiện tìm hiểu về nội dung này ghi chép lại đây các điểm chính yếu để sau này có lúc càn thì đọc lại.

# 1. Các thành phần của cụm Kubernetes

Trước khi bắt đầu cần hiểu qua về các thành phần trong cụm Kubernetes (Thủy tổ của EKS/AKS). Nội dung này rất nhiều chỗ nói đi nói lại rồi, chỉ xin phép post lại cái hình và ghi chú sơ qua về các thành phần.

![Kubernetes Component]({{site.url}}/assets/img/2024/04/22/1-kubernetes-component.png)

Chúng ta chia ra Control Plane phần nằm trên Controller Node và các Data Plane nằm trên các Worker Node. Khái niệm về Control Plane và Data Plane là khái niệm rất quen thuộc được dùng rất nhiều trong các mô hình của các hệ thống IT hiện tại. Trong đó phần Control Plane là phần thực hiện việc Orchestration ~ Tiếng việt có thể hiểu với ý nghĩa là điều khiển, điều phối nhiều Data Plane thực hiện các các công việc, đảm bảo việc phối hợp giữa các Data Plane trở nên nhịp nhàng. 

* Control Plane có các thành phần
    * kube-api-server: Là thành phần trung tâm của cụm Kubernetes. Cung cấp các HTTP API để End User và các thành phần khác trong và ngoài cụm Kubernetes giao tiếp với cụm
    * kube-scheduler: Là thành phần quản lý lập lịch, chịu trách nhiệm xác định Node phù hợp để triển khai các Pod
    * kube-controler-manager: Là thành phần quản lý & điều khiển, chịu trách nhiệm quản lý các controller và điều khiển các tài nguyên trong cụm
    * etcd: Là hệ thống lưu trữ phân tán được sử dụng để lưu trữ cấu hình của cụm Kubernetes

* Data Plane có các thành phần
    * kubelet: Hoạt động như một kênh truyền thông giữa máy chủ API và node. Đồng thời chịu trách nhiệm quản lý và điều khiển các container trên Node
    * kube-proxy: Là một agent trên Node trong hệ thống Kubernetes. Nó được sử dụng để quản lý các dịch vụ mạng và cung cấp khả năng định tuyến cho các yêu cầu truy cập đến các Pod trên Node

* Với trường hợp Kubernetes chạy trên các nền tảng Cloud như EKS, AKS còn có thêm thành phần nữa gọi là cloud-controller-manager (CCM)). Thành phần này làm nhiệm vụ kết nối và quản lý tài nguyên đám mây trong một cụm Kubernetes, giúp tự động hóa việc triển khai và vận hành các ứng dụng trên hạ tầng đám mây (Cloud).

# 2. Kuberetes API server

Kubernetes API là một HTTP RESTful API. Được Expose trên Control Node chạy service kube-api-server. HTTP API này được End User và các thành phần khác trong hoặc ngoài cụm Kubernetes giao tiếp với cụm. 

![Kubernetes Authen Author]({{site.url}}/assets/img/2024/04/22/1-Kubernetes-authen-author-01.png)

API này hỗ trợ việc duyệt (retrieve), tạo (create), sửa (update) và xóa (delete) các kubernetes resource thông qua chuẩn HTTP verbs (POST, PUT, PATCH, DELETE, GET). Dĩ nhiên để thực hiện các việc này cần thông qua các bước Xác thực và kiểm tra phân quyền.

## Thử gọi API của Kubernetes

Có nhiều cách để gọi API của kubernetes có thể tham khảo tại [Link này](https://kubernetes.io/docs/tasks/administer-cluster/access-cluster-api/). Xem qua thì đơn giản nhất là sử dụng ```kubectl proxy```.

* Bước 1:  là ```kubectl proxy --port=8080 &``` để kubectl tự binding một port 8080 ở local.
* Bước 2: Chuyển qua terminal khác và thử dùng curl truy cập vào api này ```curl http://localhost:8080/api/``` thì ra kết quả như sau:


