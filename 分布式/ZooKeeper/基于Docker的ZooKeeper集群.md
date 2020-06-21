# 构建

- **Node 1**

  ```shell
  docker run -d -p 2181:2181 --name zookeeper_node1 --privileged --restart always --network=net \
  -v "D:\Docker\ZooKeeper.Cluster\Node1\data":/data \
  -v "D:\Docker\ZooKeeper.Cluster\Node1\datalog":/datalog  \
  -v "D:\Docker\ZooKeeper.Cluster\Node1\logs":/logs  \
  -e ZOO_MY_ID=1  \
  -e "ZOO_SERVERS=server.1=zookeeper_node1:2888:3888 server.2=zookeeper_node2:2888:3888 server.3=zookeeper_node3:2888:3888" zookeeper:3.4.11 
  ```

- **Node 2**

  ```shell
  docker run -d -p 2182:2181 --name zookeeper_node2 --privileged --restart always --network=net \
  -v "D:\Docker\ZooKeeper.Cluster\Node2\data":/data \
  -v "D:\Docker\ZooKeeper.Cluster\Node2\datalog":/datalog  \
  -v "D:\Docker\ZooKeeper.Cluster\Node2\logs":/logs  \
  -e ZOO_MY_ID=2  \
  -e "ZOO_SERVERS=server.1=zookeeper_node1:2888:3888 server.2=zookeeper_node2:2888:3888 server.3=zookeeper_node3:2888:3888" zookeeper:3.4.11 
  ```

  

- **Node 3**

  ```shell
  docker run -d -p 2183:2181 --name zookeeper_node3 --privileged --restart always --network=net \
  -v "D:\Docker\ZooKeeper.Cluster\Node3\data":/data \
  -v "D:\Docker\ZooKeeper.Cluster\Node3\datalog":/datalog  \
  -v "D:\Docker\ZooKeeper.Cluster\Node3\logs":/logs  \
  -e ZOO_MY_ID=3  \
  -e "ZOO_SERVERS=server.1=zookeeper_node1:2888:3888 server.2=zookeeper_node2:2888:3888 server.3=zookeeper_node3:2888:3888" zookeeper:3.4.11 
  ```



> ✔ `2888` 和 `3888` 更多用于内部的沟通所以没有映射到外面。
>
> ✔ 自定义了一个网卡`--network=net`
>
> ✔  指定了 `zookeeper:3.4.11 ` 的版本（因为用新版报错）

# 服务状态

> 以 Node 1 为例

```shell
/zookeeper-3.4.11/bin # ./zkServer.sh
ZooKeeper JMX enabled by default
Using config: /conf/zoo.cfg
Usage: ./zkServer.sh {start|start-foreground|stop|restart|status|upgrade|print-cmd}
/zookeeper-3.4.11/bin # ./zkServer.sh status
ZooKeeper JMX enabled by default
Using config: /conf/zoo.cfg
Mode: leader
```





# 测试数据一致性

## 在 Node 1 中

```shell
[zk: localhost:2181(CONNECTED) 2] create /test "hello"
Created /test
[zk: localhost:2181(CONNECTED) 3] ls /
[baba, zookeeper, test]
```



## 到 Node 2 中

```shell
[zk: localhost:2181(CONNECTED) 4] ls /
[baba, zookeeper, test]
```

