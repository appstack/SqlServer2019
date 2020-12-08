# SQL 基础知识


# 前言

> 此次新增具体的示例数据库SQL_Road，SQL Server 作为讲解平台。

# 第一章 SELECT 的用法

##### SQL 执行顺序

> 在普及知识之前，我想先让大家看一下数据库在运行时的先后顺序，这个请务必多看多记，因为它真的很重要，要深入学习一定要记得这个运行先后顺序。

> (**8**)**SELECT** 
>
> (**9**)DISTINCT 
>
> (**11**)\<Top Num\> \<select list\>
>
> (**1**)FROM \[left_table\]
>
> (**3**)\<join_type\> JOIN \<right_table\>
>
> (**2**) ON \<join_condition\>
>
> (**4**) WHERE \<where_condition\> 
>
> (**5**)GROUP BY \<group_by_list\> 
>
> (**6**)WITH \<CUBE \| RollUP\> 
>
> (**7**)HAVING \<having_condition\> 
>
> (**10**)ORDER BY \<order_by_list\>

##### SELECT 的用法

> 先来讲解 SELECT 的用法。

```sql
SELECT column_name,column_name
FROM table_name;
```
与
```sql
SELECT * FROM table_name;
```


# 第二章 DISTINCT 的用法

> 

# 第三章 TOP 的用法

> 
