# 典型回答


在Oracle中，有专门的函数实现了行转列和列转行的功能，分别是pivot和unpivot函数。



`**PIVOT**`** 用于将行数据转换为列数据**。通常应用在需要对数据进行聚合（如求和、计数等）并将某一列的不同值转化为列名的情况。  



假设有一个表 `sales`，结构如下：

| year | product | sales |
| --- | --- | --- |
| 2020 | A | 100 |
| 2020 | B | 200 |
| 2021 | A | 150 |
| 2021 | B | 300 |


我们希望将 `product` 列中的值（`A` 和 `B`）转换为列，得到以下格式：

| year | A | B |
| --- | --- | --- |
| 2020 | 100 | 200 |
| 2021 | 150 | 300 |


在 Oracle 中，`PIVOT` 操作的实现如下：

```sql
SELECT *
FROM sales
PIVOT (
  SUM(sales) FOR product IN ('A' AS A, 'B' AS B)
)
ORDER BY year;
```



`**UNPIVOT**`** 用于将列数据转换为行数据。**通常用于将表中的列名转换成行数据，以便对数据进行进一步分析。  



假设有一个表 `sales_data`，结构如下：

| year | A | B |
| --- | --- | --- |
| 2020 | 100 | 200 |
| 2021 | 150 | 300 |


我们希望将 `A` 和 `B` 列的数据转换成行数据，得到以下格式：

| year | product | sales |
| --- | --- | --- |
| 2020 | A | 100 |
| 2020 | B | 200 |
| 2021 | A | 150 |
| 2021 | B | 300 |




在 Oracle 中，`UNPIVOT`操作的实现如下：



```sql
SELECT 
    year,
    product,
    sales
FROM sales_data
UNPIVOT (
    sales FOR product IN (A AS 'A', B AS 'B')
)
ORDER BY year, product;

```



# 扩展知识


## 其他实现


不是用函数的其他实现方式，可以参考MySQL中的实现：



[✅MySQL如何实现行转列和列转行？](https://www.yuque.com/hollis666/qyhor6/wb2h3s2lsaw5egk8)

