---
layout: post
title: centos7 mysql5.7一主多从配置
date: 2018-08-14
categories: blog
tags: [centos,nginx]
description: centos7 mysql5.7一主多从配置
---


#### centos7 mysql5.7一主多从配置

###### 准备工作

- 主centos1: 192.168.0.165

- 从centos2: 192.168.0.168

- 从centos3: 192.168.0.163



###### 主库配置

1. 开启binlog

```
vim /etc/my.cnf

[mysqld]
log-bin=mysql-bin
server-id=1
```

2. 创建账号并仅授予复制权限

```
create user 'reps'@'192.168.0.%' identified by '123456';
grant replication slave on *.* to 'rep'@'192.168.0.%';
```

3. 记录主库file以及position

```
mysql> show master status;
+------------------+----------+--------------+------------------+-------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+------------------+----------+--------------+------------------+-------------------+
| mysql-bin.000003 |      447 |              |                  |                   |
+------------------+----------+--------------+------------------+-------------------+
1 row in set (0.00 sec)

```

###### 从库centos2配置

1. 配置mysql

```
vim /etc/my.cnf
[mysqld]
server-id=2
```

2. 配置主库通信

```
CHANGE MASTER TO MASTER_HOST='192.168.0.165', MASTER_USER='reps', MASTER_PASSWORD='123456', MASTER_LOG_FILE='mysql-bin.000003', MASTER_LOG_POS=447;
```

3. 启动从服务器复制线程

```
start slave;
```

4. 查看状态

```
mysql> show slave status\G;
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 192.168.0.165
                  Master_User: reps
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000003
          Read_Master_Log_Pos: 154
               Relay_Log_File: localhost-relay-bin.000002
                Relay_Log_Pos: 320
        Relay_Master_Log_File: mysql-bin.000003
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
              Replicate_Do_DB:
            ...
            ...

```


5. 说明
 

```
Slave_IO_State #从站的当前状态 
Slave_IO_Running： Yes #读取主程序二进制日志的I/O线程是否正在运行 
Slave_SQL_Running： Yes #执行读取主服务器中二进制日志事件的SQL线程是否正在运行。与I/O线程一样 
Seconds_Behind_Master #是否为0，0就是已经同步了
```


###### 从库centos3配置


1. 配置mysql

```
vim /etc/my.cnf
[mysqld]
server-id=3
```

2. 配置主库通信

```
CHANGE MASTER TO MASTER_HOST='192.168.0.165', MASTER_USER='reps', MASTER_PASSWORD='123456', MASTER_LOG_FILE='mysql-bin.000003', MASTER_LOG_POS=447;
```

3. 启动从服务器复制线程

```
start slave;
```

4. 查看状态

```
mysql> show slave status\G;
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 192.168.0.165
                  Master_User: reps
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000003
          Read_Master_Log_Pos: 154
               Relay_Log_File: localhost-relay-bin.000002
                Relay_Log_Pos: 320
        Relay_Master_Log_File: mysql-bin.000003
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
              Replicate_Do_DB:
            ...
            ...

```


###### 测试

1. 主库插入一条数据

```
mysql> select * from test_table;
+----+----------+
| id | name     |
+----+----------+
|  1 | panxu    |
|  2 | panxu222 |
|  3 | panxu333 |
|  4 | panxu444 |
|  5 | panxu666 |
+----+----------+
5 rows in set (0.00 sec)

mysql> insert into test_table(name) values('panxu_lalalalala');
Query OK, 1 row affected (0.00 sec)

mysql> select * from test_table;
+----+------------------+
| id | name             |
+----+------------------+
|  1 | panxu            |
|  2 | panxu222         |
|  3 | panxu333         |
|  4 | panxu444         |
|  5 | panxu666         |
|  6 | panxu_lalalalala |
+----+------------------+
6 rows in set (0.00 sec)

```

2. 从库1查询数据

```
mysql> select * from test_table;
+----+----------+
| id | name     |
+----+----------+
|  1 | panxu    |
|  2 | panxu222 |
|  3 | panxu333 |
|  4 | panxu444 |
|  5 | panxu666 |
+----+----------+
5 rows in set (0.00 sec)

mysql> select * from test_table;
+----+------------------+
| id | name             |
+----+------------------+
|  1 | panxu            |
|  2 | panxu222         |
|  3 | panxu333         |
|  4 | panxu444         |
|  5 | panxu666         |
|  6 | panxu_lalalalala |
+----+------------------+
6 rows in set (0.00 sec)

```


3. 从库2查询数据

```
mysql> select * from test_table;
Empty set (0.00 sec)

mysql> select * from test_table;
+----+------------------+
| id | name             |
+----+------------------+
|  6 | panxu_lalalalala |
+----+------------------+
1 row in set (0.00 sec)

```
