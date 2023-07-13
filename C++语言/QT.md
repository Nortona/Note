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

