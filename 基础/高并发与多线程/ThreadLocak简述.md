# 概念

ThreadLocal提供了线程的局部变量，每个线程都可以通过`set()`和`get()`来对这个局部变量进行操作，但不会和其他线程的局部变量进行冲突，**实现了线程的数据隔离**。

简要言之：往ThreadLocal中填充的变量属于**当前**线程，该变量对其他线程而言是隔离

# 用法

```java
// 实例化一个
ThreadLocal<String> threadLocal = new ThreadLocal<String>();

// 在不同的线程执行
String s = threadLocal.get();
System.out.println(s);
threadLocal.set("hello");
```

# 实例

```java
static ThreadLocal<String> threadLocal = new ThreadLocal<String>();
public static void main(String[] args) {
    Thread t1 = new Thread(new Runnable() {
        @Override
        public void run() {
            String s = threadLocal.get();
            System.out.println(s);
            threadLocal.set("baba");
            s = threadLocal.get();
            System.out.println(s);
        }
    });

    Thread t2 = new Thread(new Runnable() {
        @Override
        public void run() {
            String s = threadLocal.get();
            System.out.println(s);
            threadLocal.set("niuniu");
            s = threadLocal.get();
            System.out.println(s);
        }
    });

    t1.start();
    t2.start();
}

// 控制台输出结果
null
null
baba
niuniu
```

得到结论：使用`ThreadLocal`设置变量仅本线程可读取

# 原理

```java
public void set(T value) {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null)
        map.set(this, value);
    else
        createMap(t, value);
}
```

在`ThreadLocal`类中有个`get`，可以看出核心使用过的是`Map`一个**Key**、**Value**键值对的形式进行存储。

所以当存储多个值的时候他是会使用`Thread`作为key去查找