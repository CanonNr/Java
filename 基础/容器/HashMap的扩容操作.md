## 什么是扩容

顾名思义就是扩大容量，在实例化时`HashMap`的容量为 16，但是如果数据量越来越大肯定是需要变化的，所以本文主要讲了：什么是时候扩容、扩多少、怎么扩。

## 核心字段

在`Hashmap`中有几个字段还是必须要了解的

```java
int DEFAULT_INITIAL_CAPACITY = 1 << 4;   // 默认的容量
int MAXIMUM_CAPACITY = 1 << 30;			// 最大容量
float DEFAULT_LOAD_FACTOR = 0.75f;		// 扩容因子

int threshold;             // 容量阈值 
final float loadFactor;    // 负载因子
int modCount;  			  // 一个计数器
int size;  				 // 当前容量
```



## 什么时候扩容

### 首次扩容

需要说一下，虽然默认容量为 16 好像已经人尽皆知，但是在声明一个`HashMap`的时候容量其实还是 0 ，而且最核心的数组并没有声明，只有在`put` 第一个元素的时候容量才会变成16。

在首次`put`操作时会做一个判断：如果为空则进行扩容操作
```java
Node<K,V>[] tab; Node<K,V> p; int n, i;
// 因为是新声明的 table 还是 空
if ((tab = table) == null || (n = tab.length) == 0){
    // 执行扩容操作
	n = (tab = resize()).length;  
}         
```

```java
 final Node<K,V>[] resize() {
     Node<K,V>[] oldTab = table;
     int oldCap = (oldTab == null) ? 0 : oldTab.length;
     int oldThr = threshold;
     int newCap, newThr = 0;
     // 首先判断 容量是否大于 0
     if (oldCap > 0) {
         // 是否大于了  MAXIMUM_CAPACITY 的 最大容量 也就是前面定义的 1 << 30
         if (oldCap >= MAXIMUM_CAPACITY) {
             threshold = Integer.MAX_VALUE;
             return oldTab;
         }
         // 然后判断 扩容后的容量是否大于最大容量 且 旧的容量大于等于默认的容量
         else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                  oldCap >= DEFAULT_INITIAL_CAPACITY)
             newThr = oldThr << 1; // double threshold
     }
     else if (oldThr > 0) 
         newCap = oldThr;
     else {
         // 新的容量 为 默认的值16 
         newCap = DEFAULT_INITIAL_CAPACITY;
         // 新的阈值为 默认的容量*扩容因子  也就是16*0.75 并强转为int 得到 12
         newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
     }
     if (newThr == 0) {
         float ft = (float)newCap * loadFactor;
         newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                   (int)ft : Integer.MAX_VALUE);
     }
     // 将前面计算好的 阈值传入 threshold 中
     threshold = newThr;
     
     // 最关键的一个点来了
     // 声明了一个 newTab 数组 长度为 newCap 也就是16
     Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
     // 判断一下 旧的容器是否为空
     // 此步主要是用来做数据迁移的 
     // 因为我们此步是首次 所以 oldTab 自然是空 ,后面再说数据迁移
     if (oldTab != null) {
         for (int j = 0; j < oldCap; ++j) {
             .....
         }
     }
  	// 赋值
     table = newTab;
     // 返回
     return newTab;
```





### 超过阈值扩容



前面我们说到了`threshold`这个字段，再把公式搬出来一下。

```java
// 在首次扩容的时候 定义了 threshold 值为 12
threshold = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
```

```java
// putVal 方法
// 一个计数器 只增不减
++modCount;
// 当 size 大于 threshold时则进行扩容操作
// 而 size 在每次 put 时会 ++ 操作，remove时 -- 操作
// 也就是 当 put 了 13次（ 大于 大于 大于 不是大于等于 所以要13 ）时会进行扩容操作
if (++size > threshold){
        resize();
}
```

```java
final Node<K,V>[] resize() {
    Node<K,V>[] oldTab = table;
    int oldCap = (oldTab == null) ? 0 : oldTab.length;
    int oldThr = threshold;
    int newCap, newThr = 0;
    // 同上
    if (oldCap > 0) {
        // 并没有超过最大容量
        if (oldCap >= MAXIMUM_CAPACITY) {
            threshold = Integer.MAX_VALUE;
            return oldTab;
        }
        // 旧的容量左移一位后容量没有超过最大容量也就是32 且 旧容量大于等于默认16 
        else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                 oldCap >= DEFAULT_INITIAL_CAPACITY)
            // 旧的阈值左移一位 也就是 12 << 1 ，得到了新的阈值也就是 24 
            newThr = oldThr << 1; 
    }
    // 赋值
    threshold = newThr;
    // 声明了一个 newTab 数组 长度为 newCap 也就 32
    Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
    table = newTab;
    // 因为已经存在旧数据 所以肯定不为空 进行数据转移
    // 所以此时就需要将 oldTab 数据转移到 newTab 中
    if (oldTab != null) {
        // 根据旧的容量进行循环
        for (int j = 0; j < oldCap; ++j) {
            Node<K,V> e;
            // 如果不为空则进行操作 并将 oldTab[j] 赋值给 e
            if ((e = oldTab[j]) != null) {
                // 此时已经有了 e ，所以就不需要 oldTab[j]了
                oldTab[j] = null;
                // 提醒一下 HashMap 的三种 存储结构 数组 链表 红黑树
                // 如果没有下一个节点 
                if (e.next == null)
                    // 直接将e 赋值给新的容器
                    // 通过之前说的 位运算 得出数组的下标
                    newTab[e.hash & (newCap - 1)] = e;
                // 如果是 红黑树
                else if (e instanceof TreeNode)
                    // todo
                    ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                else { 
                    // 链表结构
                    Node<K,V> loHead = null, loTail = null;
                    Node<K,V> hiHead = null, hiTail = null;
                    Node<K,V> next;
                    do {
                        next = e.next;
                        if ((e.hash & oldCap) == 0) {
                            if (loTail == null)
                                loHead = e;
                            else
                                loTail.next = e;
                            loTail = e;
                        }
                        else {
                            if (hiTail == null)
                                hiHead = e;
                            else
                                hiTail.next = e;
                            hiTail = e;
                        }
                    } while ((e = next) != null);
                    if (loTail != null) {
                        loTail.next = null;
                        newTab[j] = loHead;
                    }
                    if (hiTail != null) {
                        hiTail.next = null;
                        newTab[j + oldCap] = hiHead;
                    }
                }
            }
        }
    }
    return newTab;
```

