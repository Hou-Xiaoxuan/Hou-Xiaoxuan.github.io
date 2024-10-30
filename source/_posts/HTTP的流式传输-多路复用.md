---
title: HTTP的流式传输&多路复用
tags: 网络, 八股
date: 2024-10-31 01:47:27
---


## 背景

最近为了面试又得重新复习一遍八股知识。这其中就有已经不止看过多少遍的，不同版本的 `HTTP` 协议之间的区别。这次为了有一个更好的理解，决定搭配一些实验性质的代码，动手来理解 HTTP 的一些特性。两个常用特性：“流式传输”与“多路复用”。

“流式传输”和“多路复用”其实是两个不太相关的概念。

而多路复用指多个数据流通过同一个通道传输。在 HTTP 的场景中，意味着可以在同一个请求中同时传输多个请求和响应。

>问：你了解不同版本HTTP的特性区别吗？
>
>答：
>
>HTTP的标准主要有三个，1.0(96)、1.1(97)、2.0(15) 和仍在实验中的 3.0。现行应用最广的是1.1标准，部分场景下有服务开始使用2.0版本。
>
>- 1.0 vs 1.1
>HTTP/1.0 和 HTTP/1.1 都是基于纯文本的。
>1. 新增了部分状态码
>2. 优化了缓存策略（静态文件）
>3. 新增了资源范围请求（静态文件）、压缩支持、100状态码
>4. 请求体中新增了HOST字段
>5. 默认支持长连接，复用TCP连接来执行多个请求。
>
>- 1.1 vs 2.0
>HTTP/2 不再是文本的形式，而是使用二进制帧传输。
>1. 头部压缩，减小传输数据的大小。可以在C/S之间维护使用过的头部的索引，使用索引代替实际头部。
>2. 服务器推送，服务器可以在收到请求时主动推动客户端还没有请求的资源（如绑定的css/js文件等）
>3. 多路复用，实现了全双工的通信，解决了长连接的”对头阻塞“问题。

## HTTP/1.1 中的流式传输：`Transfer-Encoding: chunked`

“流式传输”指数据可以是一边生成一遍处理的，而不需要等待所有数据都获取到再一次新传输。在 HTTP 的场景中，如果不能提前知道传输数据的大小，设置对应的`Content-Length`，就可以使用 HTTP/1.1 中的 `Transfer-Encoding: chunked`特性进行“流式传输”。细节可以参考 [HTTP 协议中的 Transfer-Encoding —— JerryQu](https://imququ.com/post/transfer-encoding-header-in-http.html) 一文。

HTTP 的流式传输依赖长连接的特性。流式传输可以发生在客户端向服务端上传数据和服务端返回响应数据。在同一个请求中，服务端可以同时以流式传输处理客户端的请求，同时返回响应数据，达到一种“交互式”的效果。

使用 GO 的 `net/http` 库实现一个简单的测试 API 。在不指明的情况下，HTTP server 的版本是 HTTP/1.1。

```go
func chunkHandler(w http.ResponseWriter, r *http.Request) {
	println("Request: ", r.Method)
	if r.Method == "GET" {
		for i := 0; i < 3; i++ {
			fmt.Fprintf(w, "Chunk %d", i)
			w.(http.Flusher).Flush()
			time.Sleep(1 * time.Second)
		}
	} else if r.Method == "POST" {
		for {
			buf := make([]byte, 1024)
			n, err := r.Body.Read(buf)
			if err != nil {
				if err == io.EOF {
					fmt.Println("Read complete")
					break
				}
				fmt.Println("error: ", err)
				break
			}
			if n > 0 {
				println("Read: ", string(buf))
				w.Write([]byte("read one chunk\n"))
				// w.(http.Flusher).Flush() // flush will cause `http: invalid Read on closed Body`
			}
		}
	}
}
func main() {
	http.HandleFunc("/chunk", chunkHandler)
}
```

使用 telnet 作为客户端发送 HTTP 请求：

```bash
> telent 127.0.0.1 8080
 telnet localhost 8080
Trying ::1...
Connected to localhost.
Escape character is '^]'.

GET /chunk HTTP/1.1
Host: localhost

HTTP/1.1 200 OK
Date: Fri, 18 Oct 2024 16:26:56 GMT
Content-Type: text/plain; charset=utf-8
Transfer-Encoding: chunked

8
Chunk 0
8
Chunk 1
8
Chunk 2
0

```

在服务端的代码中，接收到 GET 请求时会分三次写入数据，并调用`w.(http.Flusher).Flush()`立刻发送数据。在这种情况下，`net/http`自动将请求的格式转为`chunked`并分批返回数据。telnet 并未对返回的数据做特殊处理，可以直接看到编码先发送了数据的长度，然后发送了数据内容，最后以0和两次换行符标识请求结束。

由于 **HTTP/1.1 默认支持长连接**，因此可以在 telnet 的终端下继续执行第二次请求，测试 POST 以`chunked`的格式分批上传数据。

```bash
POST /chunk HTTP/1.1
HOST: localhost
Transfer-Encoding: chunked

5
send1
5
send2
0

HTTP/1.1 200 OK
Date: Fri, 18 Oct 2024 16:35:41 GMT
Content-Length: 30
Content-Type: text/plain; charset=utf-8

read one chunk
read one chunk
```

在请求数据发送结束后，telnet 收到了服务端返回的结果。可以看到每次以 chunk 的编码发送数据后服务端立刻收到并处理了请求，并在读取全部请求数据后返回了响应。

### 交互式响应

HTTP的响应理所当然也是支持 `chunked`编码的，不需要等待请求处理完才返回。但是由于各种问题（详见文章末尾）没能使用 Go 成功实现交互式的 API，交互式采用 Rust 编写，逻辑与 `chunkHandler` 的 POST 部分类似。

```rust
use axum::{
    body::Body, extract::Request, http::StatusCode, response::Response, routing::post, Router,
};
use tokio_stream::{wrappers::ReceiverStream, StreamExt};
#[tokio::main]
async fn main() {
    let app = Router::new().route("/chunk-echo", post(echo));
    let listener = tokio::net::TcpListener::bind("0.0.0.0:8000").await.unwrap();
    axum::serve(listener, app).await.unwrap();
}
// async echo handler, read plain text and response with chunked encoding concurrently
async fn echo(req: Request<Body>) -> Result<Response<Body>, StatusCode> {
    let mut body_stream = req.into_body().into_data_stream();
    let (tx, rx) = tokio::sync::mpsc::channel::<Result<bytes::Bytes, axum::Error>>(1);
    tokio::spawn(async move {
        while let Some(chunk) = body_stream.next().await {
            let chunk = chunk.unwrap();
            let mut resp_chunk = bytes::BytesMut::from(format!("receive {} bytes: ", chunk.len()).as_str());
            resp_chunk.extend_from_slice(&chunk);
            tx.send(Ok(resp_chunk.into())).await.unwrap();
            println!("send chunk: {:?}", chunk);
        }
        tx.send(Ok(bytes::Bytes::from("finish request"))).await.unwrap();
        println!("end of body");
    });
    let body = ReceiverStream::new(rx);
    // create a response with the request body
    Ok(Response::builder()
        .header("Content-Type", "application/octet-stream")
        .header("Transfer-Encoding", "chunked")
        .body(Body::from_stream(body))
        .unwrap())
}
```

使用 Telnet 进行相同测试后，结果如下。可以看到，这次在请求头发送完毕后，立即收到了响应头。并且，每次发送数据后，服务端都及时回复，实现了交互式通信。

```bash
> telnet localhost 8000
Trying ::1...
telnet: connect to address ::1: Connection refused
Trying 127.0.0.1...
Connected to localhost.
Escape character is '^]'.

POST /chunk-echo HTTP/1.1
Host: localhost
Content-Type: text/plain
Transfer-Encoding: chunked

HTTP/1.1 200 OK
content-type: application/octet-stream
transfer-encoding: chunked
date: Fri, 18 Oct 2024 16:56:57 GMT

5
send1
16
receive 5 bytes: send1
0

E
finish request
0
```

## HTTP/2 基于流和帧的数据传输

在HTTP/2中，不再需要使用 `chunked` 传输编码，因为 HTTP/2 的传输是基于流和帧的。每个HTTP消息都会被分割成多个帧，并通过流传输。HTTP/2 的帧结构天然支持数据的分段传输，所有数据都是以二进制帧的形式传输，且帧大小可以动态控制。

```bash
# 数据帧的格式
+-------------------------------------+
|               帧长度（3字节）         |
+-------------------------------------+
|    帧类型（1字节） | 标志（1字节）      |
+-------------------------------------+
|     保留位（1位） | 流ID（31位）       |
+-------------------------------------+
|              负载数据（可变长度）      ｜
+-------------------------------------+
```

帧的类型主要有 DATA 帧，HEADERS 帧和一些控制类的帧，如终止流、调整优先级等。而 HTTP/1 中的请求行则处理为“伪首部“，与头部一起发送。

```bash
# HTTP/1
POST /index.html HTTP/1.1 # 请求行
Host: example.com         # 请求头
# HTTP/2（实际为二进制格式而不是文本）
:method: POST
:scheme: https
:authority: example.com
:path: /index.html
```

**服务端代码** 

可以用下面的代码来看一下帧传输的效果。大部分情况下 HTTP/2 都与 TLS 加密式绑定的，主流浏览器也不支持访问使用 HTTP/2 访问非 HTTPS 的服务器。但是 HTTP/2 有无加密的版本 H2C（HTTP/2 over Cleartext），可以简化用于试验的代码。ÏÏÏ

```go
func main() {
	http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
		fmt.Fprintln(w, "Hello h2c")
		w.(http.Flusher).Flush()
		fmt.Fprintln(w, "Your IP is ", r.RemoteAddr)
		w.(http.Flusher).Flush()
		time.Sleep(2 * time.Second)
		fmt.Fprintln(w, "Bye h2c")
	})
	s := &http.Server{
		Addr:    ":8080",
		Handler: h2c.NewHandler(http.DefaultServeMux, &http2.Server{}),
	}
	log.Fatal(s.ListenAndServe())
}
```

**客户端代码** 

直接使用 TCP 连接来访问上面的服务器，来查看分帧的效果。其中，由于 HTTP/2 的头部压缩是必须的，为了简化代码，使用`net/http2/hpack`来完成头部的压缩。在连接开始前，需要发送特定格式的前言和 SETTINGS 帧，这一帧的 ID 必须为0，剩下的 ID只要满足自增就可以了。

```go
func main() {
	// 连接到本地HTTP/2服务器
	conn, err := net.Dial("tcp", "127.0.0.1:8080")
	if err != nil {
		log.Fatalf("连接服务器失败: %v", err)
	}
	defer conn.Close()

	// 发送 HTTP/2 连接前言
	conn.Write([]byte("PRI * HTTP/2.0\r\n\r\nSM\r\n\r\n"))
	// 发送 SETTINGS 帧，type=0x04, flags=0x00, streamID=0x00
	settingsFrame := []byte{0x00, 0x00, 0x00, 0x04, 0x00, 0x00, 0x00, 0x00, 0x00}
	conn.Write(settingsFrame)
	readResponse(conn) // 读取 settings 帧响应

	// 构造 HEADERS 帧
	headers := []hpack.HeaderField{
		{Name: ":method", Value: "GET"},
		{Name: ":scheme", Value: "http"},
		{Name: ":path", Value: "/"},
		{Name: ":authority", Value: "127.0.0.1"},
	}
	var headerBuf bytes.Buffer
	encoder := hpack.NewEncoder(&headerBuf) // 按照 HPACK 格式压缩头部
	for _, hf := range headers {
		encoder.WriteField(hf)
	}
	headerBlock := headerBuf.Bytes()
	// 创建并发送 HEADERS 帧
	headersFrame := make([]byte, 9+len(headerBlock))
	headersFrame[0], headersFrame[1], headersFrame[2] = byte(len(headerBlock)>>16), byte(len(headerBlock)>>8), byte(len(headerBlock))
	headersFrame[3], headersFrame[4], headersFrame[8] = 0x01, 0x05, 0x01 // HEADERS 帧类型和流 ID。客户端流 ID 为奇数
	copy(headersFrame[9:], headerBlock)
	conn.Write(headersFrame)
	readResponse(conn) // 读取请求的流响应
}
```

每个流有自己的 ID，为了避免混淆，客户端请求的 ID 都为奇数，服务端的流（服务端推送）的 ID 都为偶数。如果出现不匹配的情况，服务端会报告`PROTOCOL_ERROR`。每个流以 END flag 表示流的结束。简单的解析逻辑代码如下。

```go
func readResponse(conn net.Conn) {
	resp := make([]byte, 1024)
	for {
		n, err := conn.Read(resp)
		if err != nil {
			return // 连接关闭
		}
		offset := 0
		for offset < n {
			if n-offset < 9 {
				break // 等待更多数据
			}
			// 解析帧头
			frameLen := int(resp[offset])<<16 | int(resp[offset+1])<<8 | int(resp[offset+2])
			frameType, flags := resp[offset+3], resp[offset+4]
			streamID := int(resp[offset+5])<<24 | int(resp[offset+6])<<16 | int(resp[offset+7])<<8 | int(resp[offset+8])
			fmt.Printf("帧类型: %d, 长度: %d, 流ID: %d\n", frameType, frameLen, streamID)

			if n-offset < 9+frameLen {
				break
			}
			// 读取帧负载并打印
			payload := resp[offset+9 : offset+9+frameLen]
			if frameType == 0x00 {
				fmt.Printf("数据帧内容: %s\n", payload)
			} else {
				fmt.Printf("帧内容: %v\n", payload)
			}
			// 检查END_STREAM标志位表示流结束
			if flags&0x1 != 0 {
				fmt.Println("流结束")
				return
			}
			offset += 9 + frameLen
		}
	}
}
```

# 多路复用

*未完待续*


---

## *与实验无关的部分，可以跳过* 

- Go 未能实现 HTTP/1.1 交互式的 API 的原因

Go 的 `http/net` 存在一些问题。以上面处理 POST 请求的代码为例。HTTP 协议允许一边流式处理请求的数据一边流式返回响应数据。但是在读取部分数据的情况下调用`w.(http.Flusher).Flush()`会阻塞，直到客户端发送完全部的数据。并且之后再调用`r.Body.Read()`会直接返回错误“error:  http: invalid Read on closed Body"。问题的原因细节暂时不清楚。

`ResponseWriter.Write`的文档的警告 HTTP/1.1 版本的服务端必须在调用`Write`方法前读取全部的 body 数据：

> // go version: 1.23.2
>
> Depending on the HTTP protocol version and the client, calling Write or WriteHeader may prevent future reads on the Request.Body. For HTTP/1.x requests, handlers should read any needed request body data before writing the response. Once the  headers have been flushed (due to either an explicit Flusher.Flush call or writing enough data to trigger a flush), the request body may be unavailable.

然而根据 [Issue 15527](https://github.com/golang/go/issues/15527) ，这一行为已经被改变了：

> This has been implemented, documented and released in go1.21, so I guess this issue can be closed.

根据实际测试单纯的调用`Write`不会导致 body 无法读取，但是并不能调用 `Flush`来返回部分数据。 ~~也很可能是代码写得有问题~~ 

**更新**：根据 [issue 61889](https://github.com/golang/go/issues/61889)，go 的 net/http 模块在启用 POST 的 body stream 时遇到了bug，后来支持被回退了。虽然 issue 的讨论场景是 wasm，但是可能对其他场景也有类似的影响吧。

