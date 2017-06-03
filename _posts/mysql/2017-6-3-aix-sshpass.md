---
layout: post
title: "MySQL NULL值和空字符串区别"
keywords: "mysql,NULL,空字符串"
description: ""
category: mysql
tags: [MySQL]
---
{% include JB/setup %} 

`null`值需要额外的空间记录其是否为空，空字符串`''`不占用存储空间，MySQL中`null`是占用空间的，
所以MySQL在进行比较的时候，`null`会参与字段比较，对效率有一部分的影响。





索引采用B树实现，而B树是为了处理等于、范围查找、排序等操作，`null`没法采用=操作，所以B树索引时不会存储`null`值的，
只要列中包含有`null`值都将不会包含在索引中，复合索引中只要有一列含有`null`值，那么这一列对于此复合索引就是无效的。即使对该列建索引也不会提高性能。

在where子句中使用 IS NULL或IS NOT NULL的语句优化器是不允许使用的。

`null`与`任何数据类型`作比较运算都不为真。

为了处理null值，可以使用IS NULL或IS NOT NULL等操作符匹配，使用`''`无法匹配null值。

MySQL中任意类型的字段和`null`进行计算结果都为null。