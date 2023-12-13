### 原理
1. 启动服务器后，服务器中会常驻监听socket服务
2. 客户端发送连接请求，监听socket监听到，创建业务socket
3. 客户端请求socket与服务器对应业务socket配对
***注释：配对原理如下：***
客户端ip cip
客户端port cport
服务端ip sip
服务端port sport

a、客户端发送请求，请求头包括：cip:cport sip:sport

b、监听socket(sip:sport)监听到请求，创建业务socket sip:sport cip:cport

c、客户端socket与服务器业务socket配对

### 代码实现如下
#### node 服务端
```javascript
// 加载node上websocket模块 ws;
var ws = require("ws");

// 启动基于websocket的服务器,监听我们的客户端接入进来。
var server = new ws.Server({
	host: "127.0.0.1",
	port: 6080,
});

// 监听接入进来的客户端事件
function websocket_add_listener(client_sock) {
	// close事件
	client_sock.on("close", function() {
		console.log("client close");
	});

	// error事件
	client_sock.on("error", function(err) {
		console.log("client error", err);
	});
	// end 

	// message 事件, data已经是根据websocket协议解码开来的原始数据；
	// websocket底层有数据包的封包协议，所以，绝对不会出现粘包的情况。
	// 每解一个数据包，就会触发一个message事件;
	// 不会出现粘包的情况，send一次，就会把send的数据独立封包。
	// 想我们如果是直接基于TCP，我们要自己实现类是于websocket封包协议；
	client_sock.on("message", function(data) {
		console.log(data);
		client_sock.send(JSON.stringify({a: 1,b :2}));
		// client_sock.close();
	});
	// end 
}

// connection 事件, 有客户端接入进来;
function on_server_client_comming (client_sock) {
	console.log("client comming");
	websocket_add_listener(client_sock);
}

server.on("connection", on_server_client_comming);

// error事件,表示的我们监听错误;
function on_server_listen_error(err) {
	console.log(err)
}
server.on("error", on_server_listen_error);

// headers事件, 回给客户端的字符。
function on_server_headers(data) {
	console.log(data);
}
server.on("headers", on_server_headers);

```

#### node 客户端
```javascript
var ws = require("ws");

// url ws://127.0.0.1:6080
// 创建了一个客户端的socket,然后让这个客户端去连接服务器的socket
var sock = new ws("ws://127.0.0.1:6080");
sock.on("open", function () {
	console.log("connect success !!!!");
	sock.send("HelloWorld1");
	// setInterval(function() {
	// 	sock.send("Hellow")
	// }, 3000);
	sock.send("HelloWorld2");
	sock.send("HelloWorld3");
	sock.send("HelloWorld4");
	sock.send(Buffer.alloc(10));
});

sock.on("error", function(err) {
	console.log("error: ", err);
});

sock.on("close", function() {
	console.log("close");
});

sock.on("message", function(data) {
	console.log(JSON.parse(data));
});

```

#### h5 客户端
```javascript
var ws = require("ws");

// url ws://127.0.0.1:6080
// 创建了一个客户端的socket,然后让这个客户端去连接服务器的socket
var sock = new ws("ws://127.0.0.1:6080");
sock.on("open", function () {
	console.log("connect success !!!!");
	sock.send("HelloWorld1");
	// setInterval(function() {
	// 	sock.send("Hellow")
	// }, 3000);
	sock.send("HelloWorld2");
	sock.send("HelloWorld3");
	sock.send("HelloWorld4");
	sock.send(Buffer.alloc(10));
});

sock.on("error", function(err) {
	console.log("error: ", err);
});

sock.on("close", function() {
	console.log("close");
});

sock.on("message", function(data) {
	console.log(JSON.parse(data));
});

```