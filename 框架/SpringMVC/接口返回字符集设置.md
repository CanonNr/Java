## 通过 @ResponseBody 直接向浏览器返回数据

```java
@Controller
@RequestMapping("/action")
public class ActionController {
    @ResponseBody
    @RequestMapping(value = "/index",method = RequestMethod.GET)
    public String index(User user){
        System.out.println(user.toString());
        return user.toString();
    }
}
```

## 浏览器汉字乱码问题
```java
// 方法一 , 通过responseMapping设置请求头参数 (但是每个都要添加很麻烦)
@RequestMapping(value = "/index",method = RequestMethod.GET,produces = "application/json;charset=utf-8")
```

```xml
<!-- Spring.xml-->
<!-- 方法二 , 修改配置文件-->
<!-- 写在 <mvc:annotation-driven/>之前才有效 -->
<bean class="org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter" >
	<property name="messageConverters">
		<list>
			<bean class="org.springframework.http.converter.StringHttpMessageConverter">
				<property name="supportedMediaTypes">
					<list>
						<value>text/plain;charset=utf-8</value>
						<value>text/html;charset=UTF-8</value>
					</list>
				</property>
			</bean>
		</list>
	</property>
</bean>

<mvc:annotation-driven/>
```