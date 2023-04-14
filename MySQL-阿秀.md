---
doc_type: hypothesis-highlights
url: >-
  https://interviewguide.cn/notes/03-hunting_job/02-interview/04-01-01-MySQL.html
---

# MySQL01-20

## Metadata
- Author: [interviewguide.cn]()
- Title: MySQL01-20
- Reference: https://interviewguide.cn/notes/03-hunting_job/02-interview/04-01-01-MySQL.html
- Category: #article

## Page Notes
## Highlights
- 12、文件索引和数据库索引为什么使用B+树?（第9个问题的详细回答） 文件与数据库都是需要较大的存储，也就是说，它们都不可能全部存储在内存中，故需要存储到磁盘上。而所谓索引，则为了数据的快速定位与查找，那么索引的结构组织要尽量减少查找过程中磁盘I/O的存取次数，因此B+树相比B树更为合适。数据库系统巧妙利用了局部性原理与磁盘预读原理，将一个节点的大小设为等于一个页，这样每个节点只需要一次I/O就可以完全载入，而红黑树这种结构，高度明显要深的多，并且由于逻辑上很近的节点(父子)物理上可能很远，无法利用局部性。 最重要的是，B+树还有一个最大的好处：方便扫库。 B树必须用中序遍历的方法按序扫库，而B+树直接从叶子结点挨个扫一遍就完了，B+树支持range-query非常方便，而B树不支持，这是数据库选用B+树的最主要原因。 B+树查找效率更加稳定，B树有可能在中间节点找到数据，稳定性不够。 B+tree的磁盘读写代价更低：B+tree的内部结点并没有指向关键字具体信息的指针(红色部分)，因此其内部结点相对B 树更小。如果把所有同一内部结点的关键字存放在同一块盘中，那么盘块所能容纳的关键字数量也越多。一次性读入内存中的需要查找的关键字也就越多，相对来说IO读写次数也就降低了； B+tree的查询效率更加稳定：由于内部结点并不是最终指向文件内容的结点，而只是叶子结点中关键字的索引，所以，任何关键字的查找必须走一条从根结点到叶子结点的路。所有关键字查询的路径长度相同，导致每一个数据的查询效率相当； — [Updated on 2023-03-24 09:43:44](https://hyp.is/UBWsVMnlEe24fu9NRh6bdQ/interviewguide.cn/notes/03-hunting_job/02-interview/04-01-01-MySQL.html) — Group: #Public






---
doc_type: hypothesis-highlights
url: >-
  https://interviewguide.cn/notes/03-hunting_job/02-interview/04-01-02-MySQL.html
---

# MySQL21-40

## Metadata
- Author: [interviewguide.cn]()
- Title: MySQL21-40
- Reference: https://interviewguide.cn/notes/03-hunting_job/02-interview/04-01-02-MySQL.html
- Category: #article

## Page Notes
## Highlights
1. 30、MySQL常见的存储引擎InnoDB、MyISAM的区别？适用场景分别是？ 
- 1）事务：MyISAM不支持，InnoDB支持 
- 2）锁级别： MyISAM 表级锁，InnoDB 行级锁及外键约束 
- 3）MyISAM存储表的总行数；InnoDB不存储总行数； 
- 4）MyISAM采用非聚集索引，B+树叶子存储指向数据文件的指针。InnoDB主键索引采用聚集索引，B+树叶子存储数据 
- 适用场景： MyISAM适合： 插入不频繁，查询非常频繁，如果执行大量的SELECT，MyISAM是更好的选择， 没有事务。 InnoDB适合： 可靠性要求比较高，或者要求事务； 表更新和查询都相当的频繁， 大量的INSERT或UPDATE — [Updated on 2023-03-24 16:06:17](https://hyp.is/wPCFVMoaEe2tAg-4kAfujQ/interviewguide.cn/notes/03-hunting_job/02-interview/04-01-02-MySQL.html) — Group: #Public

- 31、事务四大特性（ACID）原子性、一致性、隔离性、持久性？ # 第一种回答 原子性：一个事务（transaction）中的所有操作，要么全部完成，要么全部不完成，不会结束在中间某个环节。 。事务在执行过程中发生错误，会被恢复（Rollback）到事务开始前的状态，就像这个事务从来没有执行过一样。 一致性：在事务开始之前和事务结束以后，数据库的完整性没有被破坏。这表示写入的资料必须完全符合所有的预设规则，这包含资料的精确度、串联性以及后续数据库可以自发性地完成预定的工作。 隔离性：数据库允许多个并发事务同时对其数据进行读写和修改的能力，隔离性可以防止多个事务并发执行时由于交叉执行而导致数据的不一致。事务隔离分为不同级别，包括读未提交（Read uncommitted）、读提交（read committed）、可重复读（repeatable read）和串行化（Serializable）。 持久性：事务处理结束后，对数据的修改就是永久的，即便系统故障也不会丢失。 # 第二种回答 原子性（Atomicity） 原子性是指事务包含的所有操作要么全部成功，要么全部失败回滚，因此事务的操作如果成功就必须要完全应用到数据库，如果操作失败则不能对数据库有任何影响。 一致性（Consistency） 事务开始前和结束后，数据库的完整性约束没有被破坏。比如A向B转账，不可能A扣了钱，B却没收到。 隔离性（Isolation） 隔离性是当多个用户并发访问数据库时，比如操作同一张表时，数据库为每一个用户开启的事务，不能被其他事务的操作所干扰，多个并发事务之间要相互隔离。 同一时间，只允许一个事务请求同一数据，不同的事务之间彼此没有任何干扰。比如A正在从一张银行卡中取钱，在A取钱的过程结束前，B不能向这张卡转账。 关于事务的隔离性数据库提供了多种隔离级别，稍后会介绍到。   持久性（Durability） 持久性是指一个事务一旦被提交了，那么对数据库中的数据的改变就是永久性的，即便是在数据库系统遇到故障的情况下也不会丢失提交事务的操作。 — [Updated on 2023-03-24 16:07:22](https://hyp.is/59rvpsoaEe2erhee7AOppA/interviewguide.cn/notes/03-hunting_job/02-interview/04-01-02-MySQL.html) — Group: #Public




