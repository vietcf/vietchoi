---
layout: post
title: "Sai lầm chết người khi không để ý truyền vào hàm python một tham chiếu (Reference),"
categories: job
tags: ['Python']
image: assets/img/2024/05/20/1-reference-intro.jpg
---

Mất mấy ngày để debug mới tìm ra nguyên nhân chỉ vì sơ ý trong việc trong quá trình xử lý truyền vào hàm (Function) Python một tham chiếu (Reference) :)


## Quay lại cơ bản,

Ai đã lập trình chắc không lạ gì về tham chiếu (Reference)

![Reference]({{site.url}}/assets/img/2024/05/20/2-reference.png)


## Mô tả lại lỗi ngờ nghệch,

Có một mảng là một array lưu kết quả lấy từ hàm functionGetAll()

```
array_result_all = functionGetAll() #Hàm functionGetAll trả lại một array là toàn bộ kết quả có thể sau đó nhét vào một array tên là array_result_all
```

Kết quả từ mảng array_result_all sau đó được truyền vào các functionResultX nào đó để thực hiện một việc gì đó rồi trả lại một mảng resultX tương ứng. Và ý muốn của tôi là qua mỗi lần xử lý bởi functionResultX vẫn không làm thay đổi bất cứ thông tin gì trong array_result_all sau mỗi lần gọi functionResultX.  

```
result1 = functionResult1(array_result_all)
result2 = functionResult2(array_result_all)
result3 = functionResult3(array_result_all)
result4 = functionResult4(array_result_all)
```

Sau đó gom lại các kết quả đã xử lý bằng một result_all

```
result_all = result1 + result2 + result3 + result4
```
Nhưng khi in in ra result_all kết quả lại hoàn toàn không như mong mốn. Quay lại debug tôi nhận ra rằng việc sử dụng ít hay nhiều functionResultX  nó lại làm ảnh hưởng tới kết quả In ra :). Ví dụ nếu chạy functionResult1 => functionResult2 thì giá trị của result3 lại khác khi chỉ chạy functionResult3 mà **bỏ qua functionResult1 và functionResult2**.

Sau một thời gian tìm kiếm tôi nhận ra rằng trong functionResult<N> tôi lại thản nhiên loop và sửa lại phần tử của array_result_all dẫn đến Dữ liệu gốc của array_result_all sau mỗi khi thực hiện functionResult<N> lại bị thay đổi làm cho các functionResult<N+1> sau đó nhận vào một mảng array_result_all không còn **"Nguyên vẹn"** như ban đầu => Sai lệch kết quả mong muốn.


## Phương án xử lý,

Trong functionResult<N> khi không muốn làm thay đổi dữ liệu gốc nằm trong array_result_all cần tạo bản clone bằng copy.deepCopy(). Sau đó mới sửa đổi trên bản clone để không làm sai lệch dữ liệu tham chiếu truyền vào ^^.

Đúng là một trải nghiệm "Ngờ nghệch" thực sự,

## Kinh nghiệm rút ra

>Phải cận thận với dấu = trong python và lập trình,

