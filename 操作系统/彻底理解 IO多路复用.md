---
doc_type: hypothesis-highlights
url: 'https://zhuanlan.zhihu.com/p/150972878'
---

# 彻底理解 IO多路复用

## Metadata
- Author: [zhuanlan.zhihu.com]()
- Title: 彻底理解 IO多路复用
- Reference: https://zhuanlan.zhihu.com/p/150972878
- Category: #article

## Page Notes
## Highlights
- 同步阻塞（BIO）服务端采用单线程，当accept一个请求后，在recv或send调用阻塞时，将无法accept其他请求（必须等上一个请求处recv或send完），无法处理并发 — [Updated on 2023-03-28 20:18:47](https://hyp.is/sMidLs1iEe2yko9N6bAsSQ/zhuanlan.zhihu.com/p/150972878) — Group: #Public

- 服务器端采用多线程，当accept一个请求后，开启线程进行recv，可以完成并发处理，但随着请求数增加需要增加系统线程，大量的线程占用很大的内存空间，并且线程切换会带来很大的开销，10000个线程真正发生读写事件的线程数不会超过20%，每次accept都开一个线程也是一种资源浪费 — [Updated on 2023-03-28 20:18:55](https://hyp.is/taa2AM1iEe28kH9xbG2K0w/zhuanlan.zhihu.com/p/150972878) — Group: #Public

- 同步非阻塞（NIO）服务器端当accept一个请求后，加入fds集合，每次轮询一遍fds集合recv(非阻塞)数据，没有数据则立即返回错误，每次轮询所有fd（包括没有发生读写事件的fd）会很浪费cpu — [Updated on 2023-03-28 20:18:59](https://hyp.is/uDW20s1iEe2oQ1sInFlcjw/zhuanlan.zhihu.com/p/150972878) — Group: #Public

- 每次调用poll，都需要把fd集合从用户态拷贝到内核态，这个开销在fd很多时会很大对socket扫描时是线性扫描，采用轮询的方法，效率较低（高并发时） — [Updated on 2023-03-28 20:20:11](https://hyp.is/4wrprs1iEe2JL6f7AIEuqg/zhuanlan.zhihu.com/p/150972878) — Group: #Public











---
doc_type: hypothesis-highlights
url: 'https://www.yuque.com/tuobaaxiu/inyebp/tk3ywg?'
---

# 推荐一个绝对能提高面试官印象分的知识点！ 已完结 · 语雀

## Metadata
- Author: [yuque.com]()
- Title: 推荐一个绝对能提高面试官印象分的知识点！ 已完结 · 语雀
- Reference: https://www.yuque.com/tuobaaxiu/inyebp/tk3ywg?
- Category: #article

## Page Notes
## Highlights
- 1、select、poll、epoll简单介绍一下这几个关键字：select，poll，epoll都是IO多路复用的机制。I/O多路复用就是通过一种机制，一个进程可以监视多个描述符，一旦某个描述符就绪（一般是读就绪或者写就绪），能够通知程序进行相应的读写操作。但select，poll，epoll本质上都是同步I/O，因为他们都需要在读写事件就绪后，自己负责进行读写，也就是说这个读写过程是阻塞的，而异步I/O则无需自己负责进行读写，异步I/O的实现会负责把数据从内核拷贝到用户空间。 — [Updated on 2023-03-28 19:44:26](https://hyp.is/5CH9bs1dEe2PCZd6JR_Nug/www.yuque.com/tuobaaxiu/inyebp/tk3ywg?) — Group: #Public

- epoll解决select的1，2，3，4不需要轮询，时间复杂度为O(1)epoll_create 创建一个白板 存放fd_eventsepoll_ctl 用于向内核注册新的描述符或者是改变某个文件描述符的状态。已注册的描述符在内核中会被维护在一棵红黑树上epoll_wait 通过回调函数内核会将 I/O 准备好的描述符加入到一个链表中管理，进程调用 epoll_wait() 便可以得到事件完成的描述符 — [Updated on 2023-03-28 19:47:29](https://hyp.is/UU9UkM1eEe298jc625Uo5g/www.yuque.com/tuobaaxiu/inyebp/tk3ywg?) — Group: #Public

- IO复用就是进程预先告诉内核需要监视的IO条件，使得内核一旦发现进程指定的一个或多个IO条件就绪，就通过进程进程处理，从而不会在单个IO上阻塞了。 — [Updated on 2023-03-28 20:05:43](https://hyp.is/3WOu_M1gEe2ZEgfTdZL3fA/www.yuque.com/tuobaaxiu/inyebp/tk3ywg?) — Group: #Public

- fd_set容纳的文件描述符数量由FD_SETSIZE指定，这就限制了select能同时处理的文件描述符最大个数。 — [Updated on 2023-03-28 20:05:55](https://hyp.is/5PirLM1gEe2WHdMQ6XJUZA/www.yuque.com/tuobaaxiu/inyebp/tk3ywg?) — Group: #Public

- select缺点：●每次调用select，都需要把fd集合从用户态拷贝到内核态，这个开销在fd很多时会很大●每次调用select都需要在内核遍历传递进来的所有fd，这个开销在fd很多时也很大●select支持的文件描述符数量太小了，默认是1024 — [Updated on 2023-03-28 20:06:05](https://hyp.is/6oKBTs1gEe2333NP8UHJMg/www.yuque.com/tuobaaxiu/inyebp/tk3ywg?) — Group: #Public

- 操作类型有3种●EPOLL_CTL_ADD：往事件表中注册fd上的事件●EPOLL_CTL_MOD：修改fd上的注册事件●EPOLL_CTL_DEL：删除fd上的注册时间 — [Updated on 2023-03-28 20:06:52](https://hyp.is/BprBXM1hEe2PFiuzVg3ehg/www.yuque.com/tuobaaxiu/inyebp/tk3ywg?) — Group: #Public

- epoll使用一组函数来完成操作，而不是单个函数。其次，epoll把用户关心的文件描述符上的事件放在内核上的一个事件表中，从而无须像select和poll那样每次调用都要重复传入文件描述符集合事件表。但epoll需要使用一个额外的文件描述符，来唯一标识内核中这个事件表，这个文件描述符使用如下epoll_create函数创建 — [Updated on 2023-03-28 20:21:16](https://hyp.is/_bYChs1gEe2ZCuMXY7LSWw/www.yuque.com/tuobaaxiu/inyebp/tk3ywg?) — Group: #Public
    - Annotation: 缺点：epoll只能工作在linux下








