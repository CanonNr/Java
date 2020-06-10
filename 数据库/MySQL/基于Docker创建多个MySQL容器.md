# 前言

先说明一下其实使用**虚拟机**或者**物理机**再或者**Docker**都差不多，开发环境怎么方便怎么来，先上环境根据实际情况而定。

介绍下环境：Win 10 + Docker Desktop

# 配置内部网络

多节点肯定是需要网络的互通，为了方便我这定义了一个`network`

```shell
docker network create --driver bridge  mynet
```

# 构建

```shell
# 节点一
docker run -p 5501:3306 --name MySQL.Cluster.Node1 /
 --network=mynet --restart=always /
-v "D:\Docker\MySQL.Cluster\Node1\conf":/etc/mysql /
-v "D:\Docker\MySQL.Cluster\Node1\logs":/var/log/mysql  /
-v "D:\Docker\MySQL.Cluster\Node1\data":/var/lib/mysql -e MYSQL_ROOT_PASSWORD=root /
-d mysql:5.7

# 节点二
docker run -p 5502:3306 --name MySQL.Cluster.Node2 /
 --network=mynet --restart=always /
-v "D:\Docker\MySQL.Cluster\Node2\conf":/etc/mysql /
-v "D:\Docker\MySQL.Cluster\Node2\logs":/var/log/mysql  /
-v "D:\Docker\MySQL.Cluster\Node2\data":/var/lib/mysql -e MYSQL_ROOT_PASSWORD=root /
-d mysql:5.7

# 节点N
...
```



# 测试一下网络

首先测试一下网络通了吗

- 进入到容器内部

  ```shell
  docker exec -it MySQL.Cluster.Node1 /bin/bash
  ```


- 连接`MySQL`

  ```shell
  mysql -uroot -p -h MySQL.Cluster.Node2
  ```

# 配置文件

现在的版本已经不会给你一个默认的`.cfg`文件了，需要自己到指定目录创建一个`my.cfg`。

- 节点一

  ```shell
  [mysqld]
  log-bin=mysql-bin    #[必须]启用二进制日志
  server-id=1          #[必须]服务器唯一ID，默认是1，一般取IP最后一段，这里看情况分配
  ```

- 节点二

  ```shell
  [mysqld]
  log-bin=mysql-bin    #[必须]启用二进制日志
  server-id=2          #[必须]服务器唯一ID，默认是1，一般取IP最后一段，这里看情况分配
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



​		

