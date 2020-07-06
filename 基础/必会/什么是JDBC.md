## 简介

>  全称：JAVA Database Connectivity  -   java 数据库连接
>
>  SUN公司提供的一种数据库访问规则、规范, 由于数据库种类较多，并且java语言使用比较广泛，sun公司就提供了一种规范，让其他的数据库提供商去实现底层的访问规则。 我们的java程序只要使用sun公司提供的jdbc驱动即可。

## 基本使用

1. 注册驱动

   ```java
// 加载驱动
   // 当执行了当前代码之后，会返回一个Class对象，再此对象的创建过程中，会调用具体类的静态代码块
   Class.forName("java.sql.DriverManager");
   ```
   
   
   
2. 建立连接

   

   ```java
   //建立连接 参数一： 协议 + 访问的数据库 ， 参数二： 用户名 ， 参数三： 密码。
   conn = DriverManager.getConnection("jdbc:mysql://localhost/student", "root", "root");
   ```

   

3. 创建statement

   

   ```java
   // 准备静态处理块对象，将sql语句放置到静态处理块中,理解为sql语句放置对象
   // 在执行sql语句的过程中，需要一个对象来存放sql语句，将对象进行执行的时候调用的是数据库的服务，数据库会从当前对象中拿到对应的sql语句进行执行
   st = conn.createStatement();
   ```



4. 执行sql ，得到 `ResultSet`

   

   ```java
   // 写一个 SQL 语句
   String sql = "select * from student";
   
   // 执行sql语句,返回值对象是结果集合
   // 将结果放到resultset中，是返回结果的一个集合 需要经过循环迭代才能获取到其中的每一条记录
   // statement在执行的时候可以选择三种方式：
   // 1.execute:任何SQL语句都可以执行
   // 2.executeQueryL只能执行查询语句
   // 3.executeUpdate，只能执行语句
   rs = st.executeQuery(sql);
   ```



5. 遍历结果集

   

   ```java
   while(rs.next()){
       int id = rs.getInt("id");
       String name = rs.getString("name");
       int age = rs.getInt("age");
       System.out.println("id="+id + "===name="+name+"==age="+age);
   }
   
   ```



6. 释放资源

   ```java
   statement.close();
   connection.close();
   ```
   
   