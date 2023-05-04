第二级配置器主动将任何小额区块的内存需求量上调至8的倍数，并维护16个free-list，各自管理大小为8~128bytes的小额区块； — [Updated on 2023-04-06 01:49:51](https://hyp.is/RCzNONPaEe25efPEdKD39Q/interviewguide.cn/notes/03-hunting_job/02-interview/01-04-02-STL.html) — Group: #Public

- .vector 底层数据结构为数组 ，支持快速随机访问 2.list 底层数据结构为双向链表，支持快速增删 3.deque 底层数据结构为一个中央控制器和多个缓冲区 — [Updated on 2023-04-06 01:50:31](https://hyp.is/XADKuNPaEe22xYfY9CD-MQ/interviewguide.cn/notes/03-hunting_job/02-interview/01-04-02-STL.html) — Group: #Public

- deque是一个双端队列(double-ended queue)，也是在堆中保存内容的.它的保存形式如下: — [Updated on 2023-04-06 01:50:44](https://hyp.is/Y7aUzNPaEe2Ll3N5D3nUwA/interviewguide.cn/notes/03-hunting_job/02-interview/01-04-02-STL.html) — Group: #Public

- 每个堆保存好几个元素,然后堆和堆之间有指针指向,看起来像是list和vector的结合品. — [Updated on 2023-04-06 01:50:47](https://hyp.is/Zao14NPaEe2JA8sanra6Dw/interviewguide.cn/notes/03-hunting_job/02-interview/01-04-02-STL.html) — Group: #Public

- 4.stack 底层一般用list或deque实现，封闭头部即可，不用vector的原因应该是容量大小有限制，扩容耗时 5.queue 底层一般用list或deque实现，封闭头部即可，不用vector的原因应该是容量大小有限制，扩容耗时（stack和queue其实是适配器,而不叫容器，因为是对容器的再封装） — [Updated on 2023-04-06 01:51:18](https://hyp.is/eBrE9tPaEe2Vrre1en62dQ/interviewguide.cn/notes/03-hunting_job/02-interview/01-04-02-STL.html) — Group: #Public

- 7.set 底层数据结构为红黑树，有序，不重复 8.multiset 底层数据结构为红黑树，有序，可重复 9.map 底层数据结构为红黑树，有序，不重复 10.multimap 底层数据结构为红黑树，有序，可重复 11.unordered_set 底层数据结构为hash表，无序，不重复 12.unordered_multiset 底层数据结构为hash表，无序，可重复 13.unordered_map 底层数据结构为hash表，无序，不重复 14.unordered_multimap 底层数据结构为hash表，无序，可重复 — [Updated on 2023-04-06 01:51:55](https://hyp.is/jc3N3tPaEe2E1K_4fyQg0g/interviewguide.cn/notes/03-hunting_job/02-interview/01-04-02-STL.html) — Group: #Public

- 初始时刻vector的capacity为0，塞入第一个元素后capacity增加为1； — [Updated on 2023-04-06 01:52:29](https://hyp.is/oiGZbtPaEe2sQ382AoiOVA/interviewguide.cn/notes/03-hunting_job/02-interview/01-04-02-STL.html) — Group: #Public

- 对比可以发现采用采用成倍方式扩容，可以保证常数的时间复杂度，而增加指定大小的容量只能达到O(n)的时间复杂度， — [Updated on 2023-04-06 01:52:55](https://hyp.is/sbxk-NPaEe2xP18RO3dKjg/interviewguide.cn/notes/03-hunting_job/02-interview/01-04-02-STL.html) — Group: #Public

- 1、尾后插入：size < capacity时，首迭代器不失效尾迭代失效（未重新分配空间），size == capacity时，所有迭代器均失效（需要重新分配空间）。 2、中间插入：中间插入：size < capacity时，首迭代器不失效但插入元素之后所有迭代器失效，size == capacity时，所有迭代器均失效。 — [Updated on 2023-04-06 01:57:13](https://hyp.is/S3WqKNPbEe2QIn-bJ9co4g/interviewguide.cn/notes/03-hunting_job/02-interview/01-04-02-STL.html) — Group: #Public

- 尾后删除：只有尾迭代失效。 中间删除：删除位置之后所有迭代失效。 — [Updated on 2023-04-06 01:57:22](https://hyp.is/UPLontPbEe2QI_vUj2G47g/interviewguide.cn/notes/03-hunting_job/02-interview/01-04-02-STL.html) — Group: #Public

- 而pop_back()与erase()等成员函数会改变容器的大小。 # — [Updated on 2023-04-06 02:04:18](https://hyp.is/SN3mOtPcEe2QKRfEIqYlIg/interviewguide.cn/notes/03-hunting_job/02-interview/01-04-02-STL.html) — Group: #Public
    - Annotation: 都不改变，swap可改变
- list是双向链表，而slist（single linked list）是单向链表，它们的主要区别在于：前者的迭代器是双向的Bidirectional iterator，后者的迭代器属于单向的Forward iterator。虽然slist的很多功能不如list灵活，但是其所耗用的空间更小，操作更快。 — [Updated on 2023-04-06 02:04:40](https://hyp.is/Vgpo7NPcEe25O-vNJM1AAg/interviewguide.cn/notes/03-hunting_job/02-interview/01-04-02-STL.html) — Group: #Public

- 考虑到效率问题，slist只提供push_front()操作，元素插入到slist后，存储的次序和输入的次序是相反的 — [Updated on 2023-04-06 02:05:06](https://hyp.is/ZZsR2tPcEe22vr8mhc5E3w/interviewguide.cn/notes/03-hunting_job/02-interview/01-04-02-STL.html) — Group: #Public

- 相比于vector的连续线型空间，list显得复杂许多，但是它的好处在于插入或删除都只作用于一个元素空间，因此list对空间的运用是十分精准的，对任何位置元素的插入和删除都是常数时间 — [Updated on 2023-04-06 02:08:26](https://hyp.is/3H5wbNPcEe22K5Ohuwjv7Q/interviewguide.cn/notes/03-hunting_job/02-interview/01-04-02-STL.html) — Group: #Public

- list与vector的另一个区别是，在插入和接合操作之后，都不会造成原迭代器失效，而vector可能因为空间重新配置导致迭代器失效。 — [Updated on 2023-04-06 02:08:51](https://hyp.is/68nrHtPcEe2JEvuQhe2yfQ/interviewguide.cn/notes/03-hunting_job/02-interview/01-04-02-STL.html) — Group: #Public

- 此外list也是一个环形链表，因此只要一个指针便能完整表现整个链表。list中node节点指针始终指向尾端的一个空白节点，因此是一种“前闭后开”的区间结构 — [Updated on 2023-04-06 02:09:07](https://hyp.is/9TaouNPcEe2JFC8cxrNTlA/interviewguide.cn/notes/03-hunting_job/02-interview/01-04-02-STL.html) — Group: #Public

- deque则是一种双向开口的连续线性空间，虽然vector也可以在头尾进行元素操作，但是其头部操作的效率十分低下（主要是涉及到整体的移动） — [Updated on 2023-04-06 02:09:43](https://hyp.is/CnulrNPdEe27W0sSTeFITQ/interviewguide.cn/notes/03-hunting_job/02-interview/01-04-02-STL.html) — Group: #Public

- deque和vector的最大差异一个是deque运行在常数时间内对头端进行元素操作，二是deque没有容量的概念，它是动态地以分段连续空间组合而成，可以随时增加一段新的空间并链接起来 — [Updated on 2023-04-06 02:09:55](https://hyp.is/EY_umNPdEe2W38tZPm7qeA/interviewguide.cn/notes/03-hunting_job/02-interview/01-04-02-STL.html) — Group: #Public

- 如果需要对deque排序，可以先将deque中的元素复制到vector中，利用sort对vector排序，再将结果复制回deque — [Updated on 2023-04-06 02:10:11](https://hyp.is/Gwt6PNPdEe2RXGeAPd_9Xg/interviewguide.cn/notes/03-hunting_job/02-interview/01-04-02-STL.html) — Group: #Public

- deque内部有一个指针指向map，map是一小块连续空间，其中的每个元素称为一个节点，node，每个node都是一个指针，指向另一段较大的连续空间，称为缓冲区，这里就是deque中实际存放数据的区域，默认大小512bytes。 — [Updated on 2023-04-06 02:10:55](https://hyp.is/NVcYBtPdEe2zZovep4w9gA/interviewguide.cn/notes/03-hunting_job/02-interview/01-04-02-STL.html) — Group: #Public

- 其所有操作都是围绕Sequence完成，而Sequence默认是deque数据结构。 — [Updated on 2023-04-06 02:12:26](https://hyp.is/a2EzMtPdEe2IZWvzHVQVGA/interviewguide.cn/notes/03-hunting_job/02-interview/01-04-02-STL.html) — Group: #Public

- stack除了默认使用deque作为其底层容器之外，也可以使用双向开口的list，只需要在初始化stack时，将list作为第二个参数即可。由于stack只能操作顶端的元素，因此其内部元素无法被访问，也不提供迭代器。 — [Updated on 2023-04-06 02:12:59](https://hyp.is/f4EwztPdEe2U4ENgawmxIw/interviewguide.cn/notes/03-hunting_job/02-interview/01-04-02-STL.html) — Group: #Public

- 从queue的数据结构可以看出，其所有操作都也都是是围绕Sequence完成，Sequence默认也是deque数据结构。 — [Updated on 2023-04-06 02:13:12](https://hyp.is/hzz6ANPdEe2JFl_cyFtQYw/interviewguide.cn/notes/03-hunting_job/02-interview/01-04-02-STL.html) — Group: #Public

- 同样，queue也可以使用list作为底层容器，不具有遍历功能，没有迭代器。 — [Updated on 2023-04-06 02:13:18](https://hyp.is/itLzzNPdEe2RgB-QT1I9oQ/interviewguide.cn/notes/03-hunting_job/02-interview/01-04-02-STL.html) — Group: #Public

- binary heap本质是一种complete binary tree（完全二叉树），整棵binary tree除了最底层的叶节点之外，都是填满的，但是叶节点从左到右不会出现空隙，如下图所示就是一颗完全二叉树 — [Updated on 2023-04-06 02:13:54](https://hyp.is/oCHMvNPdEe2RgZ-t2OPztQ/interviewguide.cn/notes/03-hunting_job/02-interview/01-04-02-STL.html) — Group: #Public

- 由于完全二叉树的性质，新插入的元素一定是位于树的最底层作为叶子节点，并填补由左至右的第一个空格。事实上，在刚执行插入操作时，新元素位于底层vector的end()处，之后是一个称为percolate up（上溯）的过程 — [Updated on 2023-04-06 02:17:13](https://hyp.is/Fr3uRtPeEe26ZIOSVFJxTQ/interviewguide.cn/notes/03-hunting_job/02-interview/01-04-02-STL.html) — Group: #Public

- priority_queue，优先队列，是一个拥有权值观念的queue，它跟queue一样是顶部入口，底部出口，在插入元素时，元素并非按照插入次序排列，它会自动根据权值（通常是元素的实值）排列，权值最高，排在最前面， — [Updated on 2023-04-06 02:23:54](https://hyp.is/BZAiHtPfEe2TL_cSuwv-8A/interviewguide.cn/notes/03-hunting_job/02-interview/01-04-02-STL.html) — Group: #Public

- priority_queue使用一个max-heap完成，底层容器使用的是一般为vector为底层容器，堆heap为处理规则来管理底层容器实现 。priority_queue的这种实现机制导致其不被归为容器，而是一种容器配接器。 — [Updated on 2023-04-06 02:24:13](https://hyp.is/EM6XeNPfEe22MNPAeLnV5g/interviewguide.cn/notes/03-hunting_job/02-interview/01-04-02-STL.html) — Group: #Public

- STL中的容器可分为序列式容器（sequence）和关联式容器（associative），set属于关联式容器。 set的特性是，所有元素都会根据元素的值自动被排序（默认升序），set元素的键值就是实值，实值就是键值，set不允许有两个相同的键值 set不允许迭代器修改元素的值，其迭代器是一种constance iterators — [Updated on 2023-04-06 02:24:42](https://hyp.is/IjUqhtPfEe2JIrvXLri3eQ/interviewguide.cn/notes/03-hunting_job/02-interview/01-04-02-STL.html) — Group: #Public

- 标准的STL set以RB-tree（红黑树）作为底层机制，几乎所有的set操作行为都是转调用RB-tree的操作行为，这里补充一下红黑树的特性： 每个节点不是红色就是黑色 根结点为黑色 如果节点为红色，其子节点必为黑 任一节点至（NULL）树尾端的任何路径，所含的黑节点数量必相同 — [Updated on 2023-04-06 02:24:53](https://hyp.is/KQwGNtPfEe2TQ1-yymK0ZA/interviewguide.cn/notes/03-hunting_job/02-interview/01-04-02-STL.html) — Group: #Public

- 关联式容器尽量使用其自身提供的find()函数查找指定的元素，效率更高，因为STL提供的find()函数是一种顺序搜索算法。 — [Updated on 2023-04-06 02:25:11](https://hyp.is/M70mZNPfEe2QQR81SNhC9A/interviewguide.cn/notes/03-hunting_job/02-interview/01-04-02-STL.html) — Group: #Public

- map的特性是所有元素会根据键值进行自动排序。map中所有的元素都是pair，拥有键值(key)和实值(value)两个部分，并且不允许元素有相同的key 一旦map的key确定了，那么是无法修改的，但是可以修改这个key对应的value，因此map的迭代器既不是constant iterator，也不是mutable iterator 标准STL map的底层机制是RB-tree（红黑树），另一种以hash table为底层机制实现的称为hash_map。 — [Updated on 2023-04-06 02:25:34](https://hyp.is/QQpBqNPfEe2xowvlEXH-Rg/interviewguide.cn/notes/03-hunting_job/02-interview/01-04-02-STL.html) — Group: #Public

- set的key和value其实是一样的了。其实他保存的是两份元素，而不是只保存一份元素 — [Updated on 2023-04-06 02:28:10](https://hyp.is/nngPUNPfEe2LswuM-J5dmA/interviewguide.cn/notes/03-hunting_job/02-interview/01-04-02-STL.html) — Group: #Public

- map则提供两种数据类型的接口，分别放在key和value的位置上，他的比较function采用的是红黑树的comparefunction（），保存的确实是两份元素。 — [Updated on 2023-04-06 02:28:17](https://hyp.is/okdzKNPfEe2_O5sZtxHYHg/interviewguide.cn/notes/03-hunting_job/02-interview/01-04-02-STL.html) — Group: #Public

- 他们两个的insert都是采用红黑树的insert_unique() 独一无二的插入 。 — [Updated on 2023-04-06 02:28:22](https://hyp.is/pXYAMtPfEe2yCIugoT63hQ/interviewguide.cn/notes/03-hunting_job/02-interview/01-04-02-STL.html) — Group: #Public

- multimap和map的唯一区别就是：multimap调用的是红黑树的insert_equal(),可以重复插入而map调用的则是独一无二的插入insert_unique()，multiset和set也一样，底层实现都是一样的，只是在插入的时候调用的方法不一样。 — [Updated on 2023-04-06 02:28:34](https://hyp.is/rLBCQNPfEe20GQ8yVadX9A/interviewguide.cn/notes/03-hunting_job/02-interview/01-04-02-STL.html) — Group: #Public

- 1、它是二叉排序树（继承二叉排序树特显）： 若左子树不空，则左子树上所有结点的值均小于或等于它的根结点的值。 若右子树不空，则右子树上所有结点的值均大于或等于它的根结点的值。 左、右子树也分别为二叉排序树。 2、它满足如下几点要求： 树中所有节点非红即黑。 根节点必为黑节点。 红节点的子节点必为黑（黑节点子节点可为黑）。 从根到NULL的任何路径上黑结点数相同。 3、查找时间一定可以控制在O(logn)。 — [Updated on 2023-04-06 02:29:06](https://hyp.is/v7qSRtPfEe2RdK9DseqMVw/interviewguide.cn/notes/03-hunting_job/02-interview/01-04-02-STL.html) — Group: #Public

- map支持键值的自动排序，底层机制是红黑树，红黑树的查询和维护时间复杂度均为$O(logn)$，但是空间占用比较大，因为每个节点要保持父节点、孩子节点及颜色的信息 — [Updated on 2023-04-06 02:29:34](https://hyp.is/0FNqftPfEe2E-meRYclIiw/interviewguide.cn/notes/03-hunting_job/02-interview/01-04-02-STL.html) — Group: #Public

- unordered_map是C++ 11新添加的容器，底层机制是哈希表，通过hash函数计算元素位置，其查询时间复杂度为O(1)，维护时间与bucket桶所维护的list长度有关，但是建立hash表耗时较大 从两者的底层机制和特点可以看出：map适用于有序数据的应用场景，unordered_map适用于高效查询的应用场景 — [Updated on 2023-04-06 02:29:46](https://hyp.is/16yXHtPfEe2OpaPVCoLrdA/interviewguide.cn/notes/03-hunting_job/02-interview/01-04-02-STL.html) — Group: #Public

- 线性探测 使用hash函数计算出的位置如果已经有元素占用了，则向后依次寻找，找到表尾则回到表头，直到找到一个空位 — [Updated on 2023-04-06 02:29:56](https://hyp.is/3YA7btPfEe2IdKsKon1r_A/interviewguide.cn/notes/03-hunting_job/02-interview/01-04-02-STL.html) — Group: #Public

- 开链 每个表格维护一个list，如果hash函数计算出的格子相同，则按顺序存在这个list中 — [Updated on 2023-04-06 02:29:59](https://hyp.is/32ryrNPfEe22hLtN_NW8TA/interviewguide.cn/notes/03-hunting_job/02-interview/01-04-02-STL.html) — Group: #Public

- 再散列 发生冲突时使用另一种hash函数再计算一个地址，直到不冲突 — [Updated on 2023-04-06 02:30:02](https://hyp.is/4M-FhtPfEe22yyu5YJIZWg/interviewguide.cn/notes/03-hunting_job/02-interview/01-04-02-STL.html) — Group: #Public

- 二次探测 使用hash函数计算出的位置如果已经有元素占用了，按照$1^2$、$2^2$、$3^2$...的步长依次寻找，如果步长是随机数序列，则称之为伪随机探测 — [Updated on 2023-04-06 02:30:07](https://hyp.is/4-lQYtPfEe2TOHeXlUhPyg/interviewguide.cn/notes/03-hunting_job/02-interview/01-04-02-STL.html) — Group: #Public

- 公共溢出区 一旦hash函数计算的结果相同，就放入公共溢出区 — [Updated on 2023-04-06 02:30:10](https://hyp.is/5hXjANPfEe26dxe1Y7FfXQ/interviewguide.cn/notes/03-hunting_job/02-interview/01-04-02-STL.html) — Group: #Public
