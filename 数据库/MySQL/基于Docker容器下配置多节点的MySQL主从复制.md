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

*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: MySQL.Cluster.Node1
                  Master_User: root
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000006
          Read_Master_Log_Pos: 674
               Relay_Log_File: 411f68192261-relay-bin.000005
                Relay_Log_Pos: 887
        Relay_Master_Log_File: mysql-bin.000006
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
              Replicate_Do_DB:
          Replicate_Ignore_DB:
           Replicate_Do_Table:
       Replicate_Ignore_Table:
      Replicate_Wild_Do_Table:
  Replicate_Wild_Ignore_Table:
                   Last_Errno: 0
                   Last_Error:
                 Skip_Counter: 0
          Exec_Master_Log_Pos: 674
              Relay_Log_Space: 1267
              Until_Condition: None
               Until_Log_File:
                Until_Log_Pos: 0
           Master_SSL_Allowed: No
           Master_SSL_CA_File:
           Master_SSL_CA_Path:
              Master_SSL_Cert:
            Master_SSL_Cipher:
               Master_SSL_Key:
        Seconds_Behind_Master: 0
Master_SSL_Verify_Server_Cert: No
                Last_IO_Errno: 0
                Last_IO_Error:
               Last_SQL_Errno: 0
               Last_SQL_Error:
  Replicate_Ignore_Server_Ids:
             Master_Server_Id: 1
                  Master_UUID: 6a722ca3-aa68-11ea-a983-0242ac110004
             Master_Info_File: /var/lib/mysql/master.info
                    SQL_Delay: 0
          SQL_Remaining_Delay: NULL
      Slave_SQL_Running_State: Slave has read all relay log; waiting for more updates
           Master_Retry_Count: 86400
                  Master_Bind:
      Last_IO_Error_Timestamp:
     Last_SQL_Error_Timestamp:
               Master_SSL_Crl:
           Master_SSL_Crlpath:
           Retrieved_Gtid_Set:
            Executed_Gtid_Set:
                Auto_Position: 0
         Replicate_Rewrite_DB:
                 Channel_Name:
           Master_TLS_Version:
1 row in set (0.00 sec)
```



重点看着两个选项要为 `YES`

```sql
Slave_IO_Running: Yes
Slave_SQL_Running: Yes
```

