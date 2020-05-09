## HashSet

HashSet 实现了 `Set `接口，它不允许集合中出现重复元素。当我们提到HashSet时，第一件事就是在将对象存储在

HashSet之前，要确保重写 `hashCode（）`方法和 `equals（）`方法，这样才能比较对象的值是否相等，确保集合中没有

储存相同的对象。如果不重写上述两个方法，那么将使用下面方法默认实现：

`public boolean add(Object obj)`方法用在 Set 添加元素时，如果元素值重复时返回 "false"，如果添加成功则返回"true"



## HashMap：

HashMap实现了Map接口，Map接口对键值对进行映射。Map中不允许出现重复的键（Key）。Map接口有两个基本的实现 `TreeMap` 和HashMap`。TreeMap保存了对象的排列次序，而HashMap不能。HashMap可以有空的键值对（Key（null）-Value（null））

HashMap是非线程安全的（非Synchronize），要想实现线程安全，那么需要调用collections类的静态方法synchronizeMap（）实现。

`public Object put(Object Key,Object value)`方法用来将元素添加到map中。

## 总结

|                  *HashMap*                  |                          *HashSet*                           |
| :-----------------------------------------: | :----------------------------------------------------------: |
|            HashMap实现了Map接口             |                     HashSet实现了Set接口                     |
|              HashMap储存键值对              |                     HashSet仅仅存储对象                      |
|        使用put()方法将元素放入map中         |                 使用add()方法将元素放入set中                 |
|     HashMap中使用键对象来计算hashcode值     | HashSet使用成员对象来计算hashcode值，对于两个对象来说hashcode可能相同，所以equals()方法用来判断对象的相等性，如果两个对象不同的话，那么返回false |
| HashMap比较快，因为是使用唯一的键来获取对象 |                  HashSet较HashMap来说比较慢                  |

