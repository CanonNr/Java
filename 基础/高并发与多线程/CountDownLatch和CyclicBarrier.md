## CountDownLatch

### 简述

允许一个或多个线程等待直到在其他线程中执行的一组操作完成的同步辅助。

### 使用场景

假设如下场景：创建N多个线程去执行任务如何在执行完第一时间知道。

之前都会在线程开始之后写一个`Thread.sleep()` 但是现在显然是不可以的。

### 代码实例

```java
public static int  count = 0;
public static CountDownLatch downLatch;
static void action() throws InterruptedException {
    // 模拟业务代码 睡一秒
    TimeUnit.SECONDS.sleep(1);
    synchronized(Demo1.class){
        // 喜+1
        count++;
    }
    System.out.println(count);
}

public static void main(String[] args) throws InterruptedException {
    long start = System.currentTimeMillis();
    int ThreadLength = 10000;
    // 实例化一个 CountDownLatch 并定义长度为 10000
    downLatch = new CountDownLatch(ThreadLength);

    for (int i = 0; i < ThreadLength; i++) {
        // 循环创建线程
        new Thread(new Runnable() {
            public  void run() {
                try {
                    action();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }finally {
                    // count -1
                    downLatch.countDown();
                }
            }
        }).start();
    }
    // 拦截 如果downLatch不为0则不会往下执行
    downLatch.await();
    long end = System.currentTimeMillis();
    System.out.println(end-start+"毫秒  "+"结果:"+count);
}
```

### 方法摘要

| 返回类型 | 方法简介                                                     |
| :------- | :----------------------------------------------------------- |
| void     | `await()`导致当前线程等到锁存器计数到零                      |
| bolean   | `await(long timeout, TimeUnit unit)`使当前线程等待直到锁存器计数到零为止，除非线程为 interrupted或指定的等待时间过去。 |
| void     | `countDown()`减少锁存器的计数，如果计数达到零，释放所有等待的线程。 |
| long     | `getCount()`返回当前计数。                                   |
| String   | `toString()` 返回一个标识此锁存器的字符串及其状态。          |





## CyclicBarrier

### 简述

约定一个值，当到打这个值的时候才会去执行

### 场景

创建二十个线程，每凑够三个线程就是输出“一桌斗地主“

### 代码示例

```java
static CyclicBarrier barrier;
public static void main(String[] args) {
    barrier = new CyclicBarrier(3, new Runnable() {
        public void run() {
            System.out.println("一桌斗地主");
        }
    });
    for (int i = 0; i < 10; i++) {
        new Thread(new Runnable() {
            public void run() {
                try {
                    barrier.await();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } catch (BrokenBarrierException e) {
                    e.printStackTrace();
                }
            }
        }).start();
    }
}
```

### 方法摘要

| 返回类型  | 方法简介                                                     |
| :-------: | :----------------------------------------------------------: |
| int   | `await()`等待所有 parties已经在这个障碍上调用了 `await` 。 |
| int     | `await(long timeout, TimeUnit unit)`等待所有 parties已经在此屏障上调用 `await` ，或指定的等待时间过去。 |
| int    | `getNumberWaiting()`返回目前正在等待障碍的各方的数量。       |
| int   | `getParties()`返回旅行这个障碍所需的聚会数量。               |
|boolean| `isBroken()`查询这个障碍是否处于破碎状态。                   |
| void    | `reset()`将屏障重置为初始状态。                              |





## 二者区别

|                        CountDownLatch                        |                        CyclicBarrier                         |
| :----------------------------------------------------------: | :----------------------------------------------------------: |
|                          减计数方式                          |                          加计数方式                          |
|                 计算为0时释放所有等待的线程                  |               计数达到指定值时释放所有等待线程               |
|                     计数为0时，无法重置                      |             计数达到指定值时，计数置为0重新开始              |
| 调用countDown()方法计数减一，调用await()方法只进行阻塞，对计数没任何影响 | 调用await()方法计数加1，若加1后的值不等于构造方法的值，则线程阻塞 |
|                         不可重复利用                         |                          可重复利用                          |