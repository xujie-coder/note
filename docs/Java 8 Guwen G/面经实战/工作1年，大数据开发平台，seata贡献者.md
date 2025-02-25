# 面试者背景




:::warning
**今日面试者：****23****年毕业，大数据开发平台、数据安全****&****质量监控、****seata****开源贡献、**

**权限管控是自建的吗？介绍下权限管控功能设计。一个用户想要执行****SQL****，如何判断能不能执行？**

**权限模型是如何设计的？审批流是怎么做的？流程引擎。**

**不同的表的审批流程不一样，如何实现的？责任链。不同的策略如何实现的？跨****BU****、外包、等级等。**

**项目中有并发场景吗？审批通过回调并发，授权抢锁、都用了状态判断为啥还需要加分布式锁？乐观锁**

**分布式锁如何实现的？****redisson****、自定义注解****+****切面。要锁什么东西，怎么配置？****spel****表达式。怎么写**

**为啥用****redisson****而不是用****setnx****？超时时间、自动续期。****Redisson****自动续期实现原理？****watchdog****是咋实现的？定时是怎么实现的？**

**加锁的时候，****redis****不可用了咋整？降级处理。**

**什么叫做幂等？想要实现幂等需要做什么？唯一识别号、状态机。查询接口怎么实现幂等？**

**如果请求中没有一个唯一的识别号怎么做幂等？表单提交没有幂等号怎么防止重复提交？**

**项目中解决****like****无法走索引问题，展开介绍下。扫表。这个地方有必要做分库分表吗？****1-2****亿申请记录？分了多少张表？如何基于用户****id****后两位分****16****张表？后两位直接取模、不均匀怎么办？为啥不直接用用户****id****取模****16****？**

**为啥**** like %xx****不走索引？****like xx%xxx****走索引吗？为啥可以走？**

**a，****b****都有索引，****select * from table where a = xx order by b****。走哪个索引？**

**介绍一个问题排查过程，****GC****问题。。。。。****gc****次数，****dump****，大对象。报警情况。****Dump****用什么工具看的？****dump****是如何获取到的？****jmap****、**

**给****seata****提交过什么代码？****seata 2.1****的****rocketmq****事务消息方案，****rocketmq****回滚后，全局事务怎么回滚。这个方案相比****AT****有啥区别？**

**Undo_log****存在性校验是干了啥？**

**介绍下seata各种模式的特点和适用场景?xa强一致性、at本地事务先提交、undo-log回滚，tcc空回滚、事务悬挂，saga长事务。什么情况只能用TCC不能AT？**

:::

# 题目解析




:::color4
**分布式锁如何实现的？****redisson****、自定义注解****+****切面。要锁什么东西，怎么配置？****spel****表达式。怎么写**

**为啥用****redisson****而不是用****setnx****？超时时间、自动续期。****Redisson****自动续期实现原理？****watchdog****是咋实现的？定时是怎么实现的？**

**加锁的时候，redis不可用了咋整？降级处理。**

:::



[✅分布式锁有几种实现方式？](https://www.yuque.com/hollis666/qyhor6/fvnr41#CJQP3)



[✅Redis实现分布锁的时候，哪些问题需要考虑？](https://www.yuque.com/hollis666/qyhor6/zrney050xgem0voc)



[✅如何用Redisson实现分布式锁？](https://www.yuque.com/hollis666/qyhor6/gdsvngueclva39ve)



[✅Redisson的watchdog机制是怎么样的？](https://www.yuque.com/hollis666/qyhor6/fg0f0wh41g8eu5ik)



:::color4
**什么叫做幂等？想要实现幂等需要做什么？唯一识别号、状态机。查询接口怎么实现幂等？**

**如果请求中没有一个唯一的识别号怎么做幂等？表单提交没有幂等号怎么防止重复提交？**

:::



[✅如何解决接口幂等的问题？](https://www.yuque.com/hollis666/qyhor6/gz2qwl)





:::color4
**项目中解决****like****无法走索引问题，展开介绍下。扫表。这个地方有必要做分库分表吗？****1-2****亿申请记录？分了多少张表？如何基于用户****id****后两位分****16****张表？后两位直接取模、不均匀怎么办？为啥不直接用用户****id****取模****16****？**

**为啥**** like %xx****不走索引？****like xx%xxx****走索引吗？为啥可以走？**

**a，b都有索引，select * from table where a = xx order by b。走哪个索引？**

:::



[✅MySQL中like的模糊查询如何优化](https://www.yuque.com/hollis666/qyhor6/zrt2y30mhdgiremc)



[✅索引失效的问题是如何排查的，有那些种情况？](https://www.yuque.com/hollis666/qyhor6/sgkrtodriyoliden#IfVVR)



[✅a，b都有索引，select * from table where a = xx order by b。走哪个索引？](https://www.yuque.com/hollis666/qyhor6/sopm64dgvu5g2m5h)



:::color4
**介绍一个问题排查过程，GC问题。。。。。gc次数，dump，大对象。报警情况。Dump用什么工具看的？dump是如何获取到的？jmap、**

:::



[✅频繁FullGC问题排查](https://www.yuque.com/hollis666/qyhor6/iocmzc)



[✅频繁FullGC问题排查(2)](https://www.yuque.com/hollis666/qyhor6/zpkzwgx4o9g89s8x)



这种问题，一定不能上来就说你的排查过程，那样就像是背的，一定要从背景，问题的发现开始讲起来。



:::color4
**给****seata****提交过什么代码？****seata 2.1****的****rocketmq****事务消息方案，****rocketmq****回滚后，全局事务怎么回滚。这个方案相比****AT****有啥区别？**

**Undo_log****存在性校验是干了啥？**

**介绍下seata各种模式的特点和适用场景?xa强一致性、at本地事务先提交、undo-log回滚，tcc空回滚、事务悬挂，saga长事务。什么情况只能用TCC不能AT？**

:::





[✅Seata的实现原理是什么](https://www.yuque.com/hollis666/qyhor6/qro9fl9lsiinx1tu)



[✅Seata的AT模式和XA有什么区别？](https://www.yuque.com/hollis666/qyhor6/fzd9nmraf5krr4m0)



[✅Seata的4种事务模式，各自适合的场景是什么？](https://www.yuque.com/hollis666/qyhor6/cx86tg6tdhmz1dm9)

