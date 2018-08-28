## I/O 复用

Unix 5 种 I/O 模型：

### 阻塞式 I/O

等待数据返回；

### 非阻塞式 I/O

反复调用 recvfrom 等待返回，消耗 CPU 过大；

### I/O 复用

同阻塞式 I/O， 但是可以监控多个文件描述符；

**select**

监控 可读，可写，异常 I/O, 或者超时， 

- 优点： 几乎所有平台支持
- 缺点： 存在最大文件描述符限制，linux 一般为 1024

**poll**

pollfd结构包含了要监视的event和发生的event，不再使用select“参数-值”传递的方式。
同时，pollfd并没有最大数量限制（但是数量过大后性能也是会下降）。
和select函数一样，poll返回后，需要轮询pollfd来获取就绪的描述符。

**epoll**


### 信号驱动式 I/O

### 异步 I/O


# redis
## 数据类型

- string
- hash
- list
- set
- sorted set

## 安全

需要设置密码，防止暴力破解；

## 主从复制

- master 可以拥有多个 slave；
- 多个 slave 可以连接同一个 master，还可以连接到其它 slave；
- 主从复制不会阻塞 master, 在同步数据时 master 可以继续处理 client 请求；
- 提高系统的伸缩性；

## 事物

- 原子性
- 一致性

## 持久化

## 发布及订阅

## pipline

## 虚拟内存
