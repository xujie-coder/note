# 典型回答


### 行转列
所谓行转列，就是假设有如下数据表 `sales`：

| id | year | product | sales |
| --- | --- | --- | --- |
| 1 | 2020 | A | 100 |
| 2 | 2020 | B | 200 |
| 3 | 2021 | A | 150 |
| 4 | 2021 | B | 300 |




要将 `product` 列的不同值（A 和 B）变为列标题，`year` 作为行标题，生成如下结果：

| year | A | B |
| --- | --- | --- |
| 2020 | 100 | 200 |
| 2021 | 150 | 300 |




这里，可以使用`CASE WHEN THEN` 用来根据条件为每个 `product` 创建一列，并通过 `SUM` 聚合同一年份的销售数据，达到行转列的效果。  



```java
SELECT
    year,
    SUM(CASE WHEN product = 'A' THEN sales ELSE 0 END) AS A,
    SUM(CASE WHEN product = 'B' THEN sales ELSE 0 END) AS B
FROM sales
GROUP BY year;
```



简答解释下：

+ `**SUM(CASE WHEN product = 'A' THEN sales ELSE 0 END) AS A**`：
+ 使用 `CASE WHEN` 判断 `product` 是否为 `A`：
    - 如果是 `A`，则返回该行的 `sales` 值。
    - 如果不是 `A`，则返回 `0`。
+ 然后，`SUM` 函数将每个 `year` 内所有 `product = 'A'` 的 `sales` 值加总，最终得到该 `year` 对应的 `A` 产品总销售额。



还可以使用`IF`实现类似`CASE WHEN`的功能：

```java
SELECT
    year,
    SUM(IF(product = 'A', sales, 0)) AS A,
    SUM(IF(product = 'B', sales, 0)) AS B
FROM sales
GROUP BY year;
```



### 列转行


所谓列转行，假设有如下数据表  sales_data  ：



| year | A | B |
| --- | --- | --- |
| 2020 | 100 | 200 |
| 2021 | 150 | 300 |




转成如下格式结构：



| year | product | sales |
| --- | --- | --- |
| 2020 | A | 100 |
| 2020 | B | 200 |
| 2021 | A | 150 |
| 2021 | B | 300 |




通常是使用UNION  ALL   实现：



```sql
SELECT 
    year,
    'A' AS product,
    A AS sales
FROM sales_data

UNION ALL

SELECT 
    year,
    'B' AS product,
    B AS sales
FROM sales_data;

ORDER BY year, product;

```



+ 第一个 `SELECT` 子查询提取了 `A` 列的数据，并将 `A` 作为 `product` 值：
+ 输出行：`(2020, 'A', 100)` 和 `(2021, 'A', 150)`。
+ 第二个 `SELECT` 子查询提取了 `B` 列的数据，并将 `B` 作为 `product` 值：
+ 输出行：`(2020, 'B', 200)` 和 `(2021, 'B', 300)`。
+ `UNION ALL` 将两组查询结果合并。

