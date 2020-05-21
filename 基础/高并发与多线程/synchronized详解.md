# 概念

`synchronized` 是 Java 中的关键字，是利用锁的机制来实现同步的。

锁机制有如下两种特性：

- 互斥性：**在同一时间只允许一个线程持有某个对象锁**，通过这种特性来实现多线程中的协调机制，这样在同一时间只有一个线程对需同步的代码块(复合操作)进行访问。互斥性我们也往往称为操作的原子性。
- 可见性：必须确保在锁被释放之前，对共享变量所做的修改，对于随后获得该锁的另一个线程是可见的（即在获得锁时应获得最新共享变量的值），否则另一个线程可能是在本地缓存的某个副本上继续操作从而引起不一致。

# 用法分类

## 同步代码块

```java
// 伪代码
synchronized(共享资源、共享对象，需要是object的子类){
    具体执行的代码块
}

// 实例
synchronized (this){
    if (num > 0){
        System.out.println(Thread.currentThread().getName()+"：正在出售第"+num--+"张票");
    }
}
```



## 同步方法

```java
@Override
public void run() {
    while(num > 0){
        if (num > 0){
            // 原来直接输出 现在封装一个方法
            sell();
        }
        try {
            Thread.sleep(100);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
// 用 synchronized 修饰
private synchronized void sell() {
    System.out.println(Thread.currentThread().getName()+"：正在出售第"+num--+"张票");
}
```



这两种方式得到的结果是一样的。







 未完待续。。。