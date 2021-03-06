# Volatile

通过上一篇的文章我们了解到synchronized是阻塞式同步，在线程竞争激烈的情况下会升级为重量级锁。而volatile就可以说是java虚拟机提供的最轻量级的同步机制。但它同时不容易被正确理解，也至于在并发编程中很多程序员遇到线程安全的问题就会使用synchronized。Java内存模型告诉我们，各个线程会将共享变量从主内存中拷贝到工作内存，然后执行引擎会基于工作内存中的数据进行操作处理。线程在工作内存进行操作后何时会写到主内存中？这个时机对普通变量是没有规定的，而针对volatile修饰的变量给java虚拟机特殊的约定，线程对volatile变量的修改会立刻被其他线程所感知，即不会出现数据脏读的现象，从而保证数据的“可见性”。

现在我们有了一个大概的印象就是：**被volatile修饰的变量能够保证每个线程能够获取该变量的最新值，从而避免出现数据脏读的现象。**

## 用法

```java
int volatile count = 0
```

只需要给变量定一个`volatile`修饰符

## 场景

- 运行结果并不依赖变量的当前值，或者能够确保只有单一的线程修改变量的值。

- 变量不需要与其他的状态变量共同参与不变约束。

## 内存屏障

内存屏障也称为内存栅栏或栅栏指令，是一种屏障指令，它使CPU或编译器对屏障指令之前和之后发出的内存操作执行一个排序约束。 这通常意味着在屏障之前发布的操作被保证在屏障之后发布的操作之前执行。

| 类型           | 简介                                                         |                      |
| -------------- | ------------------------------------------------------------ | -------------------- |
| LoadLoad屏障   | Load1 和 Load2 代表两条读取指令。在Load2要读取的数据被访问前，保证Load1要读取的数据被读取完毕。 | 连续两个读不能换顺序 |
| StoreStore屏障 | Store1 和 Store2代表两条写入指令。在Store2写入执行前，保证Store1的写入操作对其它处理器可见 | 连续两个写不能换顺序 |
| LoadStore屏障  | 在Store2被写入前，保证Load1要读取的数据被读取完毕。          | 一读一写不能换顺序   |
| StoreLoad屏障  | 在Load2读取操作执行前，保证Store1的写入对所有处理器可见。StoreLoad屏障的开销是四种屏障中最大的。 | 一写一读不能换顺序   |

- volatile读之前，会添加LoadLoad内存屏障。
- volatile读之后，会添加LoadStore内存屏障。
- volatile写之前，会添加StoreStore内存屏障。
- volatile写之后，会添加StoreLoad型内存屏障。

# 区别

|                  | synchronized | volatile |
| :--------------: | :----------: | :------: |
|    保值原子性    |      √       |    ×     |
|    保持可见性    |      √       |    √     |
| 针对于可见性性能 |     较差     |   较强   |
|       本质       |      🔒       | 内存屏障 |