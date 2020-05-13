## 关于占位符

在前面 `jdbc` 中有使用过占位符，核心是在于帮助解决`SQL注入`的攻击，代码如下：

```java
// 假设定义一个 查询字段
// 1 or 1=1
Class.forName("java.sql.DriverManager");
Connection con = DriverManager.getConnection("jdbc:mysql://localhost/demo?useUnicode=true&characterEncoding=UTF-8&serverTimezone=Asia/Shanghai", "root", "root");
// 此处使用了 ？ 作为占位符
String sql = "select * from users where id = ?";
// 引入 prepareStatement
PreparedStatement preparedStatement = con.prepareStatement(sql);
// 为占位符赋值
// 1 表示下标 ; 从 1 开始 , 不是0 
preparedStatement.setString(1,id);

```



而在 `MyBatis` 中：

`MyBatis` 启用了**预编译功能**，在SQL执行前，会先将上面的SQL发送给数据库进行编译；执行时，直接使用**编译好的SQL**，替换**占位符** **?** 就可以了。因为**SQL注入**只能对**编译过程**起作用，所以这样的方式就很好地**避免了SQL注入的问题**。



## 如何使用

随便揪一段 `Mapper` 配置

```xml
<!--
 	这一段是通过一个学生的 ID 查询到具体的学生信息
	此处使用到了占位符 #{id}
-->
<select id="findStudentById" resultType="entity.Student">
        SELECT * FROM students WHERE id = #{id}
</select>
```



##  `#{}` 和` ${}`

查看文档和一些文章的时候不难发现除了 `#{id}` 这可以这么写`${id}`



### 区别

- `#{}` 只是替换 `？`，相当于`PreparedStatement`使用占位符去替换参数，可以防止sql注入。
- `${} `是进行字符串拼接，相当于sql语句中的`Statement`，使用字符串去拼接sql；$可以是sql中的任一部分传入到Statement中，不能防止sql注入。



## 如何选择

- 一般能用#的就别用$
- 在`order by`、表名时选择 `$`