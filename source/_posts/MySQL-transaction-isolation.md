---
title: MySQL中事物的隔离级别
date: 2020-03-03 02:38:39
tags: MySQL
---

····

<!--more-->

# MySQL中事物的隔离级别

```
事物的隔离界别
					脏读		不可重复读		幻读
read uncommited 	√			√				√
read commited		×			√				√
repeatable read		×			×				√
serialization		×			×				×

mysql中默认 repeatable read
oracle中默认 read commited

查看隔离级别：select @@tx_isolation;
设置隔离级别：set session|global transaction isolation level 隔离级别;
```
## read uncommitted
**无隔离**  
脏读（演示）、不可重复读（演示）、幻读（这里不演示）  

- 终端1中显示当前隔离级别  
- 设置当前隔离级别为read committed  
- 在此查看当前隔离级别  
- 进入test数据库  
- 查看accout表信息  
- 开启事物  
- 在此事物中进行列的修改  
- 回退  

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/read_uncommitted1.png"/>

- 终端2显示当前隔离级别
- 设置当前隔离级别为read committed
- 在此查看当前隔离级别
- 进入test数据库
- 查看account表信息（在终端1做出修改前）
- 查看account表信息（在终端1做出修改后）
- 查看account表信息（在终端1回退后）

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/read_uncommited2.png"/>


## read committed
不脏读（演示）、不可重复读（不演示）、幻读（这里不演示）  

- 设置终端1类型read committed
- 开启事物
- 修改列
- 结束事务

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/read_commited1.png"/>

- 设置终端2类型read committed   
- 开启事务  
- 查看account表信息（终端做修改后。但未结束事务，不脏读）  
- 查看account表信息（终端1做修改后，结束事务，终端2还处于同一事务中，此隔离级别下不可重复读）  
- 结束事务

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/read_commited2.png"/>

## repeatable read
不脏读（演示）、不可重复读（演示）、幻读（这里不演示）  


- 设置终端1类型repeatable read
- 开启事物
- 修改列
- 结束事务

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/repeatable_read1.png"/>

- 设置终端2类型repeatable read
- 开启事务
- 查看account表信息（终端1做修改后，未结束事务）
- 查看account表信息（终端1做修改后，结束事务，终端2未结束事务，此隔离级别下可重复读）
- 结束事务

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/repeatable_read2.png"/>

- 终端1开启事务
- 查看account表信息
- 插入一行信息
- 查看account表信息
- 结束事务

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/repeatable_read_illusion1.png"/>

- 终端2开启事务  
- 查看account表信息（终端1做出修改后）  
- 修改表中的列信息（终端1做出修改，但未结束事务，时间长了终端2会出现来纳瑟方框报错）  
- 在此尝试修改表中列信息（终端1做出修改并结束事务，出现红色下划线提示，4个改变，但在终端2事务中最初只有3行信息，此处却出现4行，说明出现幻读）

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/repeatable_read_illusion2.png"/>


## serialization
不脏读、可重复度、不幻读

- 设置终端1类型serialization
- 查看当前隔离级别
- 开启事务
- 查看account表信息
- 修改表的列信息（此处并未提交执行此语句）

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/serializable1.png"/>

- 设置终端1类型serialization
- 开启事务
- 修改表的列信息（终端1的表修改操作在终端2前，此时终端2列修改无法操作，红色方框未报错信息，等待时间过长，只有端事务1事务结束才能进行此操作）

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/serializable2.png"/>
