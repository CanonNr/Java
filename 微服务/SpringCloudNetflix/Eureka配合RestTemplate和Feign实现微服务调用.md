# Eureka配合RestTemplate和Feign实现微服务调用

在讲服务注册到`Eureka`后，配合`Ribbon`即可实现负载均衡，即动态的去调用服务。

本文会通过`RestTemplate` 和 `Feign` 分别实现

## 准备工作

> 模拟一个场景：订单模块根据用户的`id`到用户模块查询用户信息。
>
> 所需食材：
>
> - 一个`Eureka`服务端，端口为`7001`
> - 两个用户模块且已注册在`Eureka`中，端口分别为`8001`和`8002`
>

### 创建一个订单模块 

> 初始化一个Spring Boot 项目，此处省略...

### 引入相关依赖

```xml
<!-- 引入 Eureka 客户端-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
<!-- 如果项目版本较老则引入 -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-eureka</artifactId>
</dependency>

<!-- feign 客户端 -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```



### 配置文件

```yaml
server:
  port: 9021
eureka:
  client:
    # 是否将自己注册到 Eureka ， 如果单纯是客户端 不存在被其他项目调用则为false
    register-with-eureka: false
    # 扫描注册，因为是客户端肯定需要在Eureka中扫描 所以为true
    fetch-registry: true
    service-url:
      # Eureka 服务端地址，如果为集群可以写多个,通过英文逗号(,)隔开
      defaultZone: http://euk1.local:7001/eureka/
```



### 开启Eureka 注解

> - 启动类中加入 `@EnableEurekaClient` (我使用的 2.2.5 版本不加也可以)
>
> - 启动类中加入`@EnableFeignClients`

```java
@SpringBootApplication
@EnableEurekaClient
@EnableFeignClients
public class OrderMain9021 {
    public static void main(String[] args) {
        SpringApplication.run(OrderMain9021.class,args);
    }
}
```



## 写一个远程的服务端

```java
@RestController
public class TestController {
    @Value("${server.port}")
    public String port;

    @RequestMapping("/user")
    public CommonResult get(Integer id){
        GetUser user = new GetUser();
        return new CommonResult(200,this.port,user.getUserById(id));
    }
    
    public User getUserById(Integer id){
        // 测试 临时写死了
        User user = new User();
        user.setId(id);
        user.setName("周杰伦");
        user.setSex("男");
        user.setAge(18);
        return user;
    }
}
```

## RestTemplate


### 注入Spring容器

```java
@Bean
@LoadBalanced
public RestTemplate getRT(){
    RestTemplate restTemplate = new RestTemplate();
    // 防止中文返回乱码
    restTemplate.getMessageConverters().set(1, new StringHttpMessageConverter(StandardCharsets.UTF_8));
    return restTemplate;
}
```

> `@LoadBalanced` 很关键，如果你使用的是`Eureka` 只需要使用服务名即可调用则必须加入该注解。如：`http://user-service/user/1`
>
> 如果只是普通的 Http 调用则无需添加例如 `http://192.168.1.1/user/1`

### 控制器

```java
@RequestMapping(value = "rt",method = RequestMethod.GET)
public String test2(@RequestParam(value = "id",defaultValue = "1") String id){
    System.out.println("RestTemplate");
    // 此处 第一个user是Eureka中的模块名,第二个user是路由路径
    // ?后面的id则为传参
    String url = "http://user/user?id="+id;
    return restTemplate.getForObject(url, String.class);
}
```



### 结果

- `http://127.0.0.1:9021/test/rt?id=4`

  ```json
  {"code":200,"message":"8001","data":{"id":4,"name":"周杰伦","sex":"男","age":18}}
  ```

- `http://127.0.0.1:9021/test/rt?id=10`

  ```json
  {"code":200,"message":"8002","data":{"id":4,"name":"周杰伦","sex":"男","age":18}}
  ```

  

## Feign

`Feign` 最大的优点可能就是在控制器省去 `Url`，直接使用方法即可实现不同接口的调用。

### 创建一个Interface

```java
// user即Eureka中的服务名
@FeignClient("user")
public interface UserClient {
    // 此 user 为远程服务的路由地址
    @RequestMapping(value = "/user",method = RequestMethod.GET)
    // @RequestParam 用于给远程服务传参
    String get(@RequestParam(value = "id") Integer id);
}
```



### 控制器

```java
@RequestMapping(value = "fg",method = RequestMethod.GET)
public String test1(@RequestParam(value = "id",defaultValue = "1") Integer id){
    return client.get(id);
}
```



### 结果

- `http://127.0.0.1:9021/test/fg?id=1`

  ```json
  {"code":200,"message":"8002","data":{"id":1,"name":"周杰伦","sex":"男","age":18}}
  ```

- `http://127.0.0.1:9021/test/fg?id=9`

  ```json
  {"code":200,"message":"8001","data":{"id":9,"name":"周杰伦","sex":"男","age":18}}
  ```