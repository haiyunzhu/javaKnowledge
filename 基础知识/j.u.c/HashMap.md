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


    计算key的hash值，然后和数组长度进行**与**运算，得到所在的数组位置，接着判断hash值和key是否都相等，若相等，则返回该值对应的value，若不相等，则遍历链表重复之前的操作。这里要讲下，hash和key进行比较时用到的==和equals。<br>
    这里简单讲下"=="和equals。equals是父类Object的方法，里面就是用==来实现的。实际涉及到需要比较的子类，一般用是否是同一个成员变量来进行判断，因此会进行重写equals()方法，**重写equals()方法需要重写hashcode()** ，为什么呢？因为重写了equals()方法后，我们认为两个相同的对象不再是地址相同的了，而是满足equals()方法，而相同对象hashcode()一定是相同的，如果不重写hashcode，可能会出现我们认定是相同的对象的hashcode值不相等，这违背了“两个相同的对象，hashcode一定相等”。

    <br>两者区别是equals用来比较是否是同一个对象，同一个对象”==“一定相等，而”==”用来比较地址是否相同，地址相同的可能不是同一个对象。重写equals/hashcode? 面试基础知识时，可能会问到equals和==区别。
  
  ```
      first.hash == hash && // always check first node
      ((k = first.key) == key || (key != null && key.equals(k)))
  ```
  
  - resize()
    
    当我们执行put()方式，把元素插入到集合后，会判断当前容量是否超过设定的阈值变量threshold，若超过则进行扩容resize()。一般扩容为原来的2的n次方，面试的时候可能会问你为什么扩容容量为原来的2的n次方。因为扩容操作本身是很耗性能的，还会涉及到数据的迁移，正常的扩容流程是这样的，key的hash值与扩容后的容量进行与运算，得到该key对应的新数组位置，jdk7是循环遍历，采用头插法插入；jdk8会遍历原数组，对于每个数组元素，遍历执行每个key的hash值和旧容量的与运算，如果与运算结果是0，则组成新的链表放到原来的索引位置上，如果与运算是1的话，则组成新的链表放到原索引+原容量索引位置上。jdk8扩容后的索引位置为什么是这样获取的呢，这就回到最初的问题上来了，扩容容量为什么是原来的2的n次方。这里涉及到二进制与运算的规则，其他数和0进行与运算结果都是0，其他数和1进行与运算都是其本身。
    ```
      0&0=0;
      0&1=0;
      1&1=1
    ```
 我们在put的时候，会将key的hash值与数组长度-1进行与运算得到所在数组位置，2^n-1对应的二进制有个特点，低位都是1，key的hash值与其进行与运算得到的结果就是原来的值，扩容为原来的2倍后的二进制数相比较原来，高位变成了1，与运算之后高位不是0就是1，也就是说，新的数组位置要么在原地不动，要么是在原来的2n，这样有个好处<br>
    1.原来在一个索引位置的拆成两处，减少hash碰撞；<br>
    2.不需要移动所有的数，减少迁移成本；<br>
如果不采取扩容为原来的2倍，也是可以的，只是hash碰撞会相对比较高，会迁移更多数据。


    
    
    
    
    
    
  
    

    
    

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
  
