# 面试者背景


23年毕业，项目是电商运营平台，主要技术mysql，mq，redis，负责过账单模块，



## 面试过程


:::warning
拉取订单数据，怎么保证订单不重复？insert or update ?

insert or update的SQL怎么写的？insert into on duplicate key update 的限制是什么？有影响吗？会有额外的加锁吗？

每天的订单量有多少？

项目的难点是什么？订单匹配，正则匹配。正则表达式判断字符串是不是邮箱怎么写？

MQ使用场景？异步导出，发MQ的应用和接MQ是同一个应用吗？不用MQ不行吗？线程池不行吗？

Rabbitmq的整体架构介绍一下。Exchange起到的作用是什么？转发路由。

RabbitMQ消息的一次生成&消费的流程是怎么样的？

死信队列用过吗？死信队列的消息需要走exchange吗？死信队列可以做什么？延迟消息。

Rabbitmq和KAFKA区别？kafka一定不丢消息吗？rabbitmq能保证不丢吗？

线程池用过吗？线程池都有哪些类型？线程池的核心参数，拒绝策略有哪些，如何选择？

线程池核心和最大线程数如何设定的？向线程池提交一个任务的处理流程是怎么样的？

Java中线程有几个状态？waiting和timed wating区别。什么时候会进入timed waiting

线程的start方法和run方法区别是什么？                            

如何在主线程中等子线程执行完？wait？

Completablefuture什么地方用到了？性能提升的原因是什么？编排。

Completablefuture实现原理？ForkJoinPool是啥？

AOP切面实现多数据源切换介绍一下？

金额的存储是怎么存的，decimal多少位？如何判断两个BigDecimal是否相等，为啥不用equals方法？

Bigdecimal如何做四舍五入？bigdecimal传给前端出现科学计数法怎么回事？bigdecimal转字符串？toString？为啥不用double和float？为啥他们会丢失精度？用double存整数有问题吗？

为啥有了基本类型还需要包装类？double Double 

缓存的穿透、击穿和雪崩？bloom过滤器。

分布式锁加锁失败后怎么做的等待？

:::

# 题目解析


:::color4
拉取订单数据，怎么保证订单不重复？insert or update ?

insert or update的SQL怎么写的？insert into on duplicate key update 的限制是什么？有影响吗？会有额外的加锁吗？

:::



因为他的项目中提到了做了订单拉取，所以问了下如何防止重复，本质上还是一个幂等问题。

涉及到insert or update的操作，就深入问一下具体用法和实现



[✅如何解决接口幂等的问题？](https://www.yuque.com/hollis666/qyhor6/gz2qwl)



[✅SQL语句如何实现insertOrUpdate的功能？](https://www.yuque.com/hollis666/qyhor6/gal4lxk8ug9g2bwk)





:::color4
项目的难点是什么？订单匹配，正则匹配。正则表达式判断字符串是不是邮箱怎么写？

:::



提到自己项目中写了很多正则表达式，那就随便问一个正则的问题。



邮箱正则验证：`^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+. [a-zA-Z]{2,}$`



:::color4
MQ使用场景？异步导出，发MQ的应用和接MQ是同一个应用吗？不用MQ不行吗？线程池不行吗？

:::



这里是候选人的项目对MQ做了个自发自收，实现一个异步的场景，但是聊下来其实自发自收消息的必要性并没有特别大，完全可以用线程池来做异步，所以项目中的一些方案，需要了解前因后果，要知道为啥一定要这个方案，或者知道这个方案还有什么不足以及替代方案。





:::color4
Rabbitmq的整体架构介绍一下。Exchange起到的作用是什么？转发路由。

RabbitMQ消息的一次生成&消费的流程是怎么样的？

死信队列用过吗？死信队列的消息需要走exchange吗？死信队列可以做什么？延迟消息。

Rabbitmq和KAFKA区别？kafka一定不丢消息吗？rabbitmq能保证不丢吗？

:::



[✅rabbitMQ的整体架构是怎么样的？](https://www.yuque.com/hollis666/qyhor6/qh56y0u8fs2gom42)



[✅RabbitMQ是怎么做消息分发的？](https://www.yuque.com/hollis666/qyhor6/qdmqppwgypsifot5)



[✅什么是RabbitMQ的死信队列？](https://www.yuque.com/hollis666/qyhor6/rd0ah4r97wevzmcw)



[✅RabbitMQ如何保证消息不丢](https://www.yuque.com/hollis666/qyhor6/ku3fxiie005axgrz)



:::color4
线程池用过吗？线程池都有哪些类型？线程池的核心参数，拒绝策略有哪些，如何选择？

线程池核心和最大线程数如何设定的？向线程池提交一个任务的处理流程是怎么样的？

:::



[✅线程池的拒绝策略有哪些？](https://www.yuque.com/hollis666/qyhor6/gfoppg6a3stefkig)



[✅什么是线程池，如何实现的？](https://www.yuque.com/hollis666/qyhor6/fb5th6)



:::color4
Java中线程有几个状态？waiting和timed wating区别。什么时候会进入timed waiting

线程的start方法和run方法区别是什么？                            

如何在主线程中等子线程执行完？wait？

:::



[✅线程有几种状态，状态之间的流转是怎样的？](https://www.yuque.com/hollis666/qyhor6/rt6e6b)



[✅run/start、wait/sleep、notify/notifyAll区别?](https://www.yuque.com/hollis666/qyhor6/bw9p42)



:::color4
Completablefuture什么地方用到了？性能提升的原因是什么？编排。

Completablefuture实现原理？ForkJoinPool是啥？

:::



[✅CompletableFuture的底层是如何实现的？](https://www.yuque.com/hollis666/qyhor6/qgrygdsu04a6vfzw)



[✅ForkJoinPool和ThreadPoolExecutor区别是什么？](https://www.yuque.com/hollis666/qyhor6/wl8s1swvh7g841be)



:::color4
AOP切面实现多数据源切换介绍一下？

:::



[✅介绍一下Spring的AOP](https://www.yuque.com/hollis666/qyhor6/nget4r5wl2imegi7)



:::color4
金额的存储是怎么存的，decimal多少位？如何判断两个BigDecimal是否相等，为啥不用equals方法？

Bigdecimal如何做四舍五入？bigdecimal传给前端出现科学计数法怎么回事？bigdecimal转字符串？toString？为啥不用double和float？为啥他们会丢失精度？用double存整数有问题吗？

为啥有了基本类型还需要包装类？double Double 

:::



[✅为什么不能用浮点数表示金额？](https://www.yuque.com/hollis666/qyhor6/vmrkz84g8c6ypu5s)



[✅为什么不能用BigDecimal的equals方法做等值比较？](https://www.yuque.com/hollis666/qyhor6/qmx8yss8tve7w73q)



[✅Java中有了基本类型为什么还需要包装类？](https://www.yuque.com/hollis666/qyhor6/xtd0s5)



:::color4
缓存的穿透、击穿和雪崩？bloom过滤器。

分布式锁加锁失败后怎么做的等待？

:::



[✅什么是缓存击穿、缓存穿透、缓存雪崩？](https://www.yuque.com/hollis666/qyhor6/abfis3)



[✅什么是布隆过滤器，实现原理是什么？](https://www.yuque.com/hollis666/qyhor6/gp9ymie1n39uavah)



分布式锁加锁失败后等待其实是阻塞锁的实现，一般redis实现的都是非阻塞锁。zookeeper更适合实现阻塞锁。



[✅如何用Zookeeper实现分布式锁？](https://www.yuque.com/hollis666/qyhor6/bdxuqt775i5zo9kz)

