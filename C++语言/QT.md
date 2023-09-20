# 基础知识

## .pro
`$$()`取环境变量中的变量

`$${}`取.pro中自己定义的变量

`message()` 用来打印

## 信号与槽
```c++
connect(btn, &QAbstractButton::clicked,this,&QMainWindow::close);
```
`[static] QMetaObject::Connection QObject::connect(const QObject *sender, const QMetaMethod &signal, const QObject *receiver, const QMetaMethod &method, Qt::ConnectionType type = Qt::AutoConnection)`


## 连接两个信号
```c++
connect(sender, &Sender::exit, receiver, &Receiver::handleExit);

connect(sender, &Sender::exit2, receiver, &Receiver::handleExit2);

connect(sender, &Sender::exit, sender, &Sender::exit2);
```

信号与槽是Qt特有的的消息传输机制，在Qt中信号与槽用得十分广泛。在编程的过程中，我们都会遇到消息传递的事情，本质上就是发出命令（信号、消息），执行命令（相应的执行）。

比如单击窗口上一个按钮然后弹出一个对话框，那么就可以将这个按钮的单击信号和自定义的槽关联起来，信号是按钮的单击信号，槽实现了创建一个对话框并显示的功能。

**信号与槽就是实现对象之间通信的一种机制**，在其他编程语言中也有通过回调机制来实现对象之间的通信。

信号：当对象改变其状态时，信号就由该对象发射 (emit) 出去，而且对象只负责发送信号，它不知道另一端是谁在接收这个信号。
槽：用于接收信号，而且槽只是普通的对象成员函数。一个槽并不知道是否有任何信号与自己相连接。

**信号槽是设计模式==观察者模式==的一种实现**：

A、一个信号就是一个能够被观察的事件，或者至少是事件已经发生的一种通知；
B、一个槽就是一个观察者，通常就是在被观察的对象发生改变的时候——也可以说是信号发出的时候——被调用的函数；
C、信号与槽的连接，形成一种观察者-被观察者的关系；
D、当事件或者状态发生改变的时候，信号就会被发出；同时，信号发出者有义务调用所有注册的对这个事件（信号）感兴趣的函数（槽）。


信号和槽是多对多的关系。一个信号可以连接多个槽，而一个槽也可以监听多个信号。
