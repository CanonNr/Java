## java

`MyBatis` 内置了一个强大的事务性查询缓存机制，它可以非常方便地配置和定制。 为了使它更加强大而且易于配置，我们对 MyBatis 3 中的缓存实现进行了许多改进。

默认情况下，只启用了本地的会话缓存，它仅仅对一个会话中的数据进行缓存。 要启用全局的二级缓存，只需要在你的 SQL 映射文件中添加一行：

```
<cache/>
```

当添加上该标签之后，会有如下效果：

- 映射语句文件中的所有 select 语句的结果将会被缓存。
- 映射语句文件中的所有 insert、update 和 delete 语句会刷新缓存。
- 缓存会使用最近最少使用算法（LRU, Least Recently Used）算法来清除不需要的缓存。
- 缓存不会定时进行刷新（也就是说，没有刷新间隔）。
- 缓存会保存列表或对象（无论查询方法返回哪种）的 1024 个引用。
- 缓存会被视为读/写缓存，这意味着获取到的对象并不是共享的，可以安全地被调用者修改，而不干扰其他调用者或线程所做的潜在修改。

在进行配置的时候还会分为一级缓存和二级缓存：

一级缓存：线程级别的缓存，是本地缓存，sqlSession级别的缓存

二级缓存：全局范围的缓存，不止局限于当前会话

## 一级缓存

一级缓存是`sqlsession`级别的缓存，**默认是存在的**。发送两个相同的请求，但是sql语句仅仅执行了一次，那么就意味着第一次查询的时候已经将结果进行了缓存。

### 示例

```java
@Test
public void test01(){
    SqlSession sqlSession = sqlSessionFactory.openSession();
    UserDao mapper = sqlSession.getMapper(UserDao.class);

    // 第一次查询
    List<User> users = mapper.getUserByName("admin");
    System.out.println(users);

    System.out.println("---------- 这是一个分割线 ----------");

    // 第二次查询
    List<User> users1 = mapper.getUserByName("admin");
    System.out.println(users1);

    sqlSession.commit();
    sqlSession.close();
}


// 输出结果
DEBUG [main] - ==>  Preparing: SELECT * FROM users WHERE name = ? 
DEBUG [main] - ==> Parameters: admin(String)
TRACE [main] - <==    Columns: id, name, password, email, address, birthday
TRACE [main] - <==        Row: 1, admin, 123123, 717437709@qq.com, 山东济宁, 1996-10-14
DEBUG [main] - <==      Total: 1
[User{id=1, hashid='null', status=null, name='admin', email='717437709@qq.com', password='123123', photo='null'}]
---------- 这是一个分割线 ----------
[User{id=1, hashid='null', status=null, name='admin', email='717437709@qq.com', password='123123', photo='null'}]



```



### 分析

结果还是很显然易见的，同一个方法中同一个SQL执行两次只有第一次的真正的查询了MySQL。

我犯的傻：不要多次编译运行去测试，要同一个查询差两次



### 怎样是无法使用一级缓存

- 一级缓存是`sqlSession`级别的缓存，如果在应用程序中只有开启了多个sqlsession，会造成缓存失效

  ```java
  @Test
  public void test01(){
      // 第一次查询
      SqlSession sqlSession = sqlSessionFactory.openSession();
      UserDao mapper = sqlSession.getMapper(UserDao.class);
      List<User> users = mapper.getUserByName("admin");
      System.out.println(users);
  
      System.out.println("---------- 这是一个分割线 ----------");
  
      // 第二次查询
      SqlSession sqlSession1 = sqlSessionFactory.openSession();
      UserDao mapper1 = sqlSession1.getMapper(UserDao.class);
      List<User> users1 = mapper1.getUserByName("admin");
      System.out.println(users1);
  
      sqlSession.commit();
      sqlSession.close();
  }
  
  // 输出结果
  DEBUG [main] - ==>  Preparing: SELECT * FROM users WHERE name = ? 
  DEBUG [main] - ==> Parameters: admin(String)
  TRACE [main] - <==    Columns: id, name, password, email, address, birthday
  TRACE [main] - <==        Row: 1, admin, 123123, 717437709@qq.com, 山东济宁, 1996-10-14
  DEBUG [main] - <==      Total: 1
  [User{id=1, hashid='null', status=null, name='admin', email='717437709@qq.com', password='123123', photo='null'}]
  ---------- 这是一个分割线 ----------
  DEBUG [main] - ==>  Preparing: SELECT * FROM users WHERE name = ? 
  DEBUG [main] - ==> Parameters: admin(String)
  TRACE [main] - <==    Columns: id, name, password, email, address, birthday
  TRACE [main] - <==        Row: 1, admin, 123123, 717437709@qq.com, 山东济宁, 1996-10-14
  DEBUG [main] - <==      Total: 1
  [User{id=1, hashid='null', status=null, name='admin', email='717437709@qq.com', password='123123', photo='null'}]
  Picked up JAVA_TOOL_OPTIONS: -Dfile.encoding=UTF-8
  
  Process finished with exit code 0
  ```

  

  和上一个案例不同的在于  `SqlSession sqlSession = sqlSessionFactory.openSession(); ` 创建了两个，两次查询用了不同的会话

  

