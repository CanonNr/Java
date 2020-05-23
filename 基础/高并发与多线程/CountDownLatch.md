## 简述

假设如下场景：创建N多个线程去执行任务如何在执行完第一时间知道。

之前都会在线程开始之后写一个`Thread.sleep()` 但是现在显然是不可以的。

## 代码

```java

import java.util.concurrent.BrokenBarrierException;
import java.util.concurrent.CountDownLatch;
import java.util.concurrent.CyclicBarrier;
import static java.lang.Thread.sleep;

public class Demo1 {

    public static volatile int  count = 0;
    public static CountDownLatch downLatch;
     static synchronized void action() throws InterruptedException {
        sleep(5);
        for (int i = 0; i < 1000; i++){
            count++;
            System.out.println(count);
        }
    }

    public static void main(String[] args) throws InterruptedException {
        long start = System.currentTimeMillis();
        int ThreadLength = 1000;
        downLatch = new CountDownLatch(ThreadLength);

        for (int i = 0; i < ThreadLength; i++) {
            new Thread(new Runnable() {
                public  void run() {
                    try {
                        action();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }finally {
                        downLatch.countDown();
                    }
                }
            }).start();
        }
        downLatch.await();
        long end = System.currentTimeMillis();
        System.out.println(end-start+"毫秒,"+"结果:"+count);
    }
}

```



