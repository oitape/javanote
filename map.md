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

- 链表的新增
    - 新增就是把节点追加到链表尾部，和LinkedList的追加一样
    - 当链表长度大于等于8时，此时的链表就会转化为红黑树，转化的方法时treeifyBin，此方法有一个判断，当链表长度大于等于8并且数组大小大于64时，才会转成红黑树，当数组大小小于64时，只会触发扩容，不会转化为红黑树，
- 红黑树新增节点过程
    - 判断节点在红黑树上是不是已经存在，
        - 节点没有实现Comparable接口，使用equals判断
    - 新增节点如果在红黑树上直接返回，若不在，判断新增节点是在当前节点的左边还是右边，左边值小，右边值大
    - 自旋递增上面两步，直到当前节点的左边或者右边的节点为空时，停止自旋，当前节点即为我们新增节点的父节点
    - 把新增节点放到当前节点的左边或右边为空的地方。并与当前节点建立父子节点关系
    - 进行着色和旋转 结束
        
- 红黑树的5个原则
    - 节点是红色或黑色
    - 根是黑色
    - 所有叶子都是黑色
    - 从任一节点到其每个叶子的所有简单路径都包含相同数目的黑色节点
    - 每个红色节点的两个子节点都是黑色也就是从每个叶子到根的所有路径上不能有两个连续的红色节点
    
- 红黑树查找
    - 从根节点递归查找;
    - 根据 hashcode，比较查找节点，左边节点，右边节点之间的大小，根本红黑树左小右大的
特性进行判断;
    - 判断查找节点在第 2 步有无定位节点位置，有的话返回，没有的话重复 2，3 两步;
    - 一直自旋到定位到节点位置为止。

- 各个map区别
    - HashMap 是无序的；
    - TreeMap 可以按照 key 进行排序；
    - LinkedHashMap可以维护插入的顺序；LinkedHashMap 只提供了单向访问，即按照插入的顺序从头到尾进行访问。LinkedHashMap实现了LRU算法，经常访问的元素会被追加到队尾，不经常访问的数据自然就靠近队头，我们可以自定义删除策略，对队头元素进行删除，默认不删除。


- 面试问题
    - HashMap底层数据结构？<br>
    HashMap 底层是数组 + 链表 + 红黑树的数据结构，数组的主要作用是方便快速查找，时 间复杂度是 O(1)，默认大小是 16，数组的下标索引是通过 key 的 hashcode 计算出来的，数 组元素叫做 Node，当多个 key 的 hashcode 一致，但 key 值不同时，单个 Node 就会转化 成链表，链表的查询复杂度是 O(n)，当链表的长度大于等于 8 并且数组的大小超过 64 时，链 表就会转化成红黑树，红黑树的查询复杂度是 O(log(n))，简单来说，最坏的查询次数相当于红 黑树的最大深度
    - HashMap、TreeMap、LinkedHashMap三者异同? <br>
        - 相同点
            - 三者特定情况下都会使用红黑树
            - 底层hash算法相同
            - 迭代过程中，Map被改动都会抛ConcurrentModificationException异常
        - 不同点
            - HashMap数据结构以数组为主，查询较快；TreeMap数据结构以红黑树为主，利用红黑树左小右大实现key的排序；LinkedHashMap继承HashMap，增加了插入顺序访问和LRU策略        
            - 三种 map 的底层数据结构的不同，导致上层包装的api略有差别。
    - Map的hash算法
    ```java
    static final int hash(Object key) {
     int h;
     return (key == null) ? 0 : 
     (h = key.hashCode()) ^ (h >>> 16); 
    }
    ```
            
            
            
            
            
            
            
   
   