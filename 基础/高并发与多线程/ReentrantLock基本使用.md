

# 概述



java除了使用关键字`synchronized`外，还可以使用`ReentrantLock`实现独占锁的功能。而且`ReentrantLock`相比`synchronized`而言功能更加丰富，使用起来更为灵活，也更适合复杂的并发场景。

而`ReentrantLock`正是实现了`Lock`接口的其中一个。

```java
public class ReentrantLock implements Lock, java.io.Serializable {
    
}
```



# 代码实例

```java
static int count = 0;
static CountDownLatch downLatch;
static Lock lock = new ReentrantLock();
static void m (){
    try {
        lock.lock();
        count++;
    }finally {
        lock.unlock();
    }
}

public static void main(String[] args) throws InterruptedException {
    downLatch = new CountDownLatch(10000);
    for (int i = 0; i < 10000; i++) {
        new Thread(new Runnable() {
            public void run() {
                try {
                    m();
                }finally {
                    downLatch.countDown();
                }
            }
        }).start();
    }
    downLatch.await();
    System.out.println(count);
}
```



一个很简单的demo：创建了10000个线程对`count`累加，最终我们想要的结果是 10000.

# 区别

- 首先`synchronized`是java内置关键字，在jvm层面，而`Lock`是个 java 类；
- `synchronized`无法判断是否获取锁的状态，`Lock`可以判断是否获取到锁；
- `synchronized`会自动释放锁，`lock`需在`finally`中手工释放锁，否则容易造成线程死锁；
- `synchronized`的锁可重入、不可中断、非公平，而`lock`锁可重入、可判断、可公平（两者皆可）
- `synchronized`不可响应中断，一个线程获取不到锁就一直等着；`lock`可以相应中断。

# 如何选择

- `lock`锁适合大量同步的代码的同步问题，`synchronized` 锁适合代码少量的同步问题。
- 需要高度灵活的代码建议使用`ReentrantLock`