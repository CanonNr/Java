# 概念

`synchronized` 是 Java 中的关键字，是利用锁的机制来实现同步的。

锁机制有如下两种特性：

- 互斥性：**在同一时间只允许一个线程持有某个对象锁**，通过这种特性来实现多线程中的协调机制，这样在同一时间只有一个线程对需同步的代码块(复合操作)进行访问。互斥性我们也往往称为操作的原子性。
- 可见性：必须确保在锁被释放之前，对共享变量所做的修改，对于随后获得该锁的另一个线程是可见的（即在获得锁时应获得最新共享变量的值），否则另一个线程可能是在本地缓存的某个副本上继续操作从而引起不一致。

# 用法

## 修饰实例方法

```java
public class Demo {
    public synchronized void get(){

    }
}
```



## 修饰静态方法

```java

public class Demo {
    public void get(){
        synchronized(Synchronized.class){

        }
    }
}
```



## 修饰代码块

```java
public class Synchronized {
    public void husband(){
        synchronized(new test()){

        }
    }
}
```

