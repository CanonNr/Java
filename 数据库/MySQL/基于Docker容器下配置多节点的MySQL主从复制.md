# 配置文件

MySQL 开发人员觉得给你们太多参数会被你改坏掉，防止手贱`5.7`的版本已经不会给你一个默认的`.cnf`文件了，需要自己到指定目录创建一个`my.cnf`。

- 节点一

  ```shell
  [mysqld]
  log-bin=mysql-bin   
  server-id=1          
  ```

- 节点二

  ```shell
  [mysqld]
  log-bin=mysql-bin    
  server-id=2          
  ```



# 查看主库状态

`Node1`为主，`Node2`为从

```sql
show master status

+------------------+----------+--------------+------------------+-------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+------------------+----------+--------------+------------------+-------------------+
| mysql-bin.000003 |     1338 |              |                  |                   |
+------------------+----------+--------------+------------------+-------------------+

--- File  和 Position 记住 后面要考
```



# 配置从库

在`Node2`命令行执行如下：

```sql
mysql>change master to
master_host='MySQL.Cluster.Node1',	--- 因为配置了 Docker 的网络是可以通过容器名通讯的 
master_user='root',				   --- 密码，正常是不会用root的
master_log_file='mysql-bin.000003', --- File字段
master_log_pos=1201,			   --- Position字段
master_port=3306,				   --- 正常的端口 不要写映射出去的端口
master_password='root';			    --- 密码
```

```sql
start slave;  --- 开启从机模式
--- 如果上面写错了需要重新配置 也可以关闭重新配置
stop slave
```



# 查看主机状态

```sql
show slave status
```