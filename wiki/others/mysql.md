---
title: MySQL
date: 2018-10-14 23:45:28
tags:
---

## MySQL

~~~
sudo apt-get install mysql-server
apt-get isntall mysql-client
sudo apt-get install libmysqlclient-dev
~~~

### 基本操作

- create database xxx; # 创建数据库
- create user doc@'%' identified by 'mysql'; # 创建用户doc,密码mysql,所有域名或IP可访问（‘%’）

### 远程登录

授权root可以远程链接的权限：

~~~
update user set host = '%' where user = 'root';
GRANT ALL PRIVILEGES ON \*.\* TO 'root'@'%'WITH GRANT OPTION
~~~

### 配置 /etc/mysql

* 绑定只能本地访问：bind_address = 127.0.0.1
* 表名小写：lower_case_table_names = 1