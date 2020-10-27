# Docker Api 编程

**简单来说就是用代码执行`Docker`命令**

## 文档

[Docker docs](https://docs.docker.com/engine/api/sdk/)

[SDK](https://godoc.org/github.com/docker/docker/client)



## 开始

```go
// 具体代码按照官方文档即可
```



## 坑

### 第一大坑

如果按照官方文档直接执行`go get github.com/docker/docker/client`会找不到`client.NewClientWithOpts`方法

所以执行`go get github.com/docker/docker@master`