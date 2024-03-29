
## 1、文件索引和数据库索引为什么使用B+树?
文件与数据库都是需要较大的存储，也就是说，它们都不可能全部存储在内存中，故需要存储到磁盘上。而所谓索引，则为了数据的快速定位与查找，那么索引的结构组织要尽量减少查找过程中磁盘I/O的存取次数，因此B+树相比B树更为合适。数据库系统巧妙利用了局部性原理与磁盘预读原理，将一个节点的大小设为等于一个页，这样每个节点只需要一次I/O就可以完全载入，而红黑树这种结构，高度明显要深的多，并且由于逻辑上很近的节点(父子)物理上可能很远，无法利用局部性。 

最重要的是，B+树还有一个最大的好处：方便扫库。 B树必须用中序遍历的方法按序扫库，而B+树直接从叶子结点挨个扫一遍就完了，B+树支持range-query非常方便，而B树不支持，这是数据库选用B+树的最主要原因。 

B+树查找效率更加稳定，B树有可能在中间节点找到数据，稳定性不够。 

B+tree的磁盘读写代价更低：B+tree的内部结点并没有指向关键字具体信息的指针(红色部分)，因此其内部结点相对B 树更小。如果把所有同一内部结点的关键字存放在同一块盘中，那么盘块所能容纳的关键字数量也越多。一次性读入内存中的需要查找的关键字也就越多，相对来说IO读写次数也就降低了； 
B+tree的查询效率更加稳定：由于内部结点并不是最终指向文件内容的结点，而只是叶子结点中关键字的索引，所以，任何关键字的查找必须走一条从根结点到叶子结点的路。所有关键字查询的路径长度相同，导致每一个数据的查询效率相当；

## 2、MySQL常见的存储引擎InnoDB、MyISAM的区别？适用场景分别是？ 
- 1）事务：MyISAM不支持，InnoDB支持 
- 2）锁级别： MyISAM 表级锁，InnoDB 行级锁及外键约束 
- 3）MyISAM存储表的总行数；InnoDB不存储总行数； 
- 4）MyISAM采用非聚集索引，B+树叶子存储指向数据文件的指针。InnoDB主键索引采用聚集索引，B+树叶子存储数据 
- 适用场景： MyISAM适合： 插入不频繁，查询非常频繁，如果执行大量的SELECT，MyISAM是更好的选择， 没有事务。 InnoDB适合： 可靠性要求比较高，或者要求事务； 表更新和查询都相当的频繁， 大量的INSERT或UPDATE

## 3、事务四大特性（ACID）原子性、一致性、隔离性、持久性？ 
第一种回答 原子性：一个事务（transaction）中的所有操作，要么全部完成，要么全部不完成，不会结束在中间某个环节。 。事务在执行过程中发生错误，会被恢复（Rollback）到事务开始前的状态，就像这个事务从来没有执行过一样。 一致性：在事务开始之前和事务结束以后，数据库的完整性没有被破坏。这表示写入的资料必须完全符合所有的预设规则，这包含资料的精确度、串联性以及后续数据库可以自发性地完成预定的工作。 隔离性：数据库允许多个并发事务同时对其数据进行读写和修改的能力，隔离性可以防止多个事务并发执行时由于交叉执行而导致数据的不一致。事务隔离分为不同级别，包括读未提交（Read uncommitted）、读提交（read committed）、可重复读（repeatable read）和串行化（Serializable）。 持久性：事务处理结束后，对数据的修改就是永久的，即便系统故障也不会丢失。 # 第二种回答 原子性（Atomicity） 原子性是指事务包含的所有操作要么全部成功，要么全部失败回滚，因此事务的操作如果成功就必须要完全应用到数据库，如果操作失败则不能对数据库有任何影响。 一致性（Consistency） 事务开始前和结束后，数据库的完整性约束没有被破坏。比如A向B转账，不可能A扣了钱，B却没收到。 隔离性（Isolation） 隔离性是当多个用户并发访问数据库时，比如操作同一张表时，数据库为每一个用户开启的事务，不能被其他事务的操作所干扰，多个并发事务之间要相互隔离。 同一时间，只允许一个事务请求同一数据，不同的事务之间彼此没有任何干扰。比如A正在从一张银行卡中取钱，在A取钱的过程结束前，B不能向这张卡转账。 关于事务的隔离性数据库提供了多种隔离级别，稍后会介绍到。   持久性（Durability） 持久性是指一个事务一旦被提交了，那么对数据库中的数据的改变就是永久性的，即便是在数据库系统遇到故障的情况下也不会丢失提交事务的操作。

## 4、为什么MySQL 采用 B+ 树作为索引？
B+ 树的非叶子节点不存放实际的记录数据，仅存放索引，因此数据量相同的情况下，相比存储即存索引又存记录的 B 树，B+树的非叶子节点可以存放更多的索引，因此 B+ 树可以比 B 树更「矮胖」，查询底层节点的磁盘 I/O次数会更少。 B+ 树有大量的冗余节点（所有非叶子节点都是冗余索引），这些冗余索引让 B+ 树在插入、删除的效率都更高，比如删除根节点的时候，不会像 B 树那样会发生复杂的树的变化； B+ 树叶子节点之间用链表连接了起来，有利于范围查询，而 B 树要实现范围查询，因此只能通过树的遍历来完成范围查询，这会涉及多个节点的磁盘 I/O 操作，范围查询效率不如 B+ 树。
- B+Tree 是一种多叉树，叶子节点才存放数据，非叶子节点只存放索引，而且每个节点里的数据是按主键顺序存放的。每一层父节点的索引值都会出现在下层子节点的索引值中，因此在叶子节点中，包括了所有的索引值信息，并且每一个叶子节点都有两个指针，分别指向下一个叶子节点和上一个叶子节点，形成一个双向链表。 

