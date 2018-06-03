---
title: MySQL-语法
date: 2018-04-17 10:05:20
tags: DB
---

**WHERE vs HAVING**
首先，二者都可以实现过滤记录的功能。
区别主要有：
* 都作为过滤记录使用时，WHERE 可以作用于表的任意列字段，而 HAVING 只能作用于之前已经筛选出的列字段；
* 作用对象不同，WHERE 作用于表和视图，HAVING 作用于组。WHERE 无法与聚集函数（AVG() COUNT() SUM() MAX() MIN() 等）一起使用，即，再筛选条件只能是表中的列，而 HAVING 可以将结果的聚集处理组作为再筛选条件，即，一般会包含聚集函数，跟在 GROUP BY 之后；
* 即，WHERE 在聚合前筛选，即作用在 GROUP BY 和 HAVING 之前，HAVING 在聚合后对组记录再进行筛选。

****
