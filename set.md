- HashSet
    - 底层实现基于 HashMap，所以迭代时不能保证按照插入顺序，或者其它顺序进行迭代;
    - add、remove、contanins、size 等方法的耗时性能，是不会随着数据量的增加而增加的， 这个主要跟 HashMap 底层的数组数据结构有关，不管数据量多大，不考虑 hash 冲突的情况下，时间复杂度都是 O (1);
    - 线程不安全的，如果需要安全请自行加锁，或者使用 Collections.synchronizedSet;
    - 迭代过程中，如果数据结构被改变，会快速失败的，会抛出ConcurrentModificationException 异常。

- TreeSet
    - 底层使用的是TreeMap
    - TreeSet 定义自己想要的 api，自己定义接口规范，让 TreeMap 去实现

- LinkedHashSet
    - 底层使用LinkedHashMap
    - 根据key 的新增顺序
    
- 集合的注意点
    - 当集合的元素是自定义类时，自定义类强制实现 equals 和 hashCode 方法，并且两个都要 实现。
    - 任意循环删除的场景下，都建议使用迭代器进行删除;
    - Arrays.asList(array),数组被修改后，会直接影响到新 List 的值；不能对新 List 进行 add、remove 等操作，否则运行时会报 UnsupportedOperationException。原因如下
    ```java
     public static <T> List<T> asList(T... a) {
        return new ArrayList<>(a);
     }
     //ArrayList并非java.util包中的类，是一个静态私有内部类，并且没有实现 add、remove 等方法
    ```
    