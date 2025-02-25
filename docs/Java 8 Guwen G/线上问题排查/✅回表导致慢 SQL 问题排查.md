### 问题发现


线上突然疯狂报警，某个接口的调用成功率暴跌，失败数突增。



![](https://cdn.nlark.com/yuque/0/2024/png/5378072/1722059859328-3ded9508-0e3c-43df-9601-5b86283d8918.png)



经过一番排查，发现底层数据库CPU已经被拉到了100%，在进行了紧急扩容后，开始进行根因分析。



### 问题排查


通过数据库的诊断工具，发现有一个慢SQL，执行了很多次，并且耗时比较长，找到的SQL具体如下：

![](https://cdn.nlark.com/yuque/0/2024/png/5378072/1722060338409-e3cd2159-3007-4964-87c8-b676faf91d3e.png)



这个用户确实是一个大买家、表中一共100多万条数据，他自己就有几十万单（新的业务模式，之前也没见过）



并且在分析过程中，发现这个 SQL 有的时候走索引，有的时候不走索引，并且走不走索引都很慢。于是看一下成本消耗到底在哪里。



[✅MySQL的优化器的索引成本是怎么算出来的？](https://www.yuque.com/hollis666/qyhor6/waruyhds7gcn6srf)



```java
EXPLAIN FORMAT=json
SELECT ifnull(SUM(sum_payment), 0) AS order_actl_pay_fee_sum
	, COUNT(1) AS order_num
FROM xxxx_payment_message
WHERE byr_id = 221xxxx478
	AND biz_date >= '1719391999351'
	AND biz_date <= '1719478399351'; 
```



得到的结果如下：



```java

{
  "query_block": {
    "select_id": 1,
    "cost_info": {
      "query_cost": "139147.15" # 总成本
    },
    "table": {
      "table_name": "xxxx_payment_message",
      "access_type": "ALL",
      "possible_keys": [
        "byr_id_date_index",
        "biz_date_index"
      ],
      "rows_examined_per_scan": 1364869,
      "rows_produced_per_join": 448231,
      "filtered": "32.84",
      "cost_info": {
        "read_cost": "94323.95", # IO Cost(Engine Cost)
        "eval_cost": "44823.20", # CPU Cost(Server Cost)
        "prefix_cost": "139147.15", # 总成本
        "data_read_per_join": "458M"
      },
      "used_columns": [
        "byr_id",
        "biz_date",
        "sum_payment"
      ]
    }
  }
}
```



这条 SQL 不走索引，实际耗时675ms，预估扫描行数1374150行 ，cbo为139147.15。



在看下以下这个SQL：



```java
EXPLAIN FORMAT=json
SELECT ifnull(SUM(sum_payment), 0) AS order_actl_pay_fee_sum
	, COUNT(1) AS order_num
FROM xxxx_payment_message force index(byr_id_date_index)
WHERE byr_id = 2217618275478
	AND biz_date >= '1719392683881'
	AND biz_date <= '1719479083881'; 
```



得到结果如下：



```java
{
  "query_block": {
    "select_id": 1,
    "cost_info": {
      "query_cost": "201704.66"# 总成本
    },
    "table": {
      "table_name": "xxxx_payment_message",
      "access_type": "range",
      "possible_keys": [
        "byr_id_date_index"
      ],
      "key": "byr_id_date_index",
      "used_key_parts": [
        "byr_id",
        "biz_date"
      ],
      "key_length": "18",
      "rows_examined_per_scan": 448232,
      "rows_produced_per_join": 448232,
      "filtered": "100.00",
      "cost_info": {
        "read_cost": "156881.46",# IO Cost(Engine Cost)
        "eval_cost": "44823.20",# CPU Cost(Server Cost)
        "prefix_cost": "201704.66",# 总成本
        "data_read_per_join": "458M"
      },
      "used_columns": [
        "byr_id",
        "biz_date",
        "sum_payment"
      ]
    }
  }
}
```



这个 SQL 走索引，执行耗时709ms，扫描行数448232行，cbo为201704.66



两条SQL，一条走索引，一条不走，耗时都在700ms 左右，这说明优化器确实在走与不走索引之间摇摆，并且摇摆的也不无道理，毕竟即使走了索引也没有快到哪里去。



因为我们介绍过 MySQL是基于cbo的优化器，会对每一个索引计算成本消耗，选择成本最低的索引，但如果通过主键进行全表扫描的成本消耗比二级索引要低，那就会选择全表扫描。



这里就开始疑惑了，为啥走索引竟然比全表扫描还要低呢？分析上面的第一个执行计划的结果，我们发现成本这里，主要的消耗在 IO 上面，而 IO 的成本主要是从内存读取以及从磁盘读取数据的消耗，而走了索引的竟然消耗也这么高，不太科学。



```java
"cost_info": {
"read_cost": "94323.95", # IO Cost(Engine Cost)
"eval_cost": "44823.20", # CPU Cost(Server Cost)
"prefix_cost": "139147.15", # 总成本
"data_read_per_join": "458M"
},
```



于是继续去排查索引和 SQL 的情况（其实早就应该查这个，只不过前面一直在看为啥同样的 SQL 有的走索引，有的不走），最终发现 SQL 虽然命中了索引，但是还是需要回表，因为我们的索引中只包含了byr_id和 biz_date这两个字段，而 sum_payment 并不在里面，这就意味着每次查询都需要回表。



所以即使走了索引，他的IO消耗也很高，主要都在回表上。



### 问题解决


问题定位到了，解决就简单了。



直接把sum_payment也加到byr_id和 biz_date这的索引中，组成一个联合索引即可，减少回表次数。



优化后，这个大买家的查询 SQL 的执行耗时在200ms 以内就能返回，io 成本也降到了原来的1/3左右。

