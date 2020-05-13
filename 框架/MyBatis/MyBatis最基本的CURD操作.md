## 查

### 查一条数据

```java
// 根据 ID 查询具体某一条数据
// dao
public Student findStudentById(Integer id);

// mapper
<select id="findStudentById" resultType="entity.Student">
        SELECT * FROM students WHERE id = #{id}
</select>

// test
StudentDao mapper = sqlSession.getMapper(StudentDao.class);
Student student = mapper.findStudentById(1);

// 输出结果
Student{id=1, name='张三', sex=1, birthday=Thu Nov 06 08:00:00 CST 1997, classroom_id=1}

// 扩展
// 如果查询到不存在的数据则返回
null
```



### 查询多条数据

```java
// 查询所有数据
// dao
public List<Student> getAll();
// mapper
<select id="getAll" resultType="entity.Student">
	SELECT * FROM students
</select>
// test
List<Student> student = mapper.getAll();
System.out.println(student);
// 输出结果
[
    Student{id=1, name='张三', sex=1, birthday=Thu Nov 06 08:00:00 CST 1997, classroom_id=1}, 
    Student{id=2, name='李四', sex=0, birthday=Sat Dec 15 08:00:00 CST 1990, classroom_id=1}, 
    Student{id=3, name='王五', sex=1, birthday=Mon Oct 11 08:00:00 CST 1999, classroom_id=2}
]
```



### 查询多条数据返回 Map 类型

```java
// 查询所有数据
// dao
@MapKey("name")
public Map<String,Student> getAll1();

// mapper
<select id="getAll1" resultType="entity.Student">
	SELECT * FROM students
</select>
// test
Map<String, Student> student = mapper.getAll1();
System.out.println(student);
// 输出结果
{
    李四=Student{id=2, name='李四', sex=0, birthday=Sat Dec 15 08:00:00 CST 1990, classroom_id=1}, 
    张三=Student{id=1, name='张三', sex=1, birthday=Thu Nov 06 08:00:00 CST 1997, classroom_id=1}, 
    王五=Student{id=3, name='王五', sex=1, birthday=Mon Oct 11 08:00:00 CST 1999, classroom_id=2}
}

```



### 注解式

```java
// dao
@Select("SELECT * FROM students where name = #{name}")
public Student findStudentByName(String name);

// test
StudentDao mapper = sqlSession.getMapper(StudentDao.class);
Student student = mapper.findStudentByName("王五");
System.out.println(student);

// 输出结果
Student{id=3, name='王五', sex=1, birthday=Mon Oct 11 08:00:00 CST 1999, classroom_id=2}
```



## 增

```java
// dao
public Integer add(Student student);

// mapper
<insert id="add" >
        insert into students(name,sex,birthday,classroom_id) values (#{name},#{sex},#{birthday},#{classroom_id})
</insert>
// 输出结果
1
```





## 改

```java
// dao
Integer UpdateUserById(User user);

// mapper
<update id="UpdateUserById" >
    UPDATE users
        <trim prefix="set" suffixOverrides=",">
            <if test="name!=null">name = #{name},</if>
            <if test="password!=null">password = #{password},</if>
            <if test="email!=null">email = #{email},</if>
            <if test="address!=null">address = #{address},</if>
            <if test="birthday!=null">birthday = #{birthday},</if>
        </trim>
        WHERE id = #{id};
</update>

// test
SqlSession sqlSession = sqlSessionFactory.openSession();
UserDao mapper = sqlSession.getMapper(UserDao.class);
// 先查询到指定的 用户
User user = mapper.getUserById(1);
// 修改邮箱
user.setEmail("8888@qq.com");
// 执行修改语句
Integer integer = mapper.UpdateUserById(user);
System.out.println(integer);
sqlSession.commit();
sqlSession.close()
 
// 输出结果
1
```





## 删

```java
// dao
Integer deleteUserById(Integer id);

// mapper
<delete id="deleteUserById">
	delete from users where id = #{id}
</delete>

// test
SqlSession sqlSession = sqlSessionFactory.openSession();
UserDao mapper = sqlSession.getMapper(UserDao.class);
Integer integer = mapper.deleteUserById(9);
System.out.println(integer);
sqlSession.commit();
sqlSession.close();

// 输出结果
1
```

