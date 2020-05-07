## 基本使用
```java
Vector<String> vector = new Vector<String>();
vector.add("杨花落尽子规啼");
vector.add("烟花三月下扬州");
vector.add("遥知兄弟登高处");
vector.add("也无风雨也无晴");
System.out.println(vector);

// 控制台输出结果
[杨花落尽子规啼, 烟花三月下扬州, 遥知兄弟登高处, 也无风雨也无晴]
```

## 差异

在源码或文档中都可以看到  `ArrayList`  和 `Vector` 都继承于 `AbstractList` 抽象类

### 源码比较

随便找两个方法比较一下

- `isEmpty`

  ```java
  // ArrayList
  public boolean isEmpty() {
      return size == 0;
  }
  
  // Vector
  public synchronized boolean isEmpty() {
      return elementCount == 0;
  }
  ```


- `size`

  ```java
    // ArrayList
    public int size() {
        return size;
    }
  
    // Vector
    public synchronized int size() {
        return elementCount;
    }
  ```


除了变量名的的不同最大的不一样的就是多了`synchronized`关键字

如果存在`synchronized`关键字代表这个**方法加锁**,相当于不管哪一个线程（例如线程A），运行到这个方法时,都要**检查**有没有其它线程B（或者C、 D等）正在用这个方法(或者该类的其他同步方法)，有的话要等正在使用`synchronized`方法的线程B（或者C 、D）运行完这个方法后再运行此线程A,没有的话,锁定调用者,然后直接运行。

其实就是一个![img](../../image/252A4070.png)

关于线程安全在后续多线程相关的笔记中细讲。

### 扩容
- ArrayList

  ```java
  private void grow(int minCapacity) {
      // 原容量
      int oldCapacity = elementData.length;
      // 原容量 + 原容量右移一位
      // newCapacity = newCapacity + newCapacity/2 
      // 也就是扩容1.5倍
      int newCapacity = oldCapacity + (oldCapacity >> 1);
      if (newCapacity - minCapacity < 0)
          newCapacity = minCapacity;
      if (newCapacity - MAX_ARRAY_SIZE > 0)
          newCapacity = hugeCapacity(minCapacity);
      elementData = Arrays.copyOf(elementData, newCapacity);
  }
  
  ```



- Vector

  ```java
  // 扩容时增加的容量
  protected int capacityIncrement;  
  
  // 构造方法可传入两个参数 初始容量 和 扩容容量
  public Vector(int initialCapacity, int capacityIncrement) {
      super();
      if (initialCapacity < 0)
          throw new IllegalArgumentException("Illegal Capacity: "+
                                             initialCapacity);
      this.elementData = new Object[initialCapacity];
      this.capacityIncrement = capacityIncrement;
  }
  
  private void grow(int minCapacity) {
      // 原容量
      int oldCapacity = elementData.length; 
      // 扩容后的新容量 = 原长度 + 扩容容量
      // 如果在构造方法中定义了则加上定义的容量
      // 没有定义则加上原长度也就是翻了一倍
      int newCapacity = oldCapacity + ((capacityIncrement > 0) ?capacityIncrement : oldCapacity);
      if (newCapacity - minCapacity < 0)
          newCapacity = minCapacity;
      if (newCapacity - MAX_ARRAY_SIZE > 0)
          newCapacity = hugeCapacity(minCapacity);
      elementData = Arrays.copyOf(elementData, newCapacity);
  }
  ```




## 总结

|          |  ArrayList   |    Vector    |
| :------: | :----------: | :----------: |
| 数据结构 |     数组     |     数组     |
|   父类   | AbstractList | AbstractList |
|   扩容   |    1.5倍     | 默认扩容2倍  |
|  多线程  |  线程不安全  |   线程安全   |
| 执行效率 |     较高     |     较低     |


