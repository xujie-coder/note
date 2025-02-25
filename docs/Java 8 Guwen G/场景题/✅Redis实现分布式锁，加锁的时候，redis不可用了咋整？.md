# 典型回答


当我们使用Redis作为分布式锁的时候，如果出现了Redis不可用，那么就会导致整个系统没办法进行加锁和解锁操作了，并且业务流程会因为无法加锁和解锁而卡住，无法继续执行。



一般来说，我们在实际工作中，都不太会考虑这种情况，因为Redis基本都是集群部署的，有高可用性保障的，为了这种极低概率发生的事情做很复杂的方案的特别少。所以，这个问题也是一个典型的“面试造火箭”。



那么，如果真的出现了这种情况，我们可以做的几个事情主要就是**降级**。让我们应用不要强依赖这个Redis的分布式锁，而导致整体系统的不可用。



有两种降级方案，第一种是降级为本地锁，当Redis的分布式锁不可用的时候，我们可以把锁降级成使用Java的本地锁，比如ReentrantLock或者synchronized等，这样可以确保即使 Redis 不可用，程序依然能够在**单机模式**下正常工作，避免系统崩溃。  当然，本地锁和分布式锁最大的问题就是不能保证全局互斥，所以只能用在可以容忍不严格分布式锁的情况下，简单点说，就是你可以牺牲一致性，保可用性的场景。



假如说你在底层有其他的防并发手段兜底，比如乐观锁、比如数据库锁、比如唯一性索引等等的时候，可以考虑用这个方案。



第二种降级方案，那就是降级为其他的分布式锁方案，比如借助Zookeeper，或者数据库实现的分布式锁。这个方案的好处是他还是个分布式锁，能起到全局互斥的作用，但是缺点是原来的Redis挂了，里面已有的锁可能没办法同步过来，还有就是这个方案太复杂了，成本太高了。



以上，就是一些可以做的容灾手段，至于其他的，通过Redis的集群模式、哨兵模式，高可用性保障等等也可以随便聊，只不过如果面试官一定限定了Redis就是挂了的话，可以聊这些降级的方案。



# 扩展知识


## 如何实现降级


想要让你的应用可以在Redis不可用的时候降级到其他的锁，那么就需要提前做改造，代码中预先的预留一个if-else，然后可以通过手动或者自动的方式让你的代码在if和else之间切换。



比如手动方式的话，可以通过配置中心推送一个配置（如useRedisLock的值），然后基于这个配置走Redis的分布式锁还是降级锁：



```sql
if(useRedisLock){
  //使用Redis分布式锁  
}else{
  //使用降级锁
}
```



还有一种自动的方案，那就是自己在代码中重试几次，失败的话降级：



```sql
// 业务逻辑
public void businessLogic() {
    boolean lockAcquired = false;

    // 先尝试获取 Redis 锁
    for (int i = 0; i < RETRY_COUNT; i++) {
        lockAcquired = tryRedisLock();
        if (lockAcquired) {
            break;
        }
        try {
            Thread.sleep(RETRY_DELAY); // 重试间隔
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
    }

    if (!lockAcquired) {
        // 如果 Redis 锁获取失败，降级使用本地锁
        System.out.println("Redis 锁不可用，使用本地锁降级");

        localLock.lock();
        try {
            // 业务逻辑
            System.out.println("执行业务逻辑（本地锁）");
        } finally {
            localLock.unlock();
        }
    } else {
        try {
            // 执行 Redis 锁下的业务逻辑
            System.out.println("执行业务逻辑（Redis 锁）");
        } finally {
            // 解锁
            releaseRedisLock();
        }
    }
}
```



