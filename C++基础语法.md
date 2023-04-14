---
doc_type: hypothesis-highlights
url: >-
  https://interviewguide.cn/notes/03-hunting_job/02-interview/01-01-01-basic.html
---

# 基础语法-01-20

## Metadata
- Author: [interviewguide.cn]()
- Title: 基础语法-01-20
- Reference: https://interviewguide.cn/notes/03-hunting_job/02-interview/01-01-01-basic.html
- Category: #article

## Page Notes
## Highlights
- 指针和引用的区别 指针是一个变量，存储的是一个地址，引用跟原来的变量实质上是同一个东西，是原变量的别名 指针可以有多级，引用只有一级 指针可以为空，引用不能为NULL且在定义时必须初始化 指针在初始化后可以改变指向，而引用在初始化之后不可再改变 sizeof指针得到的是本指针的大小，sizeof引用得到的是引用所指向变量的大小 当把指针作为参数进行传递时，也是将实参的一个拷贝传递给形参，两者指向的地址相同，但不是同一个变量，在函数中改变这个变量的指向不影响实参，而引用却可以。 引用本质是一个指针，同样会占4字节内存；指针是具体变量，需要占用存储空间（，具体情况还要具体分析）。 引用在声明时必须初始化为另一变量，一旦出现必须为typename refname &varname形式；指针声明和定义可以分开，可以先只声明指针变量而不初始化，等用到时再指向具体变量。 引用一旦初始化之后就不可以再改变（变量可以被引用为多次，但引用只能作为一个变量引用）；指针变量可以重新指向别的变量。 不存在指向空值的引用，必须有具体实体；但是存在指向空值的指针。 — [Updated on 2023-03-27 15:38:27](https://hyp.is/XO17rMxyEe2tc9cVRn0Slg/interviewguide.cn/notes/03-hunting_job/02-interview/01-01-01-basic.html) — Group: #Public

- 4、在传递函数参数时，什么时候该使用指针，什么时候该使用引用呢？ 需要返回函数内局部变量的内存的时候用指针。使用指针传参需要开辟内存，用完要记得释放指针，不然会内存泄漏。而返回局部变量的引用是没有意义的 对栈空间大小比较敏感（比如递归）的时候使用引用。使用引用传递不需要创建临时变量，开销要更小 类对象作为参数传递的时候使用引用，这是C++类对象传递的标准方式 — [Updated on 2023-03-27 15:57:47](https://hyp.is/EDChGsx1Ee22x7NyeZqiDQ/interviewguide.cn/notes/03-hunting_job/02-interview/01-01-01-basic.html) — Group: #Public

- 5、堆和栈的区别 申请方式不同。 栈由系统自动分配。 堆是自己申请和释放的。 申请大小限制不同。 栈顶和栈底是之前预设好的，栈是向栈底扩展，大小固定，可以通过ulimit -a查看，由ulimit -s修改。 堆向高地址扩展，是不连续的内存区域，大小可以灵活调整。 申请效率不同。 栈由系统分配，速度快，不会有碎片。 堆由程序员分配，速度慢，且会有碎片。 栈空间默认是4M, 堆区一般是 1G - 4G — [Updated on 2023-03-27 15:58:00](https://hyp.is/GDCsrMx1Ee2H1i-Lh5axZg/interviewguide.cn/notes/03-hunting_job/02-interview/01-01-01-basic.html) — Group: #Public

- 堆向上，向高地址方向增长。 栈向下，向低地址方向增长。 — [Updated on 2023-03-27 16:11:00](https://hyp.is/6Od0JMx2Ee2tDduU7gWvcw/interviewguide.cn/notes/03-hunting_job/02-interview/01-01-01-basic.html) — Group: #Public

- 8、new / delete 与 malloc / free的异同 相同点 都可用于内存的动态申请和释放 不同点 前者是C++运算符，后者是C/C++语言标准库函数 new自动计算要分配的空间大小，malloc需要手工计算 new是类型安全的，malloc不是。例如： — [Updated on 2023-03-27 18:02:59](https://hyp.is/jZF1usyGEe2mAksLAHwQrA/interviewguide.cn/notes/03-hunting_job/02-interview/01-01-01-basic.html) — Group: #Public

- new和delete是如何实现的？ new的实现过程是：首先调用名为operator new的标准库函数，分配足够大的原始为类型化的内存，以保存指定类型的一个对象；接下来运行该类型的一个构造函数，用指定初始化构造对象；最后返回指向新分配并构造后的的对象的指针 delete的实现过程：对指针指向的对象运行适当的析构函数；然后通过调用名为operator delete的标准库函数释放该对象所用内存 — [Updated on 2023-03-27 18:05:50](https://hyp.is/87qmcsyGEe20AGNQT0NVQw/interviewguide.cn/notes/03-hunting_job/02-interview/01-01-01-basic.html) — Group: #Public

- malloc和new的区别？ malloc和free是标准库函数，支持覆盖；new和delete是运算符，支持重载。 malloc仅仅分配内存空间，free仅仅回收空间，不具备调用构造函数和析构函数功能，用malloc分配空间存储类的对象存在风险；new和delete除了分配回收功能外，还会调用构造函数和析构函数。 malloc和free返回的是void类型指针（必须进行类型转换），new和delete返回的是具体类型指针。 — [Updated on 2023-03-27 18:06:32](https://hyp.is/DNwezsyHEe2VWPOCYjpclw/interviewguide.cn/notes/03-hunting_job/02-interview/01-01-01-basic.html) — Group: #Public

- 既然有了malloc/free，C++中为什么还需要new/delete呢？直接用malloc/free不好吗？ malloc/free和new/delete都是用来申请内存和回收内存的。 在对非基本数据类型的对象使用的时候，对象创建的时候还需要执行构造函数，销毁的时候要执行析构函数。而malloc/free是库函数，是已经编译的代码，所以不能把构造函数和析构函数的功能强加给malloc/free，所以new/delete是必不可少的。 — [Updated on 2023-03-27 18:07:05](https://hyp.is/ILNRVsyHEe20OouAuzMDpQ/interviewguide.cn/notes/03-hunting_job/02-interview/01-01-01-basic.html) — Group: #Public

- 12、被free回收的内存是立即返还给操作系统吗？ 不是的，被free回收的内存会首先被ptmalloc使用双链表保存起来，当用户下一次申请内存的时候，会尝试从这些内存中寻找合适的返回。这样就避免了频繁的系统调用，占用过多的系统资源。同时ptmalloc也会尝试对小块内存进行合并，避免过多的内存碎片。 — [Updated on 2023-03-27 19:17:13](https://hyp.is/7NSRpsyQEe2Uya--PSuVJw/interviewguide.cn/notes/03-hunting_job/02-interview/01-01-01-basic.html) — Group: #Public

- 13、宏定义和函数有何区别？ 宏在预处理阶段完成替换，之后被替换的文本参与编译，相当于直接插入了代码，运行时不存在函数调用，执行起来更快；函数调用在运行时需要跳转到具体调用函数。 宏定义属于在结构中插入代码，没有返回值；函数调用具有返回值。 宏定义参数没有类型，不进行类型检查；函数参数具有类型，需要检查类型。 宏定义不要在最后加分号。 — [Updated on 2023-03-27 19:20:21](https://hyp.is/XIWcIMyREe2EAJvtnj63HA/interviewguide.cn/notes/03-hunting_job/02-interview/01-01-01-basic.html) — Group: #Public

- 14、宏定义和typedef区别？ 宏主要用于定义常量及书写复杂的内容；typedef主要用于定义类型别名。 宏替换发生在编译阶段之前，属于文本插入替换；typedef是编译的一部分。 宏不检查类型；typedef会检查数据类型。 宏不是语句，不在在最后加分号；typedef是语句，要加分号标识结束。 注意对指针的操作，typedef char * p_char和#define p_char char *区别巨大。 — [Updated on 2023-03-27 19:20:43](https://hyp.is/aaTBasyREe2x47N7cKl0PA/interviewguide.cn/notes/03-hunting_job/02-interview/01-01-01-basic.html) — Group: #Public
















---
doc_type: hypothesis-highlights
url: >-
  https://interviewguide.cn/notes/03-hunting_job/02-interview/01-01-02-basic.html
---

# 基础语法-21-40

## Metadata
- Author: [interviewguide.cn]()
- Title: 基础语法-21-40
- Reference: https://interviewguide.cn/notes/03-hunting_job/02-interview/01-01-02-basic.html
- Category: #article

## Page Notes
## Highlights
- 22、C++中struct和class的区别 相同点 两者都拥有成员函数、公有和私有部分 任何可以使用class完成的工作，同样可以使用struct完成 不同点 两者中如果不对成员不指定公私有，struct默认是公有的，class则默认是私有的 class默认是private继承， 而struct默认是public继承 — [Updated on 2023-03-27 19:57:40](https://hyp.is/kz-CMMyWEe20K7Os95RgFQ/interviewguide.cn/notes/03-hunting_job/02-interview/01-01-02-basic.html) — Group: #Public

- static 不考虑类的情况 隐藏。所有不加static的全局变量和函数具有全局可见性，可以在其他文件中使用，加了之后只能在该文件所在的编译模块中使用 默认初始化为0，包括未初始化的全局静态变量与局部静态变量，都存在全局未初始化区 静态变量在函数内定义，始终存在，且只进行一次初始化，具有记忆性，其作用范围与局部变量相同，函数退出后仍然存在，但不能使用 — [Updated on 2023-03-27 19:59:09](https://hyp.is/yFRovsyWEe2rmGecx5YjeA/interviewguide.cn/notes/03-hunting_job/02-interview/01-01-02-basic.html) — Group: #Public

- 考虑类的情况 static成员变量：只与类关联，不与类的对象关联。定义时要分配空间，不能在类声明中初始化，必须在类定义体外部初始化，初始化时不需要标示为static；可以被非static成员函数任意访问。 static成员函数：不具有this指针，无法访问类对象的非static成员变量和非static成员函数；不能被声明为const、虚函数和volatile；可以被非static成员函数任意访问 — [Updated on 2023-03-27 19:59:40](https://hyp.is/2s2_QMyWEe2VigdweVdEmA/interviewguide.cn/notes/03-hunting_job/02-interview/01-01-02-basic.html) — Group: #Public

- const 不考虑类的情况 const常量在定义时必须初始化，之后无法更改 const形参可以接收const和非const类型的实参，例如// i 可以是 int 型或者 const int 型void fun(const int& i){ //...} 考虑类的情况 const成员变量：不能在类定义外部初始化，只能通过构造函数初始化列表进行初始化，并且必须有构造函数；不同类对其const数据成员的值可以不同，所以不能在类中声明时初始化 const成员函数：const对象不可以调用非const成员函数；非const对象都可以调用；不可以改变非mutable（用该关键字声明的变量可以在const成员函数中被修改）数据的值 — [Updated on 2023-03-27 20:00:00](https://hyp.is/5t2IGsyWEe2TZ4Mq0tl3zw/interviewguide.cn/notes/03-hunting_job/02-interview/01-01-02-basic.html) — Group: #Public

- 26、数组名和指针（这里为指向数组首元素的指针）区别？ 二者均可通过增减偏移量来访问数组中的元素。 数组名不是真正意义上的指针，可以理解为常指针，所以数组名没有自增、自减等操作。 当数组名当做形参传递给调用函数后，就失去了原有特性，退化成一般指针，多了自增、自减操作，但sizeof运算符不能再得到原数组的大小了。 # — [Updated on 2023-03-27 20:47:58](https://hyp.is/miHr1sydEe2uGKdogWOMSA/interviewguide.cn/notes/03-hunting_job/02-interview/01-01-02-basic.html) — Group: #Public

- override的作用就出来了，它指定了子类的这个虚函数是重写的父类的，如果你名字不小心打错了的话，编译器是不会编译通过的： — [Updated on 2023-03-27 20:49:51](https://hyp.is/3Vqx0MydEe237JcgSG7XZQ/interviewguide.cn/notes/03-hunting_job/02-interview/01-01-02-basic.html) — Group: #Public

- final 当不希望某个类被继承，或不希望某个虚函数被重写，可以在类名和虚函数后添加final关键字，添加final关键字后被继承或重写，编译器会报错。 — [Updated on 2023-03-27 20:50:12](https://hyp.is/6durbMydEe20UOeSSouxSg/interviewguide.cn/notes/03-hunting_job/02-interview/01-01-02-basic.html) — Group: #Public

- 直接初始化直接调用与实参匹配的构造函数，拷贝初始化总是调用拷贝构造函数。拷贝初始化首先使用指定构造函数创建一个临时对象，然后用拷贝构造函数将那个临时对象拷贝到正在创建的对象 — [Updated on 2023-03-27 20:59:03](https://hyp.is/JsCCpMyfEe2LOnNjGh9hsA/interviewguide.cn/notes/03-hunting_job/02-interview/01-01-02-basic.html) — Group: #Public

- 为了能够正确的在C++代码中调用C语言的代码：在程序中加上extern "C"后，相当于告诉编译器这部分代码是C语言写的，因此要按照C语言进行编译，而不是C++； 哪些情况下使用extern "C"： （1）C++代码中调用C语言代码； （2）在C++中的头文件中使用； （3）在多个人协同开发时，可能有人擅长C语言，而有人擅长C++； — [Updated on 2023-03-27 21:00:43](https://hyp.is/YkxtVsyfEe2U_lukGEI6wg/interviewguide.cn/notes/03-hunting_job/02-interview/01-01-02-basic.html) — Group: #Public

- 31、野指针和悬空指针 都是是指向无效内存区域(这里的无效指的是"不安全不可控")的指针，访问行为将会导致未定义行为。 野指针 野指针，指的是没有被初始化过的指针 — [Updated on 2023-03-27 21:02:45](https://hyp.is/qwwR4MyfEe2xlg-mo5UOJA/interviewguide.cn/notes/03-hunting_job/02-interview/01-01-02-basic.html) — Group: #Public

- 因此，为了防止出错，对于指针初始化时都是赋值为 nullptr，这样在使用时编译器就不会直接报错，产生非法内存访问。 悬空指针 悬空指针，指针最初指向的内存已经被释放了的一种指针。 — [Updated on 2023-03-27 21:02:53](https://hyp.is/r2xPwMyfEe2x9aPQVVSmgA/interviewguide.cn/notes/03-hunting_job/02-interview/01-01-02-basic.html) — Group: #Public

- 需要设置为p=p2=nullptr。此时再使用，编译器会直接保错。 避免野指针比较简单，但悬空指针比较麻烦。c++引入了智能指针，C++智能指针的本质就是避免悬空指针的产生。 — [Updated on 2023-03-27 21:03:03](https://hyp.is/tXMSbsyfEe2x9k-w49nFAg/interviewguide.cn/notes/03-hunting_job/02-interview/01-01-02-basic.html) — Group: #Public

- 野指针：指针变量未及时初始化 => 定义指针变量及时初始化，要么置空。 悬空指针：指针free或delete之后没有及时置空 => 释放操作后立即置空。 — [Updated on 2023-03-27 21:03:13](https://hyp.is/u640usyfEe2uHbeq70V5QQ/interviewguide.cn/notes/03-hunting_job/02-interview/01-01-02-basic.html) — Group: #Public

- 操作符new返回的指针类型严格与对象匹配，而不是void* — [Updated on 2023-03-27 21:04:51](https://hyp.is/9b6jsMyfEe2uIMdAFUgBSQ/interviewguide.cn/notes/03-hunting_job/02-interview/01-01-02-basic.html) — Group: #Public

- 33、C++中的重载、重写（覆盖）和隐藏的区别 （1）重载（overload） 重载是指在同一范围定义中的同名成员函数才存在重载关系。主要特点是函数名相同，参数类型和数目有所不同，不能出现参数个数和类型均相同，仅仅依靠返回值不同来区分的函数。重载和函数成员是否是虚函数无关 — [Updated on 2023-03-27 21:05:45](https://hyp.is/Fi0UTMygEe2VAvPv4Xn0GA/interviewguide.cn/notes/03-hunting_job/02-interview/01-01-02-basic.html) — Group: #Public

- （2）重写（覆盖）（override） 重写指的是在派生类中覆盖基类中的同名函数，重写就是重写函数体，要求基类函数必须是虚函数且： 与基类的虚函数有相同的参数个数 与基类的虚函数有相同的参数类型 与基类的虚函数有相同的返回值类型 — [Updated on 2023-03-27 21:06:04](https://hyp.is/IVex9sygEe2tf8tJ8YwH4Q/interviewguide.cn/notes/03-hunting_job/02-interview/01-01-02-basic.html) — Group: #Public

- 重载与重写的区别： 重写是父类和子类之间的垂直关系，重载是不同函数之间的水平关系 重写要求参数列表相同，重载则要求参数列表不同，返回值不要求 重写关系中，调用方法根据对象类型决定，重载根据调用时实参表与形参表的对应关系来选择函数体 — [Updated on 2023-03-27 21:06:17](https://hyp.is/KN2lKsygEe2EMhuLjOn83Q/interviewguide.cn/notes/03-hunting_job/02-interview/01-01-02-basic.html) — Group: #Public

- （3）隐藏（hide） 隐藏指的是某些情况下，派生类中的函数屏蔽了基类中的同名函数，包括以下情况： 两个函数参数相同，但是基类函数不是虚函数。**和重写的区别在于基类函数是否是虚函数。 — [Updated on 2023-03-27 21:07:06](https://hyp.is/Rib7SsygEe21OhekE6lQNg/interviewguide.cn/notes/03-hunting_job/02-interview/01-01-02-basic.html) — Group: #Public
    - Annotation: 重定义
- 两个函数参数不同，无论基类函数是不是虚函数，都会被隐藏。和重载的区别在于两个函数不在同一个类中。举个例子： — [Updated on 2023-03-27 21:07:24](https://hyp.is/UNdzEsygEe25fIMFT_Qt7w/interviewguide.cn/notes/03-hunting_job/02-interview/01-01-02-basic.html) — Group: #Public

- 34、C++有哪几种的构造函数 C++中的构造函数可以分为4类： 默认构造函数 初始化构造函数（有参数） 拷贝构造函数 移动构造函数（move和右值引用） 委托构造函数 转换构造函数 — [Updated on 2023-03-27 21:09:21](https://hyp.is/loqZXMygEe23YcOWVn6KyQ/interviewguide.cn/notes/03-hunting_job/02-interview/01-01-02-basic.html) — Group: #Public

- 默认构造函数和初始化构造函数在定义类的对象，完成对象的初始化工作 复制构造函数用于复制本类的对象 转换构造函数用于将其他类型的变量，隐式转换为本类对象 — [Updated on 2023-03-27 21:09:49](https://hyp.is/p34IwMygEe20i1PUfHbQyA/interviewguide.cn/notes/03-hunting_job/02-interview/01-01-02-basic.html) — Group: #Public

- 36、内联函数和宏定义的区别 在使用时，宏只做简单字符串替换（编译前）。而内联函数可以进行参数类型检查（编译时），且具有返回值。 内联函数在编译时直接将函数代码嵌入到目标代码中，省去函数调用的开销来提高执行效率，并且进行参数类型检查，具有返回值，可以实现重载。 宏定义时要注意书写（参数要括起来）否则容易出现歧义，内联函数不会产生歧义 内联函数有类型检测、语法判断等功能，而宏没有 — [Updated on 2023-03-27 21:41:35](https://hyp.is/F3MzXsylEe2qjGto88t-Hg/interviewguide.cn/notes/03-hunting_job/02-interview/01-01-02-basic.html) — Group: #Public

- 37、public，protected和private访问和继承权限/public/protected/private的区别？ public的变量和函数在类的内部外部都可以访问。 protected的变量和函数只能在类的内部和其派生类中访问。 private修饰的元素只能在类内访问。 — [Updated on 2023-03-27 21:42:20](https://hyp.is/MinmjsylEe2LV6_oneulxA/interviewguide.cn/notes/03-hunting_job/02-interview/01-01-02-basic.html) — Group: #Public

- public继承 公有继承的特点是基类的公有成员和保护成员作为派生类的成员时，都保持原有的状态，而基类的私有成员任然是私有的，不能被这个派生类的子类所访问 — [Updated on 2023-03-27 21:52:54](https://hyp.is/rG6knMymEe20fsOxX5djxw/interviewguide.cn/notes/03-hunting_job/02-interview/01-01-02-basic.html) — Group: #Public

- protected继承 保护继承的特点是基类的所有公有成员和保护成员都成为派生类的保护成员，并且只能被它的派生类成员函数或友元函数访问，基类的私有成员仍然是私有的，访问规则如下表 — [Updated on 2023-03-27 21:52:58](https://hyp.is/ru64QsymEe2WMv8PFY3wFw/interviewguide.cn/notes/03-hunting_job/02-interview/01-01-02-basic.html) — Group: #Public

- private继承 私有继承的特点是基类的所有公有成员和保护成员都成为派生类的私有成员，并不被它的派生类的子类所访问，基类的成员只能由自己派生类访问，无法再往下继承，访问规则如下表 — [Updated on 2023-03-27 21:53:07](https://hyp.is/s9-CWsymEe2qk-ff4qey_g/interviewguide.cn/notes/03-hunting_job/02-interview/01-01-02-basic.html) — Group: #Public

- 大端存储：字数据的高字节存储在低地址中 小端存储：字数据的低字节存储在低地址中 — [Updated on 2023-03-27 21:54:58](https://hyp.is/9nedoMymEe2WhrN56qZjdQ/interviewguide.cn/notes/03-hunting_job/02-interview/01-01-02-basic.html) — Group: #Public
























---
doc_type: hypothesis-highlights
url: >-
  https://interviewguide.cn/notes/03-hunting_job/02-interview/01-01-03-basic.html
---

# 基础语法-41-60

## Metadata
- Author: [interviewguide.cn]()
- Title: 基础语法-41-60
- Reference: https://interviewguide.cn/notes/03-hunting_job/02-interview/01-01-03-basic.html
- Category: #article

## Page Notes
## Highlights
- palcement new的主要用途就是反复使用一块较大的动态分配的内存来构造不同类型的对象或者他们的数组 — [Updated on 2023-03-30 17:06:52](https://hyp.is/Nmymgs7aEe2nDisbMWqQLg/interviewguide.cn/notes/03-hunting_job/02-interview/01-01-03-basic.html) — Group: #Public

- placement new构造起来的对象数组，要显式的调用他们的析构函数来销毁（析构函数并不释放对象的内存），千万不要使用delete，这是因为placement new构造起来的对象或数组大小并不一定等于原来分配的内存大小，使用delete会造成内存泄漏或者之后释放内存时出现运行时错误。 — [Updated on 2023-03-30 17:07:06](https://hyp.is/Po9InM7aEe2IHU8cAdp7Xw/interviewguide.cn/notes/03-hunting_job/02-interview/01-01-03-basic.html) — Group: #Public

- 值传递、指针传递、引用传递的区别和效率 值传递：有一个形参向函数所属的栈拷贝数据的过程，如果值传递的对象是类对象 或是大的结构体对象，将耗费一定的时间和空间。（传值） 指针传递：同样有一个形参向函数所属的栈拷贝数据的过程，但拷贝的数据是一个固定为4字节的地址。（传值，传递的是地址值） 引用传递：同样有上述的数据拷贝过程，但其是针对地址的，相当于为该数据所在的地址起了一个别名。（传地址） 效率上讲，指针传递和引用传递比值传递效率高。一般主张使用引用传递，代码逻辑上更加紧凑、清晰。 # — [Updated on 2023-03-30 17:25:15](https://hyp.is/x21-ys7cEe2cNFdRDdbbwA/interviewguide.cn/notes/03-hunting_job/02-interview/01-01-03-basic.html) — Group: #Public

- 43、static的用法和作用？ 1.先来介绍它的第一条也是最重要的一条：隐藏。（static函数，static变量均可） 当同时编译多个文件时，所有未加static前缀的全局变量和函数都具有全局可见性。 2.static的第二个作用是保持变量内容的持久。（static变量中的记忆功能和全局生存期）存储在静态数据区的变量会在程序刚开始运行时就完成初始化，也是唯一的一次初始化。共有两种变量存储在静态存储区：全局变量和static变量，只不过和全局变量比起来，static可以控制变量的可见范围，说到底static还是用来隐藏的。 — [Updated on 2023-04-05 15:29:11](https://hyp.is/j3lJxtODEe2NPHM1Jyu1Zw/interviewguide.cn/notes/03-hunting_job/02-interview/01-01-03-basic.html) — Group: #Public

- 4.static的第四个作用：C++中的类成员声明static 函数体内static变量的作用范围为该函数体，不同于auto变量，该变量的内存只被分配一次，因此其值在下次调用时仍维持上次的值； 在模块内的static全局变量可以被模块内所有函数访问，但不能被模块外其它函数访问； 在模块内的static函数只可被这一模块内的其它函数调用，这个函数的使用范围被限制在声明它的模块内； 在类中的static成员变量属于整个类所拥有，对类的所有对象只有一份拷贝； 在类中的static成员函数属于整个类所拥有，这个函数不接收this指针，因而只能访问类的static成员变量。 — [Updated on 2023-04-05 15:29:47](https://hyp.is/pIF7QNODEe2oqJvx8WJ_kA/interviewguide.cn/notes/03-hunting_job/02-interview/01-01-03-basic.html) — Group: #Public

- static类对象必须要在类外进行初始化，static修饰的变量先于对象存在，所以static修饰的变量要在类外初始化； 由于static修饰的类成员属于类，不属于对象，因此static类成员函数是没有this指针的，this指针是指向本对象的指针。正因为没有this指针，所以static类成员函数不能访问非static的类成员，只能访问 static修饰的类成员； static成员函数不能被virtual修饰，static成员不属于任何对象或实例，所以加上virtual没有任何实际意义；静态成员函数没有this指针，虚函数的实现是为每一个对象分配一个vptr指针，而vptr是通过this指针调用的，所以不能为virtual；虚函数的调用关系，this->vptr->ctable->virtual function # — [Updated on 2023-04-05 15:29:57](https://hyp.is/qtAeZtODEe2vknc61fTGaQ/interviewguide.cn/notes/03-hunting_job/02-interview/01-01-03-basic.html) — Group: #Public

- 实参可以是常量、变量、表达式、函数等， 无论实参是何种类型的量，在进行函数调用时，它们都必须具有确定的值， 以便把这些值传送给形参。 因此应预先用赋值，输入等办法使实参获得确定值，会产生一个临时变量 — [Updated on 2023-04-05 15:48:01](https://hyp.is/MJdO5tOGEe2vt0OGSJP1yg/interviewguide.cn/notes/03-hunting_job/02-interview/01-01-03-basic.html) — Group: #Public

- 当形参和实参不是指针类型时，在该函数运行时，形参和实参是不同的变量，他们在内存中位于不同的位置，形参将实参的内容复制一份，在该函数运行结束的时候形参被释放，而实参内容不会改变。 — [Updated on 2023-04-05 15:48:17](https://hyp.is/OmeVDNOGEe28FEv_Kr6eFg/interviewguide.cn/notes/03-hunting_job/02-interview/01-01-03-basic.html) — Group: #Public

- 。所以C++标准定为全局或静态对象是有首次用到时才会进行构造，并通过atexit()来管理。在程序结束，按照构造顺序反方向进行逐个析构。所以在C++中是可以使用变量对静态局部变量进行初始化的。 — [Updated on 2023-04-05 16:01:21](https://hyp.is/Dbyx8tOIEe2c-8N0wOq6Ng/interviewguide.cn/notes/03-hunting_job/02-interview/01-01-03-basic.html) — Group: #Public

- 在C中，初始化发生在代码执行之前，编译阶段分配好内存之后，就会进行初始化，所以我们看到在C语言中无法使用变量对静态局部变量进行初始化 — [Updated on 2023-04-05 16:01:30](https://hyp.is/EseCMNOIEe2NJGs6LS5L6w/interviewguide.cn/notes/03-hunting_job/02-interview/01-01-03-basic.html) — Group: #Public

- 阻止一个变量被改变，可以使用const关键字。在定义该const变量时，通常需要对它进行初始化，因为以后就没有机会再去改变它了； 对指针来说，可以指定指针本身为const，也可以指定指针所指的数据为const，或二者同时指定为const； 在一个函数声明中，const可以修饰形参，表明它是一个输入参数，在函数内部不能改变其值； — [Updated on 2023-04-05 16:03:29](https://hyp.is/WiGpCNOIEe2vn0MuLQDl8g/interviewguide.cn/notes/03-hunting_job/02-interview/01-01-03-basic.html) — Group: #Public

- 对于类的成员函数，若指定其为const类型，则表明其是一个常函数，不能修改类的成员变量，类的常对象只能访问类的常成员函数； — [Updated on 2023-04-05 16:09:53](https://hyp.is/PohDwtOJEe2de9_tO0RTcQ/interviewguide.cn/notes/03-hunting_job/02-interview/01-01-03-basic.html) — Group: #Public

- const成员函数可以访问非const对象的非const数据成员、const数据成员，也可以访问const对象内的所有数据成员； — [Updated on 2023-04-05 16:10:40](https://hyp.is/WwLxPNOJEe248deXICnDVA/interviewguide.cn/notes/03-hunting_job/02-interview/01-01-03-basic.html) — Group: #Public

- 非const成员函数可以访问非const对象的非const数据成员、const数据成员，但不可以访问const对象的任意数据成员； — [Updated on 2023-04-05 16:10:57](https://hyp.is/ZOjlxtOJEe2UNAcooHHbWw/interviewguide.cn/notes/03-hunting_job/02-interview/01-01-03-basic.html) — Group: #Public

- const类型变量可以通过类型转换符const_cast将const类型转换为非const类型； — [Updated on 2023-04-05 16:12:28](https://hyp.is/mzHJGNOJEe22qqN8387MwA/interviewguide.cn/notes/03-hunting_job/02-interview/01-01-03-basic.html) — Group: #Public

- const类型变量必须定义的时候进行初始化，因此也导致如果类的成员变量有const类型的变量，那么该变量必须在类的初始化列表中进行初始化； — [Updated on 2023-04-05 16:12:37](https://hyp.is/oM-UItOJEe2HoUPqRyYXqw/interviewguide.cn/notes/03-hunting_job/02-interview/01-01-03-basic.html) — Group: #Public

- 但是在引用或指针传递函数调用中，因为传进去的是一个引用或指针，这样函数内部可以改变引用或指针所指向的变量，这时const 才是实实在在地保护了实参所指向的变量。 — [Updated on 2023-04-05 16:13:06](https://hyp.is/sY8EeNOJEe2Vbp_dr8uAYA/interviewguide.cn/notes/03-hunting_job/02-interview/01-01-03-basic.html) — Group: #Public

- 有引用传递和指针传递可以用是否加const来重载。一个拥有顶层const的形参无法和另一个没有顶层const的形参区分开来。 — [Updated on 2023-04-05 16:13:22](https://hyp.is/u21-hNOJEe2DJ7epErE5Eg/interviewguide.cn/notes/03-hunting_job/02-interview/01-01-03-basic.html) — Group: #Public

- has-A包含关系，用以描述一个类由多个部件类构成，实现has-A关系用类的成员属性表示，即一个类的成员属性是另一个已经定义好的类； use-A，一个类使用另一个类，通过类之间的成员函数相互联系，定义友元或者通过传递参数的方式来实现； is-A，继承关系，关系具有传递性； — [Updated on 2023-04-05 16:17:36](https://hyp.is/UpqrztOKEe2v-ie8gSDFMg/interviewguide.cn/notes/03-hunting_job/02-interview/01-01-03-basic.html) — Group: #Public

- 所谓的继承就是一个类继承了另一个类的属性和方法，这个新的类包含了上一个类的属性和方法，被称为子类或者派生类，被继承的类称为父类或者基类； — [Updated on 2023-04-05 16:17:51](https://hyp.is/W6a9jtOKEe2_bRe1NWI6Gw/interviewguide.cn/notes/03-hunting_job/02-interview/01-01-03-basic.html) — Group: #Public

- 子类拥有父类的所有属性和方法，子类可以拥有父类没有的属性和方法，子类对象可以当做父类对象使用； — [Updated on 2023-04-05 16:18:03](https://hyp.is/YubbdNOKEe2x7t9_6dxo8w/interviewguide.cn/notes/03-hunting_job/02-interview/01-01-03-basic.html) — Group: #Public

- 浅复制 ：只是拷贝了基本类型的数据，而引用类型数据，复制后也是会发生引用，我们把这种拷贝叫做“（浅复制）浅拷贝”，换句话说，浅复制仅仅是指向被复制的内存地址，如果原地址中对象被改变了，那么浅复制出来的对象也会相应改变。 深复制 ：在计算机中开辟了一块新的内存地址用于存放复制的对象。 在某些状况下，类内成员变量需要动态开辟堆内存，如果实行位拷贝，也就是把对象里的值完全复制给另一个对象，如A=B。这时，如果B中有一个成员变量指针已经申请了内存，那A中的那个成员变量也指向同一块内存。这就出现了问题：当B把内存释放了（如：析构），这时A内的指针就是野指针了，出现运行错误。 — [Updated on 2023-04-05 16:27:04](https://hyp.is/pQipvtOLEe2vzJeV-RhHTQ/interviewguide.cn/notes/03-hunting_job/02-interview/01-01-03-basic.html) — Group: #Public

- 3、 new操作符内存分配成功时，返回的是对象类型的指针，类型严格与对象匹配，无须进行类型转换，故new是符合类型安全性的操作符。而malloc内存分配成功则是返回void * ，需要通过强制类型转换将void*指针转换成我们需要的类型。 — [Updated on 2023-04-05 16:27:43](https://hyp.is/vLUr3NOLEe2HrM_0mQSJWQ/interviewguide.cn/notes/03-hunting_job/02-interview/01-01-03-basic.html) — Group: #Public

- 1、 new/delete是C++关键字，需要编译器支持。malloc/free是库函数，需要头文件支持； 2、 使用new操作符申请内存分配时无须指定内存块的大小，编译器会根据类型信息自行计算。而malloc则需要显式地指出所需内存的尺寸。 — [Updated on 2023-04-05 16:27:47](https://hyp.is/vqu26tOLEe2wArOzsGjxpw/interviewguide.cn/notes/03-hunting_job/02-interview/01-01-03-basic.html) — Group: #Public

- 4、 new内存分配失败时，会抛出bac_alloc异常。malloc分配内存失败时返回NULL。 5、 new会先调用operator new函数，申请足够的内存（通常底层使用malloc实现）。然后调用类型的构造函数，初始化成员变量，最后返回自定义类型指针。delete先调用析构函数，然后调用operator delete函数释放内存（通常底层使用free实现）。malloc/free是库函数，只能动态的申请和释放内存，无法强制要求其做自定义类型对象构造和析构工作。 — [Updated on 2023-04-05 16:30:55](https://hyp.is/Lvtt-tOMEe2Sup-D7S1sYg/interviewguide.cn/notes/03-hunting_job/02-interview/01-01-03-basic.html) — Group: #Public

- 动态数组管理new一个数组时，[]中必须是一个整数，但是不一定是常量整数，普通数组必须是一个常量整数； — [Updated on 2023-04-05 16:31:11](https://hyp.is/OIWG7tOMEe2NNA9-E5yxBA/interviewguide.cn/notes/03-hunting_job/02-interview/01-01-03-basic.html) — Group: #Public

- new动态数组返回的并不是数组类型，而是一个元素类型的指针； — [Updated on 2023-04-05 16:31:25](https://hyp.is/QK6ovtOMEe2wA5dZRYPquw/interviewguide.cn/notes/03-hunting_job/02-interview/01-01-03-basic.html) — Group: #Public

- 3、 delete[]时，数组中的元素按逆序的顺序进行销毁； — [Updated on 2023-04-05 16:32:03](https://hyp.is/V1NkLtOMEe2x89f7nJUt-A/interviewguide.cn/notes/03-hunting_job/02-interview/01-01-03-basic.html) — Group: #Public

- 4、 new在内存分配上面有一些局限性，new的机制是将内存分配和对象构造组合在一起，同样的，delete也是将对象析构和内存释放组合在一起的。allocator将这两部分分开进行，allocator申请一部分内存，不进行初始化对象，只有当需要的时候才进行初始化操作。 # — [Updated on 2023-04-05 16:32:22](https://hyp.is/YuREXNOMEe2dD88ebNK3pQ/interviewguide.cn/notes/03-hunting_job/02-interview/01-01-03-basic.html) — Group: #Public

- new[]先调用operator new[]分配内存，然后在p的前四个字节写入数组大小n，然后调用n次构造函数，针对复杂类型，new[]会额外存储数组大小； — [Updated on 2023-04-05 16:35:20](https://hyp.is/zPvJtNOMEe2VdRea2b7JfA/interviewguide.cn/notes/03-hunting_job/02-interview/01-01-03-basic.html) — Group: #Public

- new表达式调用一个名为operator new(operator new[])函数，分配一块足够大的、原始的、未命名的内存空间； ② 编译器运行相应的构造函数以构造这些对象，并为其传入初始值； ③ 对象被分配了空间并构造完成，返回一个指向该对象的指针。 — [Updated on 2023-04-05 16:35:33](https://hyp.is/1M5i0tOMEe2NOLfhorVbRg/interviewguide.cn/notes/03-hunting_job/02-interview/01-01-03-basic.html) — Group: #Public

- delete简单数据类型默认只是调用free函数；复杂数据类型先调用析构函数再调用operator delete；针对简单类型，delete和delete[]等同。假设指针p指向new[]分配的内存。因为要4字节存储数组大小，实际分配的内存地址为[p-4]，系统记录的也是这个地址。delete[]实际释放的就是p-4指向的内存。而delete会直接释放p指向的内存，这个内存根本没有被系统记录，所以会崩溃。 — [Updated on 2023-04-05 16:37:05](https://hyp.is/C5BoLtONEe2uGZ-NpWaR0g/interviewguide.cn/notes/03-hunting_job/02-interview/01-01-03-basic.html) — Group: #Public

- 需要在 new [] 一个对象数组时，需要保存数组的维度，C++ 的做法是在分配数组空间时多分配了 4 个字节的大小，专门保存数组的大小，在 delete [] 时就可以取出这个保存的数，就知道了需要调用析构函数多少次了。 — [Updated on 2023-04-05 16:37:14](https://hyp.is/EJ36ZtONEe2_dws2YFDlEg/interviewguide.cn/notes/03-hunting_job/02-interview/01-01-03-basic.html) — Group: #Public

- 55、malloc申请的存储空间能用delete释放吗? 不能，malloc /free主要为了兼容C，new和delete 完全可以取代malloc /free的。 malloc /free的操作对象都是必须明确大小的，而且不能用在动态类上。 new 和delete会自动进行类型检查和大小，malloc/free不能执行构造函数与析构函数，所以动态对象它是不行的。 当然从理论上说使用malloc申请的内存是可以通过delete释放的。不过一般不这样写的。而且也不能保证每个C++的运行时都能正常。 # — [Updated on 2023-04-05 16:37:59](https://hyp.is/K5vgWNONEe29JkcD5zYajQ/interviewguide.cn/notes/03-hunting_job/02-interview/01-01-03-basic.html) — Group: #Public

- 56、malloc与free的实现原理？ — [Updated on 2023-04-05 16:39:42](https://hyp.is/aS4epNONEe2NYos_1412CA/interviewguide.cn/notes/03-hunting_job/02-interview/01-01-03-basic.html) — Group: #Public

- malloc申请的空间的值是随机初始化的，calloc申请的空间的值是初始化为0的； — [Updated on 2023-04-05 16:39:58](https://hyp.is/cpT6stONEe27nG8G-ZTqZw/interviewguide.cn/notes/03-hunting_job/02-interview/01-01-03-basic.html) — Group: #Public

- 赋值初始化，通过在函数体内进行赋值初始化；列表初始化，在冒号后使用初始化列表进行初始化。 — [Updated on 2023-04-05 16:40:29](https://hyp.is/hT8U1tONEe25R-9gxCBdcQ/interviewguide.cn/notes/03-hunting_job/02-interview/01-01-03-basic.html) — Group: #Public

- 对于在函数体中初始化,是在所有的数据成员被分配内存空间后才进行的。 列表初始化是给数据成员分配内存空间时就进行初始化,就是说分配一个数据成员只要冒号后有此数据成员的赋值表达式(此表达式必须是括号赋值表达式),那么分配了内存空间后在进入函数体之前给数据成员赋值，就是说初始化这个数据成员此时函数体还未执行。 — [Updated on 2023-04-05 16:40:41](https://hyp.is/jHDNlNONEe246Zur7UaXog/interviewguide.cn/notes/03-hunting_job/02-interview/01-01-03-basic.html) — Group: #Public

- ③ 类类型的成员对象的构造函数（按照初始化顺序） — [Updated on 2023-04-05 16:41:27](https://hyp.is/p2YhxtONEe2v1ef5m2sN-Q/interviewguide.cn/notes/03-hunting_job/02-interview/01-01-03-basic.html) — Group: #Public

- 必须使用成员初始化的四种情况 ① 当初始化一个引用成员时； ② 当初始化一个常量成员时； ③ 当调用一个基类的构造函数，而它拥有一组参数时； ④ 当调用一个成员类的构造函数，而它拥有一组参数时； — [Updated on 2023-04-05 16:41:54](https://hyp.is/t_1iftONEe2ozTNHDq87AA/interviewguide.cn/notes/03-hunting_job/02-interview/01-01-03-basic.html) — Group: #Public

- string继承自basic_string,其实是对char*进行了封装，封装的string包含了char*数组，容量，长度等等属性。 string可以进行动态扩展，在每次扩展的时候另外申请一块原空间大小两倍的空间（2*n），然后将原字符串拷贝过去，并加上新增的内容。 — [Updated on 2023-04-05 16:42:06](https://hyp.is/vv_85NONEe27nYNTMOi92A/interviewguide.cn/notes/03-hunting_job/02-interview/01-01-03-basic.html) — Group: #Public









