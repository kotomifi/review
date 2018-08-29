# 操作系统

进程，线程，协程区别

- 进程
    - 系统资源分配和调度
    - 独立内存空间
    - 进程间通信来通信
    - 切换开销大（栈，寄存器，虚拟内存，文件句柄）
 - 线程
    - cpu 调度的基本单位
    - 独立运行的基本单位
    - 只拥有少量资源（pc， 寄存器， 栈）
    - 线程通信主要通过共享内存
 - 协程
    - 用户轻量级线程
    - 用户调度
    - 有寄存器上下文和栈
    - 无需加锁，速度快
    
 tcp 三次握手，四次挥手
 
 ![](https://coolshell.cn/wp-content/uploads/2014/05/tcp_open_close.jpg)
 
 三次握手
 
 1. client 发送 SYN, seq=x; (SYN_SENT, connect())
 2. server 回复 SYN, ack=x+1, seq=Y (SYN_RCVD)
 3. client 回复 ack=y+x (ESTABLISHED)
 
 四次挥手
 
 1. client 发送 FIN seq=x+2 ACK=y+1 (FIN_WAIT_1)
 2. server 回复 ACK x+3 (CLOSE_WAIT)
 3. server 发送 FIN seq=y+1 (FIN_WAIT_2)
 4. client 回复 ACK=y+2 (TIME_WAIT)

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
## metaclass

类也是对象，可以赋值给变量，可以拷贝，可以增加属性，可以作为函数参数， 用元类创建类， type 创建， 元类主要在类工厂。

如果在类中使用 __metaclass__， 则用 __metaclass__ 创建类本身。

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

select 监控 可读，可写，异常 I/O, 或者超时，调用 select 函数后会阻塞，只到有描述符就绪或超时。
函数返回后，通过遍历 fdset 来i找到就绪的描述符。

- 优点： 几乎所有平台支持
- 缺点
    - 存在最大文件描述符限制，linux 一般为 1024；
    - 对 socket 扫描是线性扫描，采用轮询的方式，fd 比较多时效率低；
    - 需要维护一个用来存放所有 fd 的数据结构，这样会使得用户空间和内核空间在传递该数据结构时复制开销大。

**poll**
poll本质上和select没有区别，它将用户传入的数组拷贝到内核空间，然后查询每个fd对应的设备状态，
如果设备就绪则在设备等待队列中加入一项并继续遍历，如果遍历完所有fd后没有发现就绪设备，
则挂起当前进程，直到设备就绪或者主动超时，被唤醒后它又要再次遍历fd。这个过程经历了多次无谓的遍历。

它没有最大连接数的限制，原因是它是基于链表来存储的，但是同样有一个缺点：

大量的fd的数组被整体复制于用户态和内核地址空间之间，而不管这样的复制是不是有意义。
poll还有一个特点是“水平触发”，如果报告了fd后，没有被处理，那么下次poll时会再次报告该fd。

pollfd结构包含了要监视的event和发生的event，不再使用select“参数-值”传递的方式。
同时，pollfd并没有最大数量限制（但是数量过大后性能也是会下降）。
和select函数一样，poll返回后，需要轮询pollfd来获取就绪的描述符。

**epoll**

epoll使用一个文件描述符管理多个描述符，将用户关系的文件描述符的事件存放到内核的一个事件表中，这样在用户空间和内核空间的copy只需一次。

epoll支持水平触发和边缘触发，最大的特点在于边缘触发，它只告诉进程哪些fd刚刚变为就绪态，并且只会通知一次。还有一个特点是，epoll使用“事件”的就绪通知方式，通过epoll_ctl注册fd，一旦该fd就绪，内核就会采用类似callback的回调机制来激活该fd，epoll_wait便可以收到通知。

优点：
- 没有最大并发连接的限制，能打开的FD的上限远大于1024
- 效率提升，不是轮询的方式，不会随着FD数目的增加效率下降。只有活跃可用的FD才会调用callback函数；即Epoll最大的优点就在于它只管你“活跃”的连接，而跟连接总数无关，因此在实际的网络环境中，Epoll的效率就会远远高于select和poll
- 内存拷贝，利用mmap()文件映射内存加速与内核空间的消息传递；即epoll使用mmap减少复制开销

epoll对文件描述符的操作有两种模式：LT（level trigger）和ET（edge trigger）。LT模式是默认模式，LT模式与ET模式的区别如下：

LT(level triggered)是缺省的工作方式，并且同时支持block和no-block socket。在这种做法中，内核告诉你一个文件描述符是否就绪了，然后你可以对这个就绪的fd进行IO操作。如果你不作任何操作，内核还是会继续通知你的。

ET(edge-triggered)是高速工作方式，只支持no-block socket。在这种模式下，当描述符从未就绪变为就绪时，内核通过epoll告诉你。然后它会假设你知道文件描述符已经就绪，并且不会再为那个文件描述符发送更多的就绪通知，直到你做了某些操作导致那个文件描述符不再为就绪状态了(比如，你在发送，接收或者接收请求，或者发送接收的数据少于一定量时导致了一个EWOULDBLOCK 错误）。但是请注意，如果一直不对这个fd作IO操作(从而导致它再次变成未就绪)，内核不会发送更多的通知(only once)。

ET模式在很大程度上减少了epoll事件被重复触发的次数，因此效率要比LT模式高。epoll工作在ET模式的时候，必须使用非阻塞套接口，以避免由于一个文件句柄的阻塞读/阻塞写操作把处理多个文件描述符的任务饿死。

在select/poll中，进程只有在调用一定的方法后，内核才对所有监视的文件描述符进行扫描，而epoll事先通过epoll_ctl()来注册一个文件描述符，一旦基于某个文件描述符就绪时，内核会采用类似callback的回调机制，迅速激活这个文件描述符，当进程调用epoll_wait()时便得到通知。


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
