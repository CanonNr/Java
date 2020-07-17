# 基于 Shiro 的权限控制

## 注解式

在需要特定权限的方法加上注解即可实现权限控制

```java
@RequestMapping(value = "",method = RequestMethod.GET)
@RequiresRoles("baba")	// 角色
@RequiresPermissions("baba")	// 权限
public String getUser(){
    // TODO
}
```

> 直接写注解是没有反应的，这个是基于AOP实现的，首先要在 Shiro 的配置文件中配置AOP

```java
@Bean
public DefaultAdvisorAutoProxyCreator advisorAutoProxyCreator() {
    DefaultAdvisorAutoProxyCreator advisorAutoProxyCreator = new DefaultAdvisorAutoProxyCreator();
    advisorAutoProxyCreator.setProxyTargetClass(true);
    return advisorAutoProxyCreator;
}

@Bean
public AuthorizationAttributeSourceAdvisor authorizationAttributeSourceAdvisor() {
    AuthorizationAttributeSourceAdvisor authorizationAttributeSourceAdvisor = new AuthorizationAttributeSourceAdvisor();
    authorizationAttributeSourceAdvisor.setSecurityManager(defaultSecurityManager);
    return authorizationAttributeSourceAdvisor;
}
```



### 自定义过滤方式

> 自定义的`Realm`类中 `doGetAuthorizationInfo`方法

```java
// 执行授权逻辑
@Override
protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principalCollection) {
    SimpleAuthorizationInfo info = new SimpleAuthorizationInfo();
    // 设置权限
    info.addStringPermission("baba");
    // 设置角色
    info.addRole("baba");
    return info;
}
```

