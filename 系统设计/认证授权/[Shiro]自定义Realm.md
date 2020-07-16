# Shiro 中自定义Realm

> 前文中讲到定义了一个登录的控制器，还没有写账号密码的判定。



找到之前预留到的一个"锚点"  `UserRealm`

```java
// 自定义一个 Realm
public class UserRealm extends AuthorizingRealm {
    // 执行授权逻辑
    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principalCollection) {
        System.out.println("执行授权逻辑");
        return null;
    }

    // 执行认证逻辑
    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken authenticationToken) throws AuthenticationException {
        System.out.println("执行认证逻辑");
        return null;
    }
}
```



## 认证逻辑

### 代码结构

```java
// 执行认证逻辑
@Override
protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken authcToken) throws AuthenticationException {
	// 获取到username
    
    // 根据username查询数据库
    
    // 如果没有查到则返回null
    
    // 查询到则返回
}

    
```



> 首先在控制器中会有一个`new UsernamePasswordToken(username, password)`的操作

### 获取控制器传来的用户账号

- 方法一

  ```java
  String username = (String) authcToken.getPrincipal();
  ```

  

- 方法二

  ```java
  UsernamePasswordToken user = (UsernamePasswordToken) authcToken;
  System.out.println("用户名：" + user.getUsername());
  
  // 这样可以获取到用户名但是套娃并不能获取到密码
  user.getPassword();  // 不能获取到密码!
  ```

  

- 方法三

  ```java
  // 可以获取到用户名和密码
  UsernamePasswordToken token = (UsernamePasswordToken) authcToken;
  String username = token.getUsername().trim();
  String password = String.valueOf(token.getPassword());
  ```



### 数据库查询

> 使用了MyBatis-Plus

```java
public interface AdministratorsService extends IService<Administrators> {
    public Administrators getByUsername(String username);
}
```



```java
@Service
public class AdministratorsServiceImpl extends ServiceImpl<AdministratorsMapper, Administrators> implements AdministratorsService {
    public Administrators getByUsername(String username){
        QueryWrapper<Administrators> ew = new QueryWrapper<Administrators>();
        ew.eq("username",username);
        return this.getOne(ew);
    }
}
```





### 最终代码

```java
@Override
protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken authcToken) throws AuthenticationException {
    UsernamePasswordToken token = (UsernamePasswordToken) authcToken;
    String username = token.getUsername().trim();
    String password = String.valueOf(token.getPassword());

    Administrators administrator = administratorsService.getByUsername(username);
    if (administrator != null){
        // 加密方式: 首先每个用户有独自的盐   md5(md5(明文密码).盐)
        String passwordMd5 = DigestUtils.md5DigestAsHex(password.getBytes());
        String s = DigestUtils.md5DigestAsHex((passwordMd5 + administrator.getSalt()).getBytes());
        if (administrator.getPassword().equals(s)){
            return new SimpleAuthenticationInfo(administrator,password,this.getName());
        }
    }
    return null;
}
```





### 控制器

```java
@RequestMapping(value = "/login",method = RequestMethod.GET)
public CommonResult login(AdminLoginParam adminLoginParam){
    // 获取Subject
    Subject subject = SecurityUtils.getSubject();
    // 封装用户数据
    UsernamePasswordToken token = new UsernamePasswordToken(adminLoginParam.getUsername(), adminLoginParam.getPassword());
    // 登录
    try {
        subject.login(token);
    } catch (Exception e){
        return CommonResult.validateFailed("用户名或密码错误");
    }

    return CommonResult.success(subject.getPrincipal());
}
```



### 测试

- 正确

  `http://127.0.0.1:8800/login?username=admin&password=123456`

  ```json
  {
      "code": 200,
      "message": "操作成功",
      "data": {
          "username": "admin",
          "password": "77d3b7ed9db7d236b9eac8262d27f6a5",
          "salt": "123",
          "nickname": "超级管理员",
          "email": "admin@lkschool.com",
          "profile": null,
          "telephone": "15866666666",
          "createDate": "2020-07-05T14:20:32"
      }
  }
  ```

- 错误

  `http://127.0.0.1:8800/login?username=admin&password=admin`

  ```json
  {
      "code": 402,
      "message": "用户名或密码错误",
      "data": null
  }
  ```

  