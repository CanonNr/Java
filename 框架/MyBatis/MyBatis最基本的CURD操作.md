## 查

### 查一条数据

```java
// 根据 ID 查询具体某一条数据
// dao
public Student findStudentById(Integer id);

// xml
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
// xml
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
// xml
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

// xml
<insert id="add" >
        insert into students(name,sex,birthday,classroom_id) values (#{name},#{sex},#{birthday},#{classroom_id})
</insert>
// 输出结果
1
```

