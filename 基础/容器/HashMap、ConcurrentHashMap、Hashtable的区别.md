## HashMap 

作为日常开发最常用的存储结构，前面对其特性做了大量介绍，所以本文不再多讲，主要对比与`HashTable`和`ConcurrentHashMap`区别



## HashTable

### 空键值

```java
// HashMap
HashMap<Object, Object> map = new HashMap<>();
map.put(null,10086);
System.out.println(map);
// 输出结果
{null=10086}

// HashTable
Hashtable<Object, Object> map = new Hashtable<>();
map.put(null,10086);
map.put(10086,null);
System.out.println(map);
// 输出结果
Exception in thread "main" java.lang.NullPointerException
	at java.util.Hashtable.put(Hashtable.java:465)
	at map.demo2.main(demo2.java:8)
// 无论是键或值设置为 null 都会报错
```

总结：`HashTable`的键或值都不能为`null`



### 线程安全

在`HashTable`的源码中总会找到`synchronized`，这个修饰前面是遇到过的：带有`synchronized`修饰的`StringBuffer`类是**线程安全**的，不带有的修饰的`StringBuilder`是线程不安全的。

下面截取两段`HashTable`的源码

```java
public synchronized int size() {
    return count;
}

public synchronized boolean isEmpty() {
    return count == 0;
}
```



## ConcurrentHashMap

### 空键值

```java
ConcurrentHashMap map = new ConcurrentHashMap();
map.put(null,123);
System.out.println(map);
// 首先是测试一下空的键能否运行
Exception in thread "main" java.lang.NullPointerException
	at java.util.concurrent.ConcurrentHashMap.putVal(ConcurrentHashMap.java:1011)
	at java.util.concurrent.ConcurrentHashMap.put(ConcurrentHashMap.java:1006)
	at map.demo3.main(demo3.java:8)
```

在空键值上`ConcurrentHashMap`更像是`HashTable`

### 线程安全

`Hashtable`中采用的锁机制是一次锁住整个 **hash表**，从而同一时刻只能由一个线程对其进行操作；
而ConcurrentHashMap中则是一次锁住一个桶。

ConcurrentHashMap默认将hash表分为16个桶，诸如get,put,remove等常用操作只锁当前需要用到的桶。
这样，原来只能一个线程进入，现在却能同时有16个写线程执行，并发性能的提升是显而易见的。