# Spring Cloud Config - 分布式微服务配置中心

项目越来越多配置文件也越来越难以管理，如果某一个需要修改可能要修改N多个，繁琐而且容易出错。

`Spring Cloud Config`为微服务提供了 **集中化的外部配置支持**，配置服务器为所有应用提供了一个中心化的外部部署。



## 集成

### 引入包

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-config-server</artifactId>
</dependency>
```



### 配置

- `bootstrap.yml`

  ```yaml
  spring:
    application:
      name: config-single-server  # 应用名称
    cloud:
       config:
          server:
            git:
              uri: https://github.com/huzhicheng/config-only-a-demo #配置文件所在仓库
              username: github 登录账号
              password: github 登录密码
              default-label: master #配置文件分支
              search-paths: config  #配置文件所在根目录
  ```

  

- `application.yml`

  ```yaml
  server:
    port: 4455
  ```

  

### 启动类

```java
@SpringBootApplication
@EnableConfigServer
public class ConfigMain {
    public static void main(String[] args) {
        SpringApplication.run(ConfigMain.class,args);
    }
}
```



### 远程仓库

前往`GitHub`、`GitLab`、`Gitee`等创建一个仓库。

仓库存几个配置文件

![image-20200913014400291](../../image/image-20200913014400291.png)

```yaml
data:
  title: "Config From Git"
```



### 访问

`http://127.0.0.1:4455/service-user.yml`

浏览器中得到

![image-20200913014456124](../../image/image-20200913014456124.png)