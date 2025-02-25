# 典型回答


[✅RocketMQ的事务消息是如何实现的？](https://www.yuque.com/hollis666/qyhor6/abxh7z)



[✅Kafka支持事务消息吗？如何实现的？](https://www.yuque.com/hollis666/qyhor6/yfzof8znomat1u6g)



以上是关于Kafka和RocketMQ的事务消息的介绍。



他们都叫事务消息，目的都是为了保证ACID，其实最重要的就是一个原子性。但是他们的目的和解决问题不同。



**Kafka的事务消息，确保的是一组发送操作要么全部成功，要么全部失败，以保证消息处理的原子性。**

****

**RocketMQ的事务消息，确保的是发送方的本地事务和消息发送以原子性方式执行，即要么都成功，要么都失败。**

****

所以，RocketMQ的事务消息，经常被用来解决分布式场景下的数据一致性问题，是分布式事务的一种常见方案。



而Kafka的事务消息，主要是用来来配合 Kafka 的幂等机制来实现 Kafka 的 Exactly Once 语义（每条消息只会被精确地传递一次：既不会多，也不会少；）。





