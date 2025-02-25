# 典型回答


这个问题其实考察的是 Redis 的**内存淘汰策略。**

****

所谓内存淘汰策略，其实就是Redis 的内存淘汰策略用于在内存满了之后，决定哪些 key 要被删除。Redis 支持多种内存淘汰策略，可以通过配置文件中的 maxmemory-policy 参数来指定。



[✅Redis的内存淘汰策略是怎么样的？](https://www.yuque.com/hollis666/qyhor6/xw99lcraocebx1mk)



Redis 支持很多种内存淘汰策略，如：



+ noeviction：不会淘汰任何键值对，而是直接返回错误信息。
+ allkeys-lru：从所有 key 中选择最近最少使用的那个 key 并删除。
+ volatile-lru：从设置了过期时间的 key 中选择最近最少使用的那个 key 并删除。
+ allkeys-random：从所有 key 中随机选择一个 key 并删除。
+ volatile-random：从设置了过期时间的 key 中随机选择一个 key 并删除。
+ volatile-ttl：从设置了过期时间的 key 中选择剩余时间最短的 key 并删除。
+ volatile-lfu：淘汰的对象是带有过期时间的键值对中，访问频率最低的那个。
+ allkeys-lfu：淘汰的对象则是所有键值对中，访问频率最低的那个。





因为有了淘汰策略，所以 Redis 即使内存满了，也不会立刻就挂了，而是会基于淘汰策略去移除一些 key来腾挪空间出来。



这里需要注意的是，即使是noeviction这种策略，也不会导致 Redis 立刻就挂了，如果满了，Redis 会拒绝写入操作，并返回 `OOM command not allowed when used memory > 'maxmemory'` 错误。



**总结一下就是，Redis 本身设计上不会因为内存满而崩溃，但需要通过适当的配置来管理内存使用和处理策略，以保证服务的稳定性和性能。**

