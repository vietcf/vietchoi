---
layout: post
title: "Prototypal inheritance model và kế thừa trong javascript,"
categories: job
tags: ['javascript']
image: assets/img/2024/10/05/prototype-inheritance-model-intro.jpg
---

Javascript là một ngôn ngữ thuần hướng đối tượng (OOP). Tức là mọi kiểu dữ liệu của Javascript (`number`, `string`, và `boolean`) bản chất đều là các đối tượng.

Chúng ta đều biết kế thừa **(Inheritance)**  là đặc điểm nổi  bật của các loại ngôn ngữ hướng đối tượng (OOP). Với đặc điểm này thì các Object con sẽ có khả năng truy cập thuộc tính (Attribute) và sử dụng phương thức (Method) của Object cha.

JavaScript sử dụng mô hình **Prototypal inheritance**, khá khác biệt so với mô hình **Class-based** mà nhiều ngôn ngữ OOP khác sử dụng. Trong bài viết này, tôi sẽ làm rõ về cách thức hoạt động của mô hình **Prototypal inheritance** trong Javascript ~ Tập trung vào nội dung là kế thừa (Inheritance) trong Javascript.

# Một chút khởi đầu

- Tôi sẽ bắt đầu bằng việc tạo một object trong javascript bằng đoạn code quen thuộc sau bằng cách dùng Object Literals:

```jsx
const user =  {
    username: "wiener",
    userId: 01234,
    greet: function() {
        return "Hello, my name is " + this.name;
    }
}
```

Thử các cách access attribute (có 2 cách) và call method

![image.png]({{site.url}}/assets/img/2024/10/05/image.png)

- Ngoài ra cũng có thể sử dụng Constructor Function để khởi tạo mẫu cho đối tượng rồi sau đó tạo đối tượng từ Constructor Function này *(Hình dung **"như"** Class ở các ngôn ngữ khác nhưng không phải là Class vì  luôn nhớ rằng Javascript nó là **Prototypal inheritance**)*

```jsx
function Person(_username, _userId){
  this.username= _username;
  this.userId= _userId;
  this.greet = function() {
    return "Hello, my name is " + this.name;
  }
}

let person1 = new Person("wiener", 1234); 
console.log(person1.username);
console.log(person1['username']);
console.log(person1.greet());
```

Tương tự  thử các cách access attribute (có 2 cách) và call method

![image.png]({{site.url}}/assets/img/2024/10/05/image1.png)

- Ngoài hai cách tạo Object Javascript trên đối với ES6 cũng có thể sử dụng cú pháp Class quen thuộc

```jsx
class Person {
    constructor(username, userId) {
        this.username = username;
        this.userId = userId;
    }

    greet() {
        return "Hello, my name is " + this.username;
    }
}

let person1 = new Person("wiener", 1234);
console.log(person1.username); // Output: wiener
console.log(person1['username']); // Output: wiener
console.log(person1.greet()); // Output: Hello, my name is wiener
```

![image.png]({{site.url}}/assets/img/2024/10/05/image2.png)

> **Tuy nhiên dù tạo Object theo cách nào thì Inheritance của Javascript vẫn là Prototypal Inheritance đơn giản nó đặc tính riêng của Javascript**

# Prototypal Inheritance hoạt động thế nào?

- Với mô hình **Prototypal inheritance** (gọi tắt là Prototype) thì các Object trong Javascript luôn luôn được kế thừa các tính năng từ một **Prototype object**

![image.png]({{site.url}}/assets/img/2024/10/05/image3.png)

- Việc thừa kế này được thực hiện bằng cách mỗi Object trong javascript luôn có một thuộc tính ẩn nội bộ (**internal property**) gọi là `[[Prototype]]` trỏ tới một Prototype Object. *(Chú ý tránh nhầm lẫn  Prototype Object với thuộc tính prototype của Function mô tả bên dưới)*

![image.png]({{site.url}}/assets/img/2024/10/05/image4.png)

- Các Object bất kỳ được khi được tạo ra sẽ  tự động kế thừa tất cả các thuộc tính (Attribute), phương thức (Method) của *Prototype Object* , trừ khi chúng đã có thuộc tính riêng với cùng tên.  Điều này nghĩa là mỗi khi bạn tham chiếu đến một thuộc tính/phương thức của một đối tượng, engine JavaScript đầu tiên sẽ cố gắng truy cập thuộc tính/phương thức đó trực tiếp trên đối tượng hiện tại. Nếu đối tượng hiện tại không có thuộc tính/phương thức phù hợp lúc đó engine JavaScript sẽ tìm kiếm nó trên *Prototype Object* của đối tượng. Việc này cho phép các lập trình viên tạo ra các Object mới có thể tái sử dụng các thuộc tính và phương thức của các đối tượng hiện có.

