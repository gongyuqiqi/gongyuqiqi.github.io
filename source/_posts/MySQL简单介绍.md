---
title: MySQL简单介绍
date: 2020-02-25 16:30:23
tags: MySQL
---

MySQL的下载安装  
简单命令介绍  
SQLyog的下载安装

<!--more-->

# MySQL简单介绍

### MySQL下载安装

[下载安装教程](https://baijiahao.baidu.com/s?id=1629661608981614271&wfr=spider&for=pc)  
Choose Setup Type一栏，我选择的是**Custom**  
安装路径自定义到了非系统盘
### MySQL服务的启动和停止

- 方式一：计算机---右击管理---服务  
<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/MySQL服务.png"/>
右击MySQL会显示启动、停止等选项

        

- 方式二：通过管理员身份运行cmd  
        net restart 服务名称（启动服务）  
        net stop 服务名称（停止服务）

### MySQL服务的登陆和退出  

- 方式一：通过MySQL自带的客户端，只限于root用户
<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/MySQL_Command_Line.png"/>

- 方式二：通过windows自带的客户端（cmd）  
mysql [-h 主机名 -P 端口号] -u 用户名 -p密码  
如果是本机的话，`[]`中的内容可以省去  

退出：exit或者ctrl+C
<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/MySQL_cmd_login.png"/>

### MySQL配置环境变量  

my.ini文件为MySQL的配置文件

### MySQL常见命令介绍

- 显示数据库信息

```sql
show databases;
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/show_databases.JPG"/>


information_schema：保存源数据细信息。  
mysql：保存用户信息。  
performance_schema：搜集性能信息，性能参数。  
test：测试数据哭，为空。可以在次建表，修改库，删除库。  

- 进入数据库mysql

```sql
use mysql;
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/use_mysql.JPG"/>


- 查看数据库中的表信息

```sql
show tables;
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/show_tables.JPG"/>


- 查看当前属于哪个数据库

```sql
select database();
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/select_database().JPG"/>


- 在数据库中新建一张表（test数据库）

```sql
use test;
select database()；
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/use_test.JPG"/>


```sql
creat table friends(
    id int,
    name varchar(20));
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/creat_friends.JPG"/>


- 查看当前数据库中的表

```sql
show tables();
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/show_table_friends1.JPG"/>


- 查看friends表的结构

```sql
desc friends;
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/desc_friends.JPG"/>


- 查看friends表的信息

```sql
select * from friends
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/select_friends1.JPG"/>


- 为friends表添加数据信息


```sql
insert into friends(id,name) values(1,"susu");
insert into friends(id,name) values(2,"zhengrong");
insert into friends(id,name) values(3,"yinyu");
insert into friends(id,name) values(4,"jiaru");
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/insert_friends.JPG"/>


- 查看friends表中的信息

```sql
select * from friends;
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/select-friends2.JPG"/>


- 修改friends表中的信息

```sql
update friends set name="SUSU" where id=1
select * from friends
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/updatesusu.JPG"/>

- 删除friends表中某项信息

```sql
delete from friends where id=4
select * from friends
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/deletejiaru.JPG"/>


### 查看MySQL服务端版本

- 方法一： mysql里面查询

```sql
select version();
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/select_version().JPG"/>


- 方法二： cmd里面查询

```sql
mysql --version
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/mysql__version.JPG"/>


### MySQL的语法规范

1. 不区分大小写
2. 每条命令最好用分号结尾
3. 每条命令根据需要，可以进行换行或缩进
4. 注释
        单行注释：#注释文字
        单行注释：-- 注释文字
        多行注释：/*注释文字*/

## SQLyog的下载安装

[SQLyog下载安装教程](https://www.cnblogs.com/chunguang-yao/p/10666429.html)  
<font color=red>注意</font>：在进行连接的时候一定要保证MySQL服务是开启的。如果经常用到可以改成**自动开启模式**，如果不常用可以改成**手动开启模式**，手动开启模式下就要注意用之前要保证服务是开启的。

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/SQLyogintroduction.png"/>

- 1——以root用户的方式进行登陆
- 2——SQL语句框，在此可以输入语句，自动将输入的语句规范化
- 3——执行完语句后会在此处收到相应的消息
- 4——当前处于mysql数据库，1中加粗字体**mysql**也表明当前处于mysql数据库，2中最后一条执行语句为`USE mysql;`