---
doc_type: hypothesis-highlights
url: 'https://www.cnblogs.com/sunchaothu/p/10389842.html'
---

# C++ 单例模式总结与剖析 - 行者孙 - 博客园

## Metadata
- Author: [cnblogs.com]()
- Title: C++ 单例模式总结与剖析 - 行者孙 - 博客园
- Reference: https://www.cnblogs.com/sunchaothu/p/10389842.html
- Category: #article

## Page Notes
## Highlights
- 什么是单例 单例 Singleton 是设计模式的一种，其特点是只提供唯一一个类的实例,具有全局变量的特点，在任何位置都可以通过接口获取到那个唯一实例; 具体运用场景如： 设备管理器，系统中可能有多个设备，但是只有一个设备管理器，用于管理设备驱动; 数据池，用来缓存数据的数据结构，需要在一处写，多处读取或者多处写，多处读取; — [Updated on 2023-03-26 22:52:22](https://hyp.is/0GonFsvlEe2t_sNQ-fOC-g/www.cnblogs.com/sunchaothu/p/10389842.html) — Group: #Public

- C++单例的实现 2.1 基础要点 全局只有一个实例：static 特性，同时禁止用户自己声明并定义实例（把构造函数设为 private） 线程安全 禁止赋值和拷贝 用户通过接口获取实例：使用 static 类成员函数 — [Updated on 2023-03-26 22:52:26](https://hyp.is/0wbpKMvlEe26AuOV2zZdxQ/www.cnblogs.com/sunchaothu/p/10389842.html) — Group: #Public

- C++ 实现单例的几种方式 2.2.1 有缺陷的懒汉式 懒汉式(Lazy-Initialization)的方法是直到使用时才实例化对象，也就说直到调用get_instance() 方法的时候才 new 一个单例的对象， 如果不被调用就不会占用内存。 — [Updated on 2023-03-26 22:52:34](https://hyp.is/13hsrMvlEe2JIScI3zwxkg/www.cnblogs.com/sunchaothu/p/10389842.html) — Group: #Public

- 这是个最基础版本的单例实现，他有哪些问题呢？ 线程安全的问题,当多线程获取单例时有可能引发竞态条件：第一个线程在if中判断 m_instance_ptr是空的，于是开始实例化单例;同时第2个线程也尝试获取单例，这个时候判断m_instance_ptr还是空的，于是也开始实例化单例;这样就会实例化出两个对象,这就是线程安全问题的由来; 解决办法:加锁 内存泄漏. 注意到类中只负责new出对象，却没有负责delete对象，因此只有构造函数被调用，析构函数却没有被调用;因此会导致内存泄漏。解决办法： 使用共享指针; — [Updated on 2023-03-26 22:52:55](https://hyp.is/5FKR3MvlEe2R-NvHj0uEAg/www.cnblogs.com/sunchaothu/p/10389842.html) — Group: #Public

- 2.2.3 最推荐的懒汉式单例(magic static )——局部静态变量 — [Updated on 2023-03-26 22:53:32](https://hyp.is/-miqQsvlEe21Vbv4CpM59w/www.cnblogs.com/sunchaothu/p/10389842.html) — Group: #Public




