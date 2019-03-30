---
layout : post
title : "Mysql 测试数据"
categories : Database
tags: MySQL
author : ABei
---

* content
{:toc}

前段时间在学习 Hibernate 框架，但是苦于没有什么合适的测试数据来模拟大数据环境下对 Hibernate 的优化。在网上找了很久才找到相应的数据，故在此记录一下。



## 数据获取地址

数据库数据可以从 [Sample database with test suite](https://launchpad.net/test-db/) 上下载。

> 请选择 **employees-db-full-1.0.6** 下载。

## 导入数据库

解压压缩包，在路径 `employees_db` 下找到 `employees.sql` 并打开。

将下面的代码：

```sql
set  storage_engine = InnoDB;
select CONCAT('storage engine: ', @@ storage_engine) as INFO;
```

修改为：

```sql
set  default_storage_engine = InnoDB;
select CONCAT('storage engine: ', @@ default_storage_engine) as INFO;
```

这样做的原因如下：

```txt
From the 5.6 docs: default_storage_engine should be used in preference to storage_engine, which is deprecated.
```

所以，在导入数据之前，请检查一下自己当前 MySQL 版本是否高于 5.6。

在命令窗口执行：

```cmd
mysql -u root -p
source F:\employees_db-full-1.0.6\employees_db\employees.sql
```

表关系图如下：

![](http://cdn.51leif.com/emloyees.png "employees_data_model")