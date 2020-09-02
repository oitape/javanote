### HashMap
- 类信息
    - 底层：数组+链表+红黑树。当链表的长度大于等于8时，链表转化成红黑树，当红黑树大小小于等于6时，红黑树转化为链表
    - 允许null值，不同于HashTable，是线程不安全的
    - load factor（影响因子）默认是0.75f，是均衡了时间，和空间损耗算出来的值，较高的值会减少空间开销（扩容减少，数组大小增长速度变慢），但增加了查找成本（hash冲突增加，链表长度边长）。不扩容的条件：数组容量>需要的数组大小/load factor。
    - 如果很多数据存HashMap，HashMap一开始的容量就设置足够的大小，防止过程中不断扩容，影响性能。
    - 非线程安全，我们可以自己在外部加锁，或者通过Collection#synchronizedMap来实现（原理是在方法上都加synchronized实现）

- 属性
    - 初始容量为16
    `static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; `
    
    - 最大容量
    `static final int MAXIMUM_CAPACITY = 1 << 30;`

    - 负载因子默认值
    `static final float DEFAULT_LOAD_FACTOR = 0.75f; `
    
    - 桶上的链表长度大于等于8时，链表转化成红黑树
    `static final int TREEIFY_THRESHOLD = 8;`    

    - 桶上的红黑树大小小于等于6时，红黑树转化成链表
    `static final int UNTREEIFY_THRESHOLD = 6;`
    
    - 记录迭代过程中 HashMap 结构是否发生变化，如果有变化，迭代时会 fail-fast
    `transient int modCount;`
    
    - HashMap 的实际大小，可能不准(因为当你拿到这个值的时候，可能又发生了变化)
    `transient int size;`

    - 存放数据的数组
    `transient Node<K,V>[] table;`    
    
    - 扩容的门槛，有两种情况:如果初始化时，给定数组大小的话，通过 tableSizeFor方法计算，数组大小永远接近于 2 的幂次;如果是通过 resize 方法进行扩容，大小 = 数组容量 * 0.75
    `int threshold;`
    
- 其他
   - \>>：带符号右移，正数右移高位补0，负数右移高位补1
   - \>>>：无符号右移。无论正数还是负数，高位通通补0.

  
   
   