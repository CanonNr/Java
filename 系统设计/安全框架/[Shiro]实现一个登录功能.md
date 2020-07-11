# 通过 Shiro 实现一个登录功能

> 建议搭配 [Shiro]集成到SpringBoot 一起食用。

## 登录

```java
@RequestMapping(value = "/login",method = RequestMethod.GET)
public CommonResult login(AdminLoginParam adminLoginParam){
    System.out.println(adminLoginParam);
    // 获取Subject
    Subject subject = SecurityUtils.getSubject();
    // 封装用户数据
    UsernamePasswordToken token = new UsernamePasswordToken(adminLoginParam.getUsername(), adminLoginParam.getPassword());
    // 登录
    try {
        subject.login(token);
    } catch (Exception e){
        System.out.println("登录失败,账号或密码错误");
        return CommonResult.validateFailed("用户名或密码错误");
    }

    return CommonResult.success(adminLoginParam);
}
```

### 传输对象

```java
public class AdminLoginParam {
    
    private String username;

    private String password;

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }
}
```



### 请求

此时我们访问`http://127.0.0.1:8800/login?username=admin&password=admin`

API返回：

```json
{
    "code": 402,
    "message": "用户名或密码错误",
    "data": null
}
```



控制台输出：

```shell
# 输出接收到的参数 ， 用户名和密码都对没什么问题
AdminLoginParam(username=admin, password=admin)

# 之前在 Shiro 配置中留了个'锚点'
执行认证逻辑

# 因为没有配置ini 也没有读取数据库 所以登录失败是必然的
登录失败,账号或密码错误
```



