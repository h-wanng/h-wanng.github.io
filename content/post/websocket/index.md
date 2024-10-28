+++
author = "H. Wang"
title = "Node.js websocket服务器"
date = "2024-09-19"
description = "node.js简单实现websocket服务器"
tags = [
    "Node.js", "JS", "Websocket"
]
categories = [
    "JS"
]

image = ""
+++

## 1. 导入依赖模块

```js
import express from 'express';
import { createServer } from 'http';
import { WebSocket, WebSocketServer } from 'ws';
import { fileURLToPath } from 'url';
import { join, dirname } from 'path';
```

- express：用于创建 HTTP 服务器和处理 HTTP 请求，特别是提供静态文件。
- createServer：从 http 模块导入，用于创建一个基本的 HTTP 服务器，它可以与 WebSocket 服务器一起运行。
- WebSocket 和 WebSocketServer：用于创建 WebSocket 服务器，处理双向实时通信。
- fileURLToPath 和 dirname：这些是处理 ES 模块中的 __dirname 的方法，通常 __dirname 只在 CommonJS 模块系统中可用，这里使用这两个方法来模拟获取当前文件路径。

## 2. 获取 __dirname

```js
const __filename = fileURLToPath(import.meta.url);
const __dirname = dirname(__filename);
```

- 在 ES 模块（import 语法）中，无法直接使用 __dirname。所以通过 fileURLToPath 将 import.meta.url 转换为文件路径，然后用 dirname 获取当前文件所在的目录路径。

## 3. 创建 Express 应用和 HTTP 服务器

```js
const app = express();
const server = createServer(app);
```
- app 是使用 express() 创建的一个 Express 应用，它用于处理 HTTP 请求和路由。
- server 是通过 createServer 创建的 HTTP 服务器，它使用 app 作为处理请求的回调。这个服务器不仅可以处理 HTTP 请求，还可以与 WebSocket 服务器集成。

## 4. 设置 WebSocket 服务器

```js
const wss = new WebSocketServer({ server });
```
- wss 是一个 WebSocket 服务器，创建时将 HTTP 服务器作为参数传入。这样，HTTP 服务器既可以处理普通的 HTTP 请求，也可以处理 WebSocket 连接。

## 5. 处理 WebSocket 连接

```js
wss.on('connection', (ws) => {
    // 设定心跳检测时间间隔
    const interval = setInterval(() => {
        if (ws.readyState === WebSocket.OPEN) {
            ws.ping();  // 发送 ping 消息，检测客户端是否存活
        }
    }, 30000);  // 每30秒发送一次心跳检测

    ws.on('pong', () => {
        console.log('received pong from client');
    });
  
    ws.on('message', (message) => {
    	console.log('ws message')
        const data = JSON.parse(message);
    });

    ws.on('close', () => {
      clearInterval(interval);  // 当连接关闭时清理定时器
    	console.log('ws close');
    });
});
```
- wss.on('connection', ...)：当一个客户端通过 WebSocket 连接到服务器时触发，ws 是与该客户端的 WebSocket 连接对象。
- ws.ping()：定期向客户端发送 ping 消息，客户端会自动响应 pong 消息，这可以确保连接的活跃状态。
- ws.on('pong', ...)：处理客户端返回的 pong 消息，证明连接仍然活跃。WebSocket 协议规定，当客户端接收到 ping 帧时，会自动回复一个 pong 帧。
- ws.on('message', ...)：当服务器收到来自客户端的消息时触发。这里的消息被假设为 JSON 格式，所以用 JSON.parse 进行解析。
- ws.on('close', ...)：当客户端关闭 WebSocket 连接时触发，记录连接关闭的日志。

## 6. 提供静态文件

```js
app.use(express.static(join(__dirname, '../public/dist')));
```
- app.use：设置 Express 静态文件中间件，用于提供静态文件，如 HTML、CSS、JS 文件。
  join(__dirname, '../public/dist')：将当前文件的目录与 ../public/dist 拼接，表示静态文件位于服务器的 public/dist 目录中。

## 7. 启动服务器

```js
const PORT = 3000;
server.listen(PORT, () => {
    console.log(`Server is listening on http://localhost:${PORT}`);
});
```
- server.listen(PORT, ...)：启动 HTTP 服务器并监听 3000 端口。当服务器启动后，会输出 Server is listening on http://localhost:3000。

## 8. 完整代码

```js
import express from 'express';
import { createServer } from 'http';
import { WebSocket, WebSocketServer } from 'ws';
import { fileURLToPath } from 'url';
import { join, dirname } from 'path';

// 获取 __dirname
const __filename = fileURLToPath(import.meta.url);
const __dirname = dirname(__filename);

// 创建 Express 应用和 HTTP 服务器
const app = express();
const server = createServer(app);

// 设置 WebSocket 服务器
const wss = new WebSocketServer({ server });

wss.on('connection', (ws) => {
    // 设定心跳检测时间间隔
    const interval = setInterval(() => {
        if (ws.readyState === WebSocket.OPEN) {
            ws.ping();  // 发送 ping 消息，检测客户端是否存活
        }
    }, 30000);  // 每30秒发送一次心跳检测

    ws.on('pong', () => {
        console.log('received pong from client');
    });
  
    ws.on('message', (message) => {
    	console.log('ws message')
        const data = JSON.parse(message);
    });

    ws.on('close', () => {
      clearInterval(interval);  // 当连接关闭时清理定时器
    	console.log('ws close');
    });
});

// 配置 Express 处理静态文件
app.use(express.static(join(__dirname, '../public/dist')));

// 启动服务器
const PORT = 3000;
server.listen(PORT, () => {
    console.log(`Server is listening on http://localhost:${PORT}`);
});
```

