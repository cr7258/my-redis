## 6.3 创建异步任务

### 接收 socket

现在运行我们的简单服务器 :

```bash
cargo run
```

此时服务器会处于循环等待以接收连接的状态，接下来在一个新的终端窗口中启动上一章节中的 redis 客户端，由于相关代码已经放入 examples 文件夹下，因此我们可以使用 `--example` 来指定运行该客户端示例:

```bash
cargo run --example hello-redis
```

此时，客户端的输出是: `Error: "unimplemented"`, 同时服务器端打印出了客户端发来的由 redis 命令和数据组成的数据帧: `GOT: Array([Bulk(b"set"), Bulk(b"hello"), Bulk(b"world")])`。

### 使用 HashMap 存储数据

使用 `cargo run` 运行服务器，然后再打开另一个终端窗口，运行 hello-redis 客户端示例: `cargo run --example hello-redis`。

Bingo，在看了这么多原理后，我们终于迈出了小小的第一步，并获取到了存在 HashMap 中的值: `从服务器端获取到结果=Some(b"world")`。

**但是问题又来了：这些值无法在 TCP 连接中共享，如果另外一个用户连接上来并试图同时获取 hello 这个 key，他将一无所获。**

例如我另外打开一个终端窗口，使用 redis-cli 直接去获取 hello 这个 key，将返回 nil。

```bash
redis-cli
127.0.0.1:6379> get hello
(nil)
```

## 6.4 共享状态

使用 `cargo run` 运行服务器，然后再打开另一个终端窗口，运行 hello-redis 客户端示例: `cargo run --example hello-redis`。

这次，在另一个终端窗口中再次使用 redis-cli 去获取 hello 这个 key，将返回正确的结果。

```bash
redis-cli
127.0.0.1:6379> get hello
"world"
```

## 6.5 消息传递

首先，将之前实现的 `src/main.rs` 文件中的服务器端代码放入到一个 `bin` 文件中，等下可以直接通过该文件来运行我们的服务器:

```bash
mkdir src/bin
mv src/main.rs src/bin/server.rs
```

接着创建一个新的 `bin` 文件，用于包含我们即将实现的客户端代码:

```bash
touch src/bin/client.rs
```

完成本章的代码后启动 redis 服务器。

```bash
cargo run --bin server
```

可以先用 redis 客户端往服务器中写入一条数据。

```bash
redis-cli
127.0.0.1:6379> set hello world
OK
```

然后启动 redis 客户端，客户端有两个发送者 tx 和 tx2，分别发送 get 和 set 任务。有个 manager 任务，负责接收这些任务并将其发送到服务器端。两个发送任务中分别传递 oneshot 的 resp_tx 发送端，manager 任务在和 redis 服务器完成交互后，会将结果返回给这些 resp_rx。 


```bash
cargo run --bin client

GOT = Ok(Ok(Some(b"world"))) # get 返回获取的值
GOT = Ok(Ok(())) # set 返回 OK
```