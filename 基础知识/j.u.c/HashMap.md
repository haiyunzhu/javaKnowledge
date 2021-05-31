  > 平时工作中用到的最频繁的集合类就是j.u.c下的集合类，主要有List、Set、Map等。下面来详细介绍下面试频率最高的集合HashMap及ConcurrentHashMap。
- 成员变量
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
      ```
      i = (n - 1) & hash
      ```
      如果该数组所在位置为空，则初始化Node，如果有值，这时有两种情况：<br>
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
    - 区别

      jdk7和jdk8区别有两点：<br>
      1、插入链表位置不同，jdk7插入到链表头部，jdk8插入到链表尾部。<br>
      2、jdk8引入红黑树。
      
  
  - get()
  
  - resize()
    
    当我们执行put()方式，把元素插入到集合后，会判断当前容量是否超过设定的阈值变量threshold，若超过则进行扩容resize()。一般扩容为原来的2的n次方，面试的时候可能会问你为什么扩容容量为原来的2的n次方。因为扩容操作本身是很耗性能的，还会涉及到数据的迁移，正常的扩容流程是这样的，key的hash值与扩容后的容量进行与运算，得到该key对应的新数组位置，jdk7是循环遍历，采用头插法插入；jdk8会遍历原数组，对于每个数组元素，遍历执行每个key的hash值和旧容量的与运算，如果与运算结果是0，则组成新的链表放到原来的索引位置上，如果与运算是1的话，则组成新的链表放到原索引+原容量索引位置上。jdk8扩容后的索引位置为什么是这样获取的呢，这就回到最初的问题上来了，扩容容量为什么是原来的2的n次方。这里涉及到二进制与运算的规则，其他数和0进行与运算结果都是0，其他数和1进行与运算都是其本身。
    ```
      0&0=0;
      0&1=0;
      1&1=1
    ```
 我们在put的时候，会将key的hash值与数组长度-1进行与运算得到所在数组位置，2的n次方-1对应的二进制有个特点，最低位都是1，能够保证与结果都是原来的。
    我们来看下面图解
    
    其实没有绝对的对与错，换句话说，2的n次方在提升性能方面是最优的，
    
    

    
    

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
  
