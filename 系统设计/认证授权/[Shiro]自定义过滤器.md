# 自定义过滤器

在`Shiro`定义了一些过滤器，可以实现一些过滤操作。

```java
public enum DefaultFilter {  
    anon(AnonymousFilter.class),  
    authc(FormAuthenticationFilter.class),  
    authcBasic(BasicHttpAuthenticationFilter.class),  
    logout(LogoutFilter.class),  
    noSessionCreation(NoSessionCreationFilter.class),  
    perms(PermissionsAuthorizationFilter.class),  
    port(PortFilter.class),  
    rest(HttpMethodPermissionFilter.class),  
    roles(RolesAuthorizationFilter.class),  
    ssl(SslFilter.class),  
    user(UserFilter.class);  
} 
```

## 在ShiroFilterFactoryBean中定义

```java
@Bean
public ShiroFilterFactoryBean getShiroFilterFactoryBean(){
	...
        
    // 自定义过滤器
    Map<String, Filter> filterMap = new HashMap<>();
    filterMap.put("jwt", new JWTFilter());
    shiroFilterFactoryBean.setFilters(filterMap);

    // 设置路由过滤规则
    HashMap<String, String> filterRuleMap  = new HashMap<>();
    // 所有路由都要经过JWT的过滤
    filterRuleMap.put("/**","jwt");
    
	...
    
}
```

## Filter 类

```java
public class JWTFilter extends BasicHttpAuthenticationFilter {

    // 首先判断请求头有没有Authorization字段
    @Override
    protected boolean isLoginAttempt(ServletRequest request, ServletResponse response) {
        HttpServletRequest req = (HttpServletRequest) request;
        String authorization = req.getHeader("Authorization");
        return authorization != null;
    }

    @Override
    protected boolean executeLogin(ServletRequest request, ServletResponse response) throws Exception {
        // 获取到header
        HttpServletRequest httpServletRequest = (HttpServletRequest) request;
        String authorization = httpServletRequest.getHeader("Authorization");
        // 将token封装到JWTToken
        JWTToken token = new JWTToken(authorization);
        // 如果没有抛出异常则代表登入成功，返回true
        getSubject(request, response).login(token);
        return true;
    }

    @Override
    protected boolean isAccessAllowed(ServletRequest request, ServletResponse response, Object mappedValue) {
        try {
            if (isLoginAttempt(request, response)) {
                executeLogin(request, response);
                return true;
            }
        }catch(Exception e) {
            // log
        }
        return false;
    }

    // 只有 isAccessAllowed() 返回 false 才会执行本方法
    // 如果继续返回 false 则表示拦截 , 返回 true 则继续处理
    @Override
    public boolean  onAccessDenied(ServletRequest request, ServletResponse response) throws IOException {
        HttpServletResponse res = (HttpServletResponse)response;
        res.setHeader("Access-Control-Allow-Origin", "*");
        res.setStatus(401);
        res.setCharacterEncoding("UTF-8");
        PrintWriter writer = res.getWriter();
        writer.write("Unauthorized");
        return false;
    }
}
```

