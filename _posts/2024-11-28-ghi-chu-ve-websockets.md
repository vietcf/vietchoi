---
layout: post
title: "Ghi chú về Webockets,"
categories: job
tags: ['javascript']
image: assets/img/2024/11/28/web-socket-intro.png
---

# Giới thiệu về Web Socket

HTTP là giao thức 1 chiều (Haft duplex) client gửi request và server nhận  rồi phản hồi lại response luôn. Trong HTTP có thể nhiều request → response cùng nằm trong một TCP connection tuy nhiên dù thế nào thì server vẫn không thể chủ động liên hệ để gửi (send) dữ liệu với client được,  nó chỉ nằm đó nghe ngóng đợi client send request sau đó nó mới có thể phản hồi lại mà thôi. Nói chung khá là “bị động”.

Trong thực tế nhu cầu giao tiếp hai chiều vẫn luôn thực sự cần thiết trong nhiều tình huống. Ví dụ: Trong các ứng dụng chat khi người dùng chat lên cửa sổ sẽ là chiều từ CLIENT gửi lên SERVER, ngược lại mỗi khi có tin nhắn mới gửi tới cho người dùng thì SERVER thực sự cần nhu cầu “chủ động” gửi lại tin nhắn đó cho CLIENT.

Để giải quyết vấn đề này nhóm các giao thức giao tiếp 2 chiều socket ra đời, trong đó trên nền tảng web hay được sử dụng có WebSockets.  Websockets là giao thức 2 chiều (Full duplex), sau khi đã khởi tạo kết nối thì cả 2 bên đều có khả năng **chủ động** (send/receive) data với đối phương.

![image.png]({{site.url}}/assets/img/2024/11/28/image.png)

WebSockets được khởi tạo phiên giữa CLIENT và SERVER bằng HTTP, sau khi khởi tạo xong các message qua lại giữa 2 bên bằng một định dạng riêng không phải HTTP nữa gọi là **WebSocket frames** .  So với TCP sockets thì để sử dụng Websockets đơn giản hơn rất nhiều vì điều kiện để nó hoạt động là máy tính chỉ cần có browser là có thể thực hiện được rồi.

Web socket là asynchronously (Async) nghĩa là Client và Server có thể gửi dữ liệu cho nhau bất cứ lúc nào. Không cần phải chờ yêu cầu-phản hồi như trong HTTP.

Viết khá dài nhưng tới đây chỉ cần nhớ: 

> **Websockets ↔ Full duplex & Async**
> 

Sau khi tạo xong kết nối websocket các message gửi nhận giữa CLIENT ↔ Server có thể là dạng text hoặc blob (binary). Thông thường ta hay gặp dạng text với format json. Nói tới đây để hình dung rằng bản chất Websockets có thể hình dung nó đóng ai trò tạo kênh kết nối còn dữ liệu truyền gì và theo format như thế nào do ta hoàn toàn quyết định.

Không quá khó để sử dụng một đoạn code nodejs và js nhỏ để tạo ra một Web Socket:

Code socket bằng nodejs chạy trên server

 

```jsx
const WebSocket = require('ws');
const readline = require('readline');

// Create WebSocket server on port 8080
const wss = new WebSocket.Server({ port: 8080 });

// Set up readline interface to accept console input
const rl = readline.createInterface({
    input: process.stdin,
    output: process.stdout
});

wss.on('connection', (ws) => {
    console.log('New client connected');

    // Send a welcome message to the client
    ws.send('Welcome to the WebSocket server!');

    // Handle incoming messages from clients
    ws.on('message', (message) => {
        console.log(`Received from client: ${message}`);
    });

    // When a client disconnects, log the event
    ws.on('close', () => {
        console.log('Client disconnected');
    });
});

// Prompt the server for input and send messages to all connected clients
rl.on('line', (input) => {
    console.log(`Server sending message to all clients: ${input}`);
    
    // Broadcast the message to all connected clients
    wss.clients.forEach((client) => {
        if (client.readyState === WebSocket.OPEN) {
            client.send(`Server says: ${input}`);
        }
    });
});

console.log('WebSocket server is running on ws://localhost:8080');
console.log('Type a message in the console to send it to all connected clients...');

```

Code Client Socket bằng js chạy trên browser

```jsx
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>WebSocket Client</title>
</head>
<body>
    <h1>WebSocket Client</h1>
    <p>Enter a message to send to the WebSocket server:</p>
    
    <!-- Input form for user message -->
    <form id="messageForm">
        <input type="text" id="messageInput" placeholder="Type your message..." required />
        <button type="submit">Send</button>
    </form>

    <h2>Messages:</h2>
    <ul id="messagesList"></ul>

    <script>
        // Connect to the WebSocket server
        const ws = new WebSocket('ws://localhost:8080');

        // Reference to form and input field
        const messageForm = document.getElementById('messageForm');
        const messageInput = document.getElementById('messageInput');
        const messagesList = document.getElementById('messagesList');

        // When the connection is open
        ws.onopen = () => {
            console.log('Connected to WebSocket server');
        };

        // Handle incoming messages from the server
        ws.onmessage = (event) => {
            const li = document.createElement('li');
            li.textContent = `Server: ${event.data}`;
            messagesList.appendChild(li);
        };

        // Send a message to the server when the form is submitted
        messageForm.addEventListener('submit', (event) => {
            event.preventDefault(); // Prevent form submission
            const message = messageInput.value;
            if (message && ws.readyState === WebSocket.OPEN) {
                ws.send(message); // Send the message to the server
                console.log('Sent to server:', message);
                
                // Display the message in the list of messages
                const li = document.createElement('li');
                li.textContent = `Client: ${message}`;
                messagesList.appendChild(li);
                
                // Clear the input field
                messageInput.value = '';
            }
        });

        // Handle WebSocket errors
        ws.onerror = (error) => {
            console.error('WebSocket error:', error);
        };

        // Handle WebSocket connection closure
        ws.onclose = () => {
            console.log('Disconnected from WebSocket server');
        };
    </script>
</body>
</html>

```

