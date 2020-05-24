#  简述

`wait`的作用是让当前线程进入等待状态，同时，wait()也会让当前线程释放它所持有的锁

`notify`和`notifyAll`的作用，则是唤醒当前对象上的等待线程

# 使用

```java
// 定义一个锁
final Object lock = new Object();

// 线程1中
synchronized (lock){
    lock.wait(); 
}

// 线程2中
synchronized (lock){
    lock.notify();
}
```

如上中当线程1中进行等待，线程2会对1进行唤醒

# 实例

```java
public static volatile Collection<Integer> collection = new Collection<Integer>();
public static void main(String[] args) throws InterruptedException {
    final Object lock = new Object();
    new Thread(new Runnable() {
        @Override
        public void run() {
            synchronized (lock){
                int size = collection.size();
                if (size != 5){
                    try {
                        System.out.println("我阻塞了");
                        lock.wait();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    System.out.println("end"+size);
                    lock.notify();
                }
            }
        }
    }).start();

    Thread.sleep(1000);

    new Thread(new Runnable() {
        @Override
        public void run() {
            synchronized (lock){
                for (int i = 1; i <= 10; i++) {
                    collection.add(i);
                    if (i == 5){
                        lock.notify();

                        try {
                            lock.wait();
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                    }
                    System.out.println("添加了一个元素"+i);
                }
            }
        }
    }).start();
}
```

定义了一个`collection`的容器，有`size`和`add`方法。

要实现的是：创建两个线程`T1`和`T2`，`T1`对容器进行监控当容器添加到第5个元素的时候输出一句话，`T2`则是通过`for`循环持续`add`元素到容器。

思路：

- 对`T1`执行`wait`进行阻塞操作，此时`T1`让出锁
- 当`T2`添加到第5个参数时唤醒`T1`并阻塞自己
- `T1`重新开始执行

# 与 Sleep 对比

|          |     sleep     |       wait       |
| :------: | :-----------: | :--------------: |
|   作用   | 暂停当前线程  |   暂停当前进程   |
|  释放锁  |   不释放锁    |      释放锁      |
| 使用场景 |   任何场景    | 同步代码块中使用 |
|   实现   | Thread 方法下 |   Object 方法    |
|   唤醒   |  时间到唤醒   |     随时唤醒     |