- 在编写查询的sql语句的时候，一定要注意传递的参数，如果参数不一致，那么也不会缓存结果



- 如果在发送过程中发生了数据的修改，那么结果就不会缓存

  假设缓存了`name = admin`的数据，随后修改了该数据的`password`字段，会导致缓存丢失

- 在两次查询期间，手动去清空缓存，也会让缓存失效

  ` sqlSession.clearCache()`

  

## 二级缓存

二级缓存是全局作用域缓存，默认是不开启的，需要手动进行配置。

`Mybatis`提供二级缓存的接口以及实现，缓存实现的时候要求实体类实现Serializable接口，二级缓存在sqlSession关闭或提交之后才会生效。

### 如何开启

- 全局配置文件中开启缓存

  ```xml
   <setting name="cacheEnabled" value="true"/>
  ```

- 需要在使用二级缓存的映射文件出使用<cache/>标签标注

  ```xml
  <mapper namespace="dao.UserDao">
      <cache/>
  </mapper>
  ```

### 示例

```java
// 第一次查询
SqlSession sqlSession1 = sqlSessionFactory.openSession();
UserDao mapper1 = sqlSession1.getMapper(UserDao.class);
User user1 = mapper1.getUserById(1);
System.out.println(user1);

// 当提交事务时，sqlSession1查询完数据后，sqlSession2相同的查询是否会从缓存中获取数据。
sqlSession1.close();
System.out.println("---------- 这是一个分割线 ----------");

// 第二次查询
SqlSession sqlSession2 = sqlSessionFactory.openSession();
UserDao mapper2 = sqlSession2.getMapper(UserDao.class);
User user2 = mapper2.getUserById(1);
System.out.println(user2);

// 返回结果
DEBUG [main] - Cache Hit Ratio [dao.UserDao]: 0.0
DEBUG [main] - ==>  Preparing: SELECT * FROM users WHERE id = 1 
DEBUG [main] - ==> Parameters: 
TRACE [main] - <==    Columns: id, name, password, email, address, birthday
TRACE [main] - <==        Row: 1, admin, 123123, 717437709@qq.com, 山东济宁, 1996-10-14
DEBUG [main] - <==      Total: 1
User{id=1, hashid='null', status=null, name='admin', email='717437709@qq.com', password='123123', photo='null'}
---------- 这是一个分割线 ----------
DEBUG [main] - Cache Hit Ratio [dao.UserDao]: 0.5
User{id=1, hashid='null', status=null, name='admin', email='717437709@qq.com', password='123123', photo='null'}
```

###  Cache Hit Ratio 

仔细看发现日较一级缓存多了一行 意思是 缓存命中率
开启二级缓存后，每执行一次查询，系统都会计算一次二级缓存的命中率。第一次查询也是先从缓存中查询，只不过缓存中一定是没有的。所以会再从DB中查询。由于二级缓存中不存在该数据，所以命中率为0.但第二次查询是从二级缓存中读取的，所以这一次的命中率为1/2=0.5。当然，若有第三次查询，则命中率为1/3=0.66



## 参考文章

[聊聊MyBatis缓存机制 - 美团技术团队](https://tech.meituan.com/2018/01/19/mybatis-cache.html)

[Mybatis教程-实战看这一篇就够了](https://blog.csdn.net/hellozpc/article/details/80878563)