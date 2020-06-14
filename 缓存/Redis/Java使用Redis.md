# 创建 Redis 连接

```java
@Test
public void Test() {
    // 连接本地的 Redis 服务
    Jedis jedis = new Jedis("localhost");
    // 密码的验证
    jedis.auth("123456"); // 验证成功会返回字符串的 OK 
    // 查看服务是否运行
    System.out.println("服务正在运行: "+jedis.ping());
}


```



# 字符串实例

```java
String result = jedis.set("lksun", "唧唧复唧唧");
// 成功则得到结果 OK
System.out.println(result); 

String lksun = jedis.get("lksun");
// 输出 唧唧复唧唧
System.out.println(lksun);
```

# List 实例

```java
jedis.lpush("lksun","沧海桑田珠有泪");
jedis.lpush("lksun","蓝田日暖玉生烟");
jedis.lpush("lksun","此情可待成追忆");
jedis.lpush("lksun","只是当时已惘然");
List<String> list = jedis.lrange("lksun", 0, jedis.llen("lksun"));
System.out.println(list);
// 得到结果
[只是当时已惘然, 此情可待成追忆, 蓝田日暖玉生烟, 沧海桑田珠有泪]
```



# Map 实例

```java
HashMap<String, String> map = new HashMap<String, String>();
map.put("syr","18");
map.put("55open","nb");
jedis.hmset("lksun",map);
Map<String, String> lksun = jedis.hgetAll("lksun");
System.out.println(lksun);

// 得到结果
{syr=18, 55open=nb}
```



# Set 实例

```java
jedis.sadd("lksun","sun");
jedis.sadd("lksun","uzi");
jedis.sadd("lksun","white");
Set<String> lksun = jedis.smembers("lksun");
System.out.println(lksun);

// 得到结果
[white, uzi, sun]
```



# Sorted Set 实例

```java
jedis.zadd("Lksun_sorted_set",100,"sun");
jedis.zadd("Lksun_sorted_set",200,"white");
jedis.zadd("Lksun_sorted_set",300,"uzi");
Set<Tuple> zrank = jedis.zrangeWithScores("Lksun_sorted_set", 0,-1);
System.out.println(zrank);
// 得到结果
[[sun,100.0], [white,200.0], [uzi,300.0]]
```





# BitMap 实例

```java
Boolean lksun1 = jedis.setbit("lksun", 1, "1");
System.out.println(lksun1);
// 得到结果
true
    
Boolean lksun2 = jedis.setbit("lksun", 2, "0");
System.out.println(lksun2);
// 得到结果
false
    
Boolean lksun3 = jedis.getbit("lksun", 1);
System.out.println(lksun3);
// 得到结果
true
```

