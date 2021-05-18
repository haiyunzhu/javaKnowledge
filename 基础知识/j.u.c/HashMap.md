  > 平时工作中用到的最频繁的集合类就是j.u.c下的集合类，主要有List、Set、Map等。下面来详细介绍下面试频率最高的集合HashMap及ConcurrentHashMap。
- 数据结构

hashMap底层数据结构由数组+链表构成，jdk8后又加了红黑树，红黑树主要是为了解决链表长度过长影响查询性能，
hashMap是key-value形式的存储，key经过hash后与数组初始长度进行取模运算，得到的值就是在数组中的位置。当出现相同的key即出现hash冲突时，在对应的数组位置上会转换为链表，将冲突的值存放到链表的尾部。对应到源码里分别是元素为Node<K,V>的table和Node链表

<img width="700" height="400" src="https://user-images.githubusercontent.com/16397120/118498284-730a6c00-b758-11eb-96bc-45b4621285c2.png"/>

- 方法
  - put()
  
     将key进行hash运算后得到的hash值，与数组长度进行与运算，得到的结果即是对应的数组位置。
      ```
      (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16)
      ```
      如果该数组所在位置为空，则初始化Node，如果有值，这时有两种情况：
      1）是同一个key，若是同一个key，则进行覆盖；判断是否是同一个key，其实是判断是否是同一个对象的问题，具体代码如下：
      ```
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
            e = p;
      ```
      2）不是同一个key，需要进行hash冲突解决，将新的k-v插入到该数组索引所在的链表中。至于插到哪个位置，jdk7和jdk8是不一样的。
      put完之后，将size+1，若++size超过了阈值，则进行扩容。 
  
    - jdk7及之前
      
      把新的键值对放到链表的头部
      ```
        void createEntry(int hash, K key, V value, int bucketIndex) {
          Entry<K,V> e = table[bucketIndex];
          table[bucketIndex] = new Entry<>(hash, key, value, e);
          size++;
        }
      ```
    
    - jdk8
      
      jdk8引入了红黑树，当链表长度不超过8，则插入链表尾部；若超过8，则插入到红黑树中。
      ```
        p.next = newNode(hash, key, value, null);
      ```
      
  
  - get()
  - resize()
    
    

get过程
- jdk7.hashMap
  - put过程
  
 
  - get过程
  - 扩容过程
- jdk7.concurrentHashMap
- jdk8.hashMap
- jdk8.concurrentHashMap
- 五、面试点
  - hashMap为什么不是线程安全的？
  - hashMap，jdk8相对于jdk7，做了哪些改动？
  - concurrentHashMap为什么是线程安全的？
  - cocurrentHashMap，jdk8相对于jdk7，做了哪些改动？
  - hashMap扩容容量为什么是2的n次方？
  
