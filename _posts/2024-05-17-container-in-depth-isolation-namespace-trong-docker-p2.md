---
layout: post
title: "Container in depth - Cô lập tiến trình, tính năng Namespace trên linux - điểm khởi nguồn của container, P2"
author: "vietcf"
categories: job
tags: ['Container']
image: assets/img/2024/05/17/0-intro-isolation-container.png
---

Khi sử dụng Container, đặc biệt khi attach một console vào ta cảm giác như container giống y hệt các máy ảo độc lập chạy công nghệ ảo hóa truyền thống. Độc lập với nghĩa là một Container hầu như chúng không thể nhìn, can thiệp vào các Container khác cũng như máy chủ host. Trong bài viết này, chúng ta sẽ tìm hiểu cách công nghệ Container thực hiện việc này. 


