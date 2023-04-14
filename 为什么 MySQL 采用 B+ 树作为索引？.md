---
doc_type: hypothesis-highlights
url: 'https://xiaolincoding.com/mysql/index/why_index_chose_bpuls_tree.html'
---

# 为什么 MySQL 采用 B+ 树作为索引？

## Metadata
- Author: [xiaolincoding.com]()
- Title: 为什么 MySQL 采用 B+ 树作为索引？
- Reference: https://xiaolincoding.com/mysql/index/why_index_chose_bpuls_tree.html
- Category: #article

## Page Notes
## Highlights
- B+ 树的非叶子节点不存放实际的记录数据，仅存放索引，因此数据量相同的情况下，相比存储即存索引又存记录的 B 树，B+树的非叶子节点可以存放更多的索引，因此 B+ 树可以比 B 树更「矮胖」，查询底层节点的磁盘 I/O次数会更少。 B+ 树有大量的冗余节点（所有非叶子节点都是冗余索引），这些冗余索引让 B+ 树在插入、删除的效率都更高，比如删除根节点的时候，不会像 B 树那样会发生复杂的树的变化； B+ 树叶子节点之间用链表连接了起来，有利于范围查询，而 B 树要实现范围查询，因此只能通过树的遍历来完成范围查询，这会涉及多个节点的磁盘 I/O 操作，范围查询效率不如 B+ 树。 — [Updated on 2023-03-22 13:43:26](https://hyp.is/d2j0_sh0Ee2Sjd_3AGlWmQ/xiaolincoding.com/mysql/index/why_index_chose_bpuls_tree.html) — Group: #Public




