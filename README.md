## 6.3 创建异步任务

### 接收 socket

https://course.rs/advance-practice/spawning.html#%E6%8E%A5%E6%94%B6-sockets

现在运行我们的简单服务器 :

```bash
cargo run
```

此时服务器会处于循环等待以接收连接的状态，接下来在一个新的终端窗口中启动上一章节中的 redis 客户端，由于相关代码已经放入 examples 文件夹下，因此我们可以使用 `--example` 来指定运行该客户端示例:

```bash
cargo run --example hello-redis
```

此时，客户端的输出是: `Error: "unimplemented"`, 同时服务器端打印出了客户端发来的由 redis 命令和数据组成的数据帧: `GOT: Array([Bulk(b"set"), Bulk(b"hello"), Bulk(b"world")])`。
