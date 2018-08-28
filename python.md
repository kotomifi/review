# python



## 类变量和实例变量

```python
class Person():
    name = "jack"
    nicname = []

p1 = Person()
p2 = Person()
p1.name = "liekkas"

print p1.name       # liekkas
print p2.name       # jack
print Person.name   # jace

p1.nicname.append('lina')
print p1.nickname          # ['lina']
print p2.nickname          # ['lina']
print Person.nickname      # ['lina']
```

## cProfile

性能分析工具

## 内存管理

- 引用计数器，用 sys.getrefcount() 可以查看引用次数；
- 引用为 0， 会被 gc 回收；
- Python只会在特定条件下，自动启动垃圾回收。当Python运行时，会记录其中分配对象(object allocation)和取消分配对象(object deallocation)的次数。当两者的差值高于某个阈值时，垃圾回收才会启动。
- gc.get_threshold() 查看
- 分代回收，Python将所有的对象分为0，1，2三代。所有的新建对象都是0代对象。当某一代对象经历过垃圾回收，依然存活，那么它就被归入下一代对象。
- 循环引用： 为了回收这样的引用环，Python复制每个对象的引用计数，可以记为gc_ref。假设，每个对象i，该计数为gc_ref_i。Python会遍历所有的对象i。对于每个对象i引用的对象j，将相应的gc_ref_j减1。 在结束遍历后，gc_ref不为0的对象，和这些对象引用的对象，以及继续更下游引用的对象，需要被保留。而其它的对象则被垃圾回收。

Python引用了一个内存池(memory pool)机制，即Pymalloc机制(malloc:n.分配内存)，用于对小块内存的申请和释放管理
当创建大量消耗小内存的对象时，频繁调用new/malloc会导致大量的内存碎片，致使效率降低。内存池的概念就是预先在内存中申请一定数量的，大小相等的内存块留作备用，当有新的内存需求时，就先从内存池中分配内存给这个需求，不够了之后再申请新的内存。这样做最显著的优势就是能够减少内存碎片，提升效率。

python中的内存管理机制都有两套实现

- 一套是针对小对象，就是大小小于256bits时，pymalloc会在内存池中申请内存空间；
- 当大于256bits，则会直接执行 new/malloc 的行为来申请新的内存空间。

### 引用减少

- 重新赋值
- del

### 深浅拷贝

# I/O 复用

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
