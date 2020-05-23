# 简述

`AtomicReference`和`AtomicInteger`非常类似，不同之处就在于`AtomicInteger`是对整数的封装，而`AtomicReference`则对应普通的对象引用。也就是它可以保证你在修改对象引用时的线程安全性。

```java
public class AtomicInteger extends Number implements java.io.Serializable {
    private static final long serialVersionUID = 6214790243416807050L;

    private static final Unsafe unsafe = Unsafe.getUnsafe();
    private static final long valueOffset;

    static {
        try {
            valueOffset = unsafe.objectFieldOffset
                (AtomicInteger.class.getDeclaredField("value"));
        } catch (Exception ex) { throw new Error(ex); }
    }

    private volatile int value;

    public AtomicInteger(int initialValue) {
        value = initialValue;
    }

    // 后面省略
}
```

复制了一部分`AtomicInteger`的源码，其中看到 `private volatile int value` ，确定了 `AtomicInteger` 底层上有用到`volatile`的修饰



# 示例代码



```java
// 定义一个初始值
private final static AtomicInteger atomicI = new AtomicInteger(0);

static CountDownLatch  countDownLatch;

public static void main(String[] args) throws InterruptedException {
    countDownLatch = new CountDownLatch(10000);

    for (int i = 0; i < 10000; i++) {
        new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    action();
                } finally {
                    countDownLatch.countDown();
                }
            }
        }).start();
    }
    countDownLatch.await();
    System.out.println(atomicI);
}

private static void action() {
    // 循环请求
    while (true){
        int count = atomicI.get();
        boolean suc = atomicI.compareAndSet(count, ++count);
        if (suc) {
            break;
        }
    }
}
```



# 源码分析

```java
// todo 
```

