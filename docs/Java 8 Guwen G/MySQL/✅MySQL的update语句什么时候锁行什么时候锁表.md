# 典型回答


首先，在我们常用的Innodb引擎上，默认的加锁是行级锁（当然还有gap lock 和 next key lock），虽然也有可能会进行表锁，但是Innodb会尽可能的避免锁表。只有在极特殊情况下，比如没有用到索引，需要全表扫描时，才<u>有可能</u>会用到表级锁，但是也是有可能，并不代表着一定会加表锁。



另外，在MyISAM 存储引擎上，update语句会默认使用表锁的。



还有，如果我们显示的要进行表锁了，如` LOCK TABLES  `那么也会进行表锁，包括Innodb。



# 扩展知识


### Innodb的行级锁机制


InnoDB 使用行级锁（Row-level Locking）作为默认的锁定机制。执行 `UPDATE` 语句时，InnoDB 会根据所更新的记录的索引来锁定特定的行。



如果 `UPDATE` 语句中使用了有效的索引，并且查询条件能够定位到具体的行，InnoDB 会仅锁定匹配条件的行，而不是整个表。这是最常见的情况。



如果查询条件没有明确指定索引，虽然InnoDB 可能会使用全表扫描，但也不意味着Innodb就会锁全表，它仍然会倾向于使用行锁，对查找到的每一行进行锁定。





### 行级锁&表级锁


[✅InnoDB中的表级锁、页级锁、行级锁？](https://www.yuque.com/hollis666/qyhor6/vef33zs32vyylktv)





### 锁的到底是什么？


[✅MySQL的行级锁锁的到底是什么？](https://www.yuque.com/hollis666/qyhor6/kfygzw)