- B+Tree 相比于 B 树和二叉树来说，最大的优势在于查询效率很高，因为即使在数据量很大的情况，查询一个数据的磁盘 I/O 依然维持在 3-4次 

- 主键索引的 B+Tree 和二级索引的 B+Tree 区别如下： 主键索引的 B+Tree 的叶子节点存放的是实际数据，所有完整的用户记录都存放在主键索引的 B+Tree 的叶子节点里； 二级索引的 B+Tree 的叶子节点存放的是主键值，而不是实际数据

- B+Tree 只在叶子节点存储数据，而 B 树 的非叶子节点也要存储数据，所以 B+Tree 的单个节点的数据量更小，在相同的磁盘 I/O 次数下，就能查询更多的节点。 另外，B+Tree 叶子节点采用的是双链表连接，适合 MySQL 中常见的基于范围的顺序查找，而 B 树无法做到这一点。

- Hash 在做等值查询的时候效率贼快，搜索复杂度为 O(1)。 但是 Hash 表不适合做范围查询，它更适合做等值的查询，这也是 B+Tree 索引要比 Hash 表索引有着更广泛的适用场景的原因。 

- 索引最大的好处是提高查询速度，但是索引也是有缺点的，比如： 需要占用物理空间，数量越大，占用空间越大； 创建索引和维护索引要耗费时间，这种时间随着数据量的增加而增大； 会降低表的增删改的效率，因为每次增删改索引，B+ 树为了维护索引有序性，都需要进行动态维护。 所以，索引不是万能钥匙，它也是根据场景来使用的。


## 5、视图的作用是什么？可以更改吗？ 
- 视图是虚拟的表，与包含数据的表不一样，视图只包含使用时动态检索数据的查询；不包含任何列或数据。使用视图可以简化复杂的 sql 操作，隐藏具体的细节，保护数据；视图创建后，可以使用与表相同的方式利用它们。 视图不能被索引，也不能有关联的触发器或默认值，如果视图本身内有order by 则对视图再次order by将被覆盖。


## 6、假如你所在的公司选择MySQL数据库作数据存储，一天五万条以上的增量，预计运维三年，你有哪些优化手段？ 
- 设计良好的数据库结构，允许部分数据冗余，尽量避免join查询，提高效率。 选择合适的表字段数据类型和存储引擎，适当的添加索引。 MySQL库主从读写分离。 找规律分表，减少单表中的数据量提高查询速度。 添加缓存机制，比如Memcached，Apc等。 不经常改动的页面，生成静态页面。 书写高效率的SQL。比如 SELECT * FROM TABEL 改为 SELECT field_1, field_2, field_3 FROM TABLE。 

## 7、千万级别数据查询优化

### Mysql语句优化
1. 使用EXPLAIN关键词检查SQL。EXPLAIN可以帮你分析你的查询语句或是表结构的性能瓶颈，就得EXPLAIN 的查询结果还会告诉你你的索引主键被如何利用的，你的数据表是如何被搜索和排序的，是否有全表扫描等；
2. 查询的条件尽量使用索引字段，如某一个表有多个条件，就尽量使用复合索引查询，复合索引使用要注意字段的先后顺序。
3. 多表关联尽量用join，减少子查询的使用。表的关联字段如果能用主键就用主键，也就是尽可能的使用索引字段。如果关联字段不是索引字段可以根据情况考虑添加索引。
4. 尽量使用limit进行分页批量查询，不要一次全部获取。
5. 绝对避免select *的使用，尽量select具体需要的字段，减少不必要字段的查询；
6. 尽量将or 转换为 union all。
7. 尽量避免使用is null或is not null。
8. 要注意like的使用，前模糊和全模糊不会走索引。
9. Where后的查询字段尽量减少使用函数，因为函数会造成索引失效。
10. 避免使用不等于（!=），因为它不会使用索引。
11. 用exists代替in，not exists代替not in，效率会更好；
12. 避免使用HAVING子句, HAVING 只会在检索出所有记录之后才对结果集进行过滤，这个处理需要排序，总计等操作。如果能通过WHERE子句限制记录的数目,那就能减少这方面的开销。
13. 千万不要 ORDER BY RAND()

### 数据库优化

#### 业务方向分库分表

#### 水平方向针对数据读写分离
主库写，从库读，主库写完同步到从库
- 分库：是为了解决数据库连接资源不足问题，和磁盘IO的性能瓶颈问题。
- 分表：是为了解决单表数据量太大，sql语句查询数据时，即使走了索引也非常耗时问题。此外还可以解决消耗cpu资源问题。
- 分库分表：可以解决 数据库连接资源不足、磁盘IO的性能瓶颈、检索数据耗时 和 消耗cpu资源等问题。

  
  
作者：苏三说技术  
链接：https://www.zhihu.com/question/439988021/answer/2436380280  
来源：知乎  
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