Thử chạy đoạn code trên xem ta có gì nào?

![image.png]({{site.url}}/assets/img/2024/11/28/image1.png)

- Từ client ta có hể gõ **hello server** → Trên console nhận được **hello server.** Từ server console có thể gõ **hello client** → Trên browser có thể nhận được hello client mà không cần phải refresh trình duyệt ⇒ Giao tiếp là 2 chiều, bên nào cũng có thể “chủ động” gửi nhận dữ liệu.
- Mỗi khi restart lại server process thì client bị dis kết nối phải f5 lại browser để khởi tạo lại kết nối socket ⇒ Rõ ràng là phiên kết nối được khởi tạo bằng một cặp HTTP request - response.

# Khám phá Websockets với Burpsuite

Không có cách nào để hiểu về web socket hơn là tận tay vào khám phá nó trong thực tế, để chuẩn bị tôi sử dụng Burpsuite - Công cụ quen thuộc của giới Pentester.

## Intercept Websockets

Công cụ Burpsuite hoàn toàn hỗ trợ Intercept webockets traffic.  Với websocket tính năng Proxy ta sẽ có 2 Tab **HTTP history** và **Websockets history**. Tab HTTP History chỉ lưu lại kết nối khởi tạo (handshark) Websockets còn tab Websocket history lưu lại message gửi nhận qua socket.

![image.png]({{site.url}}/assets/img/2024/11/28/image2.png)

![image.png]({{site.url}}/assets/img/2024/11/28/image3.png)

Cặp Request ↔ Response để Handshark có các header theo chuẩn như trên hình. Và sau khi đã handshark xong kết nối, hoàn toàn không có cặp Request ↔ Response nào nữa ⇒ Minh chứng cho việc Websockets chỉ sử dụng http duy nhất để khởi tạo kết nối mà thôi. Đoạn truyền nhận dữ liệu sau này hoàn toàn không base trên HTTP.

![image.png]({{site.url}}/assets/img/2024/11/28/image4.png)

## Repeat Websockets

Burp hỗ trợ Repeater với websocket như bình thường. Có thể lựa chọn gửi cho CLIENT hay SERVER.

![image.png]({{site.url}}/assets/img/2024/11/28/image5.png)

Nhấn vào hình cái bút chì của Repeater Burp còn cho nhiều lựa chọn nữa.

![image.png]({{site.url}}/assets/img/2024/11/28/image6.png)

# Khai thác lỗ hổng của Websockets

### Nguyên tắc chung trước khi bắt đầu

Như ở trên có nói, bản chất Websockets cũng chỉ là một hình thức thiết lập sau đó tạo ra một kênh kết nối. Các message text gửi nhận là gì, format thế nào do ta quyết định mà thôi. Thường thì người ta hay dùng JSON, còn nếu thích thì bạn có thể tự define format của mình thôi. Chính bởi vậy mà trên nền Websockets attacker có thể thực hiện các kiểu tấn công khác:

- User input từ CLIENT truyền tới SERVER nếu server xử lý unsafe có thể dẫn tới các kiểu tấn công server site: SQL injection; XXE; SSRF, …
- CLIENT nhận dữ liệu từ  SERVER  qua Websockets nên nếu dữ liệu Websockets sử lý unsafe sau đó lại truyền tới CLIENT thì CLIENT có thể bị các kiểu tấn công Client Side như XSS; CSRF; …
- Chú ý rằng một vài blind vulnerabilities thông qua Websockets chỉ có thể phát hiện sử dụng kỹ thuật out-of-band (OAST)

![image.png]({{site.url}}/assets/img/2024/11/28/image7.png)

# **Làm thế nào để sử dụng WebSocket một cách an toàn**

Có một vài điểm cần chú ý để sử dụng Websockets một cách an toàn:

- Sử dụng giao thức mã hóa wss:// (WebSockets qua TLS) để đảm bảo dữ liệu được mã hóa.
- Fix cứng URL WebSockets endpoint, và chắc chắn rằng không do người dùng kiểm soát URL này.
- Bảo vệ handshake message của Websockes khỏi các cuộc tấn công CSRF, để tránh việc bị cross-site WebSockets hijacking.
- Cần phải hiểu rằng dữ liệu gửi/nhận qua WebSocket coi như là không đáng tin cậy ở cả hai chiều. Luôn kiểm tra dữ liệu ở cả phía server và client đảm bảo an toàn (safely)