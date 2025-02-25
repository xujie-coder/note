# 典型回答


在回答这个问题之前，需要有一定的前提储备知识，要不然可能你连这个问题都看不懂。



<font style="color:#DF2A3F;">储备知识1</font>：MySQL数据库中的每行记录中，除了保存了我们自己定义的一些字段以外，还有一些重要的隐式字段的：db_row_id（隐藏主键，如果我们没有给这个表创建主键，那么会以这个字段来创建聚簇索引。）、 db_trx_id（对这条记录做了最新一次修改的事务的ID）、以及 db_roll_ptr（回滚指针，指向这条记录的上一个版本，其实他指向的就是Undo Log中的上一个版本的快照的地址。 ）



<font style="color:#DF2A3F;">储备知识2</font>：在MVCC中，隐藏字段中的db_roll_ptr用来构建版本链，db_trx_id也是一个重要的用来判断快照可见性的一个字段。



[✅如何理解MVCC？](https://www.yuque.com/hollis666/qyhor6/wgu1u6)



<font style="color:#DF2A3F;">储备知识3</font>：在Innodb 的聚簇索引（主键索引）中，叶子节点上保存的是整行记录。而在非聚簇索引（二级索引）中，叶子节点上值保存了主键的信息。



[✅什么是聚簇索引和非聚簇索引？](https://www.yuque.com/hollis666/qyhor6/le8gbo472cpxv63z)



<font style="color:#DF2A3F;">储备知识4</font>：在一条查询语句中，如果能用到索引覆盖，则就会直接用二级索引进行检索，并不会回表。



[✅什么是回表，怎么减少回表的次数？](https://www.yuque.com/hollis666/qyhor6/vr22wd)





确保你针对以上知识点都了解了之后，那么问题来了。



<font style="background-color:#FBDE28;">如果某个查询语句的查询字段都包含在二级索引中，那么就会走索引覆盖，不用回表去读取聚簇索引的页记录。但是，版本链的头结点在聚簇索引中，不在二级索引中，通过二级索引的记录无法直接找到版本链。在这种情况下如何使用MVCC？</font>

<font style="background-color:#FBDE28;"></font>

这个问题挺有意思的，确实，在你懂了以上几个知识点之后，可能就会陷入这个疑惑当中，那么具体是怎么回事呢？我在MySQL 官网中（[https://dev.mysql.com/doc/refman/8.4/en/optimizing-innodb-transaction-management.html](https://dev.mysql.com/doc/refman/8.4/en/optimizing-innodb-transaction-management.html) ），到了一个以下关于这个问题的描述：



![](https://www.hollischuang.com/wp-content/uploads/2024/08/17229262951707.jpg)



翻译一下就是：**如果发现二级索引页有一个 PAGE_MAX_TRX_ID 太新的，或者如果二级索引中的记录被删除标记， InnoDB可能需要使用聚集索引来查找记录。**



也就是说，**索引覆盖并不是用到了联合索引就一定会走的**！在以上这种情况下，会回表，通过覆盖索引进行查询。上面的这个解释不是很清楚，我把过程在展开说一下：



在二级索引中，用一个额外的名为page_max_trx_id的变量来记录表示修改过该页的最大事务id。



如果当前查询得到的read_view的 `up_limit_id > page_max_trx_id`，说明在创建read_view时，最后一次更新二级索引的事务已经提交了，那就意味着二级索引里的提交对于当前查询都应该是可见的。这时候如果这个二级索引的记录没有被删除，那么就可以直接走索引覆盖查询。



> up_limit_id来自于readview：
>

[✅什么是ReadView，什么样的ReadView可见？](https://www.yuque.com/hollis666/qyhor6/gq6em9bet37p4f77)



否则，就意味着数据可能被修改了，不能直接查询，而需要回表，通过聚簇索引进行查询。使用聚簇索引的时候，叶子节点行记录中设计包含了版本链的，就可以用到MVCC了。