![image.png]({{site.url}}/assets/img/2024/10/05/image5.png)

![image.png]({{site.url}}/assets/img/2024/10/05/image6.png)

- Có 2 cách truy cập Prototype Object là dùng `<object_name>.__proto__`  hoặc `Object.getPrototypeOf/Object.setPrototypeOf(<object_name>)`. Trong đó `__proto__`  là một getter/setter cho `[[Prototype]]` nhưng đã lỗi thời, JavaScript hiện đại khuyến khích lập trình viên nên sử dụng hàm `Object.getPrototypeOf/Object.setPrototypeOf` để thay thế. Tuy nhiên do khả năng tương thích ngược giữa các version của Javascript ta vẫn có thể sử dụng `obj .__ proto__` để truy cập vào `[[Prototype]]`.

![image.png]({{site.url}}/assets/img/2024/10/05/image7.png)

# The prototype chain

Mô tả ngắn gọn thì Prototype nó có dạng chuỗi nhiều lớp:

```jsx
A —inheritance—> B (Object Prototype)  —inheritance—> C (Object Prototype) … —inheritance—> Object —inheritance—> null 
```

Mô tả bằng lời cho mối liên hệ trên cụ thể như sau: Prototype Object của một Object bất kỳ là một Object khác và đương nhiên bản thân Prototype Object đó cũng sẽ có Prototype Object của riêng nó, tiếp tục cứ thế cho đến cuối cùng của chuỗi là Prototype Object ở cấp cao nhất sẽ có Prototype Object trỏ về **null**. Để dễ hình dung về điều này hãy coi hình vẽ sau:

![image.png]({{site.url}}/assets/img/2024/10/05/image8.png)

Đương nhiên trong chuỗi trên thì A hoàn toàn có khả năng truy cập các Attribute và call Method của B, C, ..., Object ~ Object bất kỳ có thể truy cập các Attribute hay call Method các Prototype Object ở level cao hơn.

Ví dụ:  Có đoạn string `a=”Hello”` có Prototype Object là **String** và **String** có Prototype Object là **Object** ⇒ A cũng sẽ được thừa kế các tính năng từ **Object**.

![image.png]({{site.url}}/assets/img/2024/10/05/image9.png)

# Một số nội dung liên quan tới Prototype

- Mặc định, đối với các kiểu dữ liệu cơ bản Number, String, Array JavaScript tự động gán cho Object mới một trong những built-in Prototype thích hợp sẵn có của nó. Ví dụ, các chuỗi (String) tự động được gán cho `String.prototype.`

```jsx
let myObject = {};
Object.getPrototypeOf(myObject);    // Object.prototype

let myString = "";
Object.getPrototypeOf(myString);    // String.prototype

let myArray = [];
Object.getPrototypeOf(myArray);	    // Array.prototype

let myNumber = 1;
Object.getPrototypeOf(myNumber);    // Number.prototype
```

![image.png]({{site.url}}/assets/img/2024/10/05/image10.png)

- Trong Javascript, một hàm (Function) cũng được coi là một Object (Điều này càng khẳng định trong javascript mọi thứ đều là Object). Và hàm có một thuộc tính đặc biệt tên là **prototype.** Bản thân thuộc tính prototype này mang giá trị là một Object.

![image.png]({{site.url}}/assets/img/2024/10/05/image11.png)

Để ý hơn một chút thì thấy thuộc tính prototype này luôn là kiểu Object và có một hàm tạo (constructor) trỏ ngược lại Function luôn.

- Nếu ta dùng hàm (Function) để tạo ra 1 mẫu khởi tạo đối tượng, thì bạn có thể thêm được các thuộc tính hoặc phương thức vào thuộc tính prototype của hàm khởi tạo để thực hiện kế thừa. Tất cả các đối tượng con tạo ra bởi hàm khởi tạo đều mang các giá trị trong thuộc tính prototype của hàm này.

![image.png]({{site.url}}/assets/img/2024/10/05/image12.png)

Đó là các nội dung liên quan tới Prototypal Inheritance trong Javascript, không quá khó hiểu lắm phải không.
