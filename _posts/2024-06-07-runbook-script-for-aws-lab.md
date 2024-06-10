---
layout: post
title: "Runbook để dựng môi trường Lab dành cho AWS,"
author: "vietcf"
categories: fun
tags: ['AWS']
image: assets/img/2024/06/07/cloudformation_intro.png
---

Lab trên AWS đôi khi cần dựng một số thành phần Infra cơ bản như Network, ECS, EKS, ... Để việc này nhanh chóng hơn khi khởi tạo và tiện dọn dẹp khi lab xong (không bị quyên hủy thiếu resource khiến AWS chạc tiền nhầm) tôi hay sử dụng CloudFormation. Hôm nay tổng hợp lại một số CloudFormation template hữu dụng:

## 1. Dựng VPC với Public Subnet và Private Subnet ở 2 AZ trong cùng một Region

### 1. Mô hình dự kiến:

![Simple network]({{site.url}}/assets/img/2024/06/07/0_runbook_aws_network_simple.png)

### 2.CloudFormation Template File: 

[Github Link](https://github.com/vietcf/aws-lab-template/tree/main/0.infras)

## 2. Dựng cụm EKS với một node Minimun 

### 1. Mô hình dự kiến:

![Eks]({{site.url}}/assets/img/2024/06/07/aws_eks_simple.png)

### 2. CloudFormation Template File: 

[Github Link](https://github.com/vietcf/aws-lab-template/tree/main/1.eks)

