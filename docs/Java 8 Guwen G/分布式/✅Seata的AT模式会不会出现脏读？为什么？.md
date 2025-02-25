# 典型回答


一句话回答：会出现一种脏读情况，只不过和传统脏读不一样，传统的脏读指的是<font style="color:rgb(64, 64, 64);">读取到其他（MySQL本地事务）事务</font>**<font style="color:rgb(64, 64, 64);">未提交</font>**<font style="color:rgb(64, 64, 64);">的数据。而Seata会出现，读取到其他（分支事务）（本地事务）事务</font>**<font style="color:rgb(64, 64, 64);">已提交但可能被全局回滚</font>**<font style="color:rgb(64, 64, 64);">的数据。</font>



（听上去有点绕？你看完下面的介绍在回过头来看，就能明白了。。。）



想要回答好这个问题，需要先了解一下Seata的AT模式的工作原理。



[✅Seata的AT模式的实现原理](https://www.yuque.com/hollis666/qyhor6/me3ge4vavi0fokgq)



上面这篇很详细的介绍了原理，这里我们简单总结下，把其中关键点单独拿出来说一下。



<font style="color:rgb(64, 64, 64);">AT模式的核心机制是两阶段提交：</font>

+ **<font style="color:rgb(64, 64, 64);">第一阶段</font>**<font style="color:rgb(64, 64, 64);">：本地事务立即提交，释放本地锁，数据对</font>**<font style="color:rgb(64, 64, 64);">其他事务可见</font>**<font style="color:rgb(64, 64, 64);">。</font>
+ **<font style="color:rgb(64, 64, 64);">第二阶段</font>**<font style="color:rgb(64, 64, 64);">：全局事务根据协调结果决定提交或回滚（通过undo log补偿）。</font>

<font style="color:rgb(64, 64, 64);">  
</font><font style="color:rgb(64, 64, 64);">那么，大家仔细想一下这个场景，其实会出现一种情况，那就是如果全局事务最终回滚，其他事务可能在P第一阶段结束后、第二阶段回滚前读取到</font>**<font style="color:rgb(64, 64, 64);">已提交但即将被撤销的数据</font>**<font style="color:rgb(64, 64, 64);">，导致</font>**<font style="color:rgb(64, 64, 64);">逻辑上的脏读</font>**<font style="color:rgb(64, 64, 64);">。</font>



举个例子。



现在有三个模块，交易模块、订单模块和库存模块。在一次下单过程中，为了保证一致性，代码可能如下：



```plain
@Service
public class TradeService{

  @Autowired
  private InventoryService inventoryService;

  @Autowired
  private OrderService orderService;


  @GlobalTransactional
  public boolean buy(){

    //库存扣减
    inventoryService.decreaseInvenroty();
    
    //创建订单
    orderService.createOrder();
  }
  

}
```





在这个分布式事务中（@GlobalTransactional开启的分布式事务），先调库存服务进行库存扣减，然后再调用订单服务进行订单创建。那么整个（大致）流程就是这样的：



![](https://cdn.nlark.com/yuque/0/2025/png/5378072/1740288289686-f88e4185-5f0f-4352-9899-837f87b80694.png)



这里需要注意的是，在第二步调库存模块之后，库存模块的在数据库上的操作，都是基于数据库的本地事务的，他需要通过数据库的本地事务来保障undolog（Seata用到的的undolog，和MySQL中MVCC的那个undolog不是一个）和库存扣减的原子性，并且这一步执行完之后，**数据库的本地事务是要做提交的！**

****

这很关键，数据库的本地事务做了提交，事务提交了，也就意味着，不管在什么样的事务隔离级别下，其他的事务都能查到提交后的新值了。



那么试想一下，如果在第二步执行成功之后，库存已经完成了扣减，但是第三步执行订单创建执行失败了，这时候整个分布式事务是要回滚的，那么就会基于库存库中的undolog做回滚。那么，在提交后，回滚前，如果有其他的事务来查询库存数据，是不是就读到了一个本该回滚的值？



这是不是也是一种脏读，只不过这个脏读并不是MySQL的本地事务中的脏读，而是Seata全局事务中的脏读。即在全局事务过程中，别的事务可能会读到全局事务尚未提交（后面可能会回滚）的数据。



# 扩展知识


## AT如何避免脏读


AT模式的脏读是他的实现机制导致的，因为他的第一阶段是借助分支事务实现的，利用分支事务的ACID保证业务操作和undolog的写入的原子性。但是第一阶段执行完，整个全局事务并没有确定要不要提交，还是有可能会回滚的。一旦发生回滚，那么在回滚前，就会能读到脏数据了。



那么这个问题如何避免呢？



其实没啥好办法，因为事务已经提交了，没办法避免其他事务的读取，就算能实现，也会大大降低可用性。所以如果不能接受脏读，那么不要使用AT模式，可以选择其他的事务方案，比如TCC。





