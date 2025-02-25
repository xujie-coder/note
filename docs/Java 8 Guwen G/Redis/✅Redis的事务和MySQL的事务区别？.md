# 典型回答


我们知道，Redis和MySQL都宣称自己支持事务的，但是他们的事务是不一样的。或者说他们对事务的定义是有差别的。



**Redis中提供了一种简单的事务机制**，通过 `MULTI`、`EXEC`、`DISCARD` 和 `WATCH` 命令来实现的。**他可以保证一组命令按顺序执行，中间不会被其他客户端的命令打断。**



但是，**Redis 的事务不支持回滚**，一旦事务中的某条命令出错，其他命令仍会继续执行。而且Redis的事务也没有隔离级别的定义，本身是单线程的，也没啥隔离的必要了。



[✅Redis 的事务机制是怎样的？](https://www.yuque.com/hollis666/qyhor6/xxxz79)



Redis的事务的好处是，简单，轻量，不需要复杂的事务管理，之所以这么设计，是因为Redis天生就是一个缓存框架，他的目的就是为了性能。所以他放弃了数据的强一致性，而选择了优先保性能！



[✅为什么Redis不支持回滚？](https://www.yuque.com/hollis666/qyhor6/gt0qpqluiwb7bg70)





MySQL 的事务机制是遵循标准的 ACID 的，所谓ACID，就是

+ **原子性（Atomicity）**：事务内的操作要么全部成功，要么全部失败并回滚。
+ **一致性（Consistency）**：事务执行后，数据库从一个一致状态转换到另一个一致状态。
+ **隔离性（Isolation）**：事务之间相互独立，避免相互影响。
+ **持久性（Durability）**：事务提交后，数据持久化存储。



[✅什么是数据库事务机制？](https://www.yuque.com/hollis666/qyhor6/bvelw3)



MySQL 支持多种事务隔离级别，控制事务间的可见性和数据一致性：

+ **READ UNCOMMITTED**：事务可以看到其他未提交事务的数据（会出现脏读）。
+ **READ COMMITTED**：只能看到已提交事务的数据（可以防止脏读，但是会出现不可重复读）。
+ **REPEATABLE READ**：同一事务内的查询结果一致（防止不可重复读，但是会出现幻读，MySQL 默认）。
+ **SERIALIZABLE**：最高隔离级别，事务完全串行化执行（防止所有读现象）。



[✅MySQL中的事务隔离级别？](https://www.yuque.com/hollis666/qyhor6/ytxaew)



MySQL 的事务依赖锁机制来实现并发控制。MySQL 使用 Undo Log 实现事务的回滚，Redo Log 提供崩溃恢复能力，保证数据一致性。



所以，MySQL作为一种专门用来持久化的数据库，他有很好的数据一致性保障，并且有强大的并发支持，但是它的性能不如Redis。





