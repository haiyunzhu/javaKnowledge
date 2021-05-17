  > 平时工作中用到的最频繁的集合类就是j.u.c下的集合类，主要有List、Set、Map等。下面来详细介绍下面试频率最高的集合HashMap及ConcurrentHashMap。
- 数据结构

hashMap底层数据结构由数组+链表构成，jdk8后又加了红黑树，红黑树主要是为了解决链表长度过长影响查询性能，
hashMap是key-value形式的存储，key经过hash后与数组初始长度进行取模运算，得到的值就是在数组中的位置。当出现相同的key即出现hash冲突时，在对应的数组位置上会转换为链表，将冲突的值存放到链表的尾部。对应到源码里分别是元素为Node<K,V>的table和Node链表

<img width="700" height="400" src="https://user-images.githubusercontent.com/16397120/118498284-730a6c00-b758-11eb-96bc-45b4621285c2.png"/>

- jdk7.hashMap
  - put过程
  将key进行hash运算后得到的hash值，与数据长度进行与运算，如果值为空，则初始化Node，如果有值，则进行hash冲突解决。
  ```
  (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16)
  ```
  
  
  - get过程
- jdk7.concurrentHashMap
- jdk8.hashMap
- jdk8.concurrentHashMap
- 五、面试点
  - hashMap为什么不是线程安全的？
  - hashMap，jdk8相对于jdk7，做了哪些改动？
  - concurrentHashMap为什么是线程安全的？
  - cocurrentHashMap，jdk8相对于jdk7，做了哪些改动？
  - hashMap扩容容量为什么是2的n次方？
  
