# 前言

先说明一下其实使用**虚拟机**或者**物理机**再或者**Docker**都差不多，开发环境怎么方便怎么来，先上环境根据实际情况而定。

介绍下环境：Win 10 + Docker Desktop

# 配置内部网络

多节点肯定是需要网络的互通，为了方便我这定义了一个`network`

```shell
docker network create --driver bridge  net
```

# 构建

```shell
# 节点一
docker run -p 5501:3306 --name MySQL.Cluster.Node1 /
--network=net --restart=always /
-v "D:\Docker\MySQL.Cluster\Node1\conf":/etc/mysql /
-v "D:\Docker\MySQL.Cluster\Node1\logs":/var/log/mysql  /
-v "D:\Docker\MySQL.Cluster\Node1\data":/var/lib/mysql -e MYSQL_ROOT_PASSWORD=root /
-d mysql:5.7

# 节点二
docker run -p 5502:3306 --name MySQL.Cluster.Node2 /
--network=net --restart=always /
-v "D:\Docker\MySQL.Cluster\Node2\conf":/etc/mysql /
-v "D:\Docker\MySQL.Cluster\Node2\logs":/var/log/mysql  /
-v "D:\Docker\MySQL.Cluster\Node2\data":/var/lib/mysql -e MYSQL_ROOT_PASSWORD=root /
-d mysql:5.7

# 节点N
...
```



# 查看列表

```shell
docker network ls

NETWORK ID          NAME                DRIVER              SCOPE
76e8336076fe        bridge              bridge              local
1ea033600e79        host                host                local
2246b4049a27        net                 bridge              local
1504988411f1        none                null                local
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


