# 索引

## 概述

* 索引是帮助Mysql高效获取数据的**排好序**的数据结构
* 如下图，在col2中如果可以变为一个二叉树，查询效率会极大地提升
* ![index_structure](/Users/huhawel/Documents/learning_note/My_Learning_Note/Database/Mysql/index_structure.jpg)
* 索引数据结构
  * 二叉树
  * 红黑树
  * Hash表
  * B-Tree

* Mysql底层索引用的B+树，属于B树的变种
  * 非叶子节点不存储数据，只存储索引（冗余），可以放进更多索引
  * 叶子节点包含索引字段
  * 叶子节点用指针连接，提高区间访问性能
  * 相对于B树，每个节点只包含索引而不包含数据，因此能指向更多节点

* 对于每一个节点，Mysql会为该节点分配一个16kb的数据页。（不能太大，否则会消耗内存）

* 也就是说一个节点可以放16kb的大小，在小数据样本下一次IO就可以实现

* 当层级为3时，可以放多少数据？

  ​	每个数据8byte，每个指针6byte，每个节点可以放：

  ​			16 kb / (8 + 6) = 1366 条

  ​	假设每个data占1kb，每个叶子节点放16个数据

  ​	1366 * 1366 * 16 = 30,000,000 条数据
  
* 表连接很难走索引，且底层计算量非常巨大，不推荐使用

### 局部性原理（locality）

空间局部性(spacial locality)：程序对数据的访问都有聚集成群的倾向

时间局部性(time locality): 程序通常倾向于访问最近访问过的磁盘空间

### 

## 哈希索引

![image-20210518000234013](/Users/huhawel/Library/Application Support/typora-user-images/image-20210518000234013.png)

* 通过hash桶和链表组织表结构，在某些情况下比B+树更高效
* 不支持范围搜寻，且要注意哈希冲突

## 联合索引

![image-20210518001637897](/Users/huhawel/Library/Application Support/typora-user-images/image-20210518001637897.png)

* 联合索引也是用B+树实现的

* 一般来说会有优先级，e.g. 先比较第一个数据，再比较第二个

* 最左前缀原则：联合索引必须要被最左端的属性触发

  * 原因：如下图，假如直接使用age无法根据name按顺序遍历B+树，索引在组织B+树时就是按顺序使用联合索引排序的，只用age无法保证name是有序的

  ![image-20210518002956514](/Users/huhawel/Library/Application Support/typora-user-images/image-20210518002956514.png)



## 存储引擎

* 我们常说的存储引擎（MyISAM、Innodedb）实际上是修饰数据库表的，而不是整个数据库

### MyISAM索引实现

![image-20210517200842175](/Users/huhawel/Library/Application Support/typora-user-images/image-20210517200842175.png)

 * MyISAM的索引文件和数据文件是分离的（非聚集）
   	* 数据文件存在.MYD文件
    * 索引文件存在.MYI文件
    * 结构存在.frm文件
 * 一般我们查找某一条信息时，Mysql遵循以下步骤：
   1. MyISAM会先到索引文件查找是否有可用索引
   2. 根据索引到MYD文件定位磁盘位置
   3. 读写IO
* MyISAM的索引为非聚集索引，既索引文件和数据文件分离

### Innodb索引实现

* 相比于MyISAM，InnodeDB只有两个数据结构存储表，分别是.frm表结构和.ibd数据及索引
* 相对于MyISAM的B+树叶节点只放了数据行的索引，Innodb叶节点放的是整一行
* 因此Innodb的索引为聚集索引，聚集索引-叶节点包含了完整的数据信息
* （面试）为什么建议InnoDB表必须建主键而且视同整型的自增主键？
  * 由于InnoDB用B+树组织整个表的数据，如果有主键就会自动用主键索引建立B+树，如果没有主键的话InnoDB会自动查找某个列（唯一），并依据此建立B+树，若没有找到，InnoDB会自己维护一个隐藏列并依据此建立InnoDB
  * 由于非自增有可能出发树的分裂和自平衡，因此一般用自增主键

![image-20210517233616952](/Users/huhawel/Library/Application Support/typora-user-images/image-20210517233616952.png)

### 聚集索引和非聚集索引

* 聚集索引：索引中键值的逻辑顺序决定了表中相应行的物理顺序，**聚簇索引**也称为**聚集索引**。
* 非聚集索引：索引中索引的逻辑顺序与磁盘上行的物理存储顺序不同。（MyISAM）
* 从效率上来说，聚集索引比非聚集索引速度更快因为数据和索引放在一起，不需要额外IO获取数据

