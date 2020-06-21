* Long Integer内部实现了一种缓存机制，缓存匆-128到127诶的所有Long值，如果是这个范围的值，则不会初始化直接从缓存中拿

```java
    public static Long valueOf(long l) {
        final int offset = 128;
        if (l >= -128 && l <= 127) { // will cache
            return LongCache.cache[(int)l + offset];
        }
        return new Long(l);
    }

    private static class LongCache {
        private LongCache(){}

        static final Long cache[] = new Long[-(-128) + 127 + 1];

        static {
            for(int i = 0; i < cache.length; i++)
                cache[i] = new Long(i - 128);
        }
    }


    Long a = new Long(1);
    Long b = Long.valueOf(1);
    Long c = Long.valueOf("1");
    Long d = Long.parseLong("1"); //自动装箱是用的valueof,所以下面查看hash  bcd都是相同的
    Long e = new Long(1);
    System.out.println(System.identityHashCode(a)); //611437735
    System.out.println(System.identityHashCode(b)); //100555887
    System.out.println(System.identityHashCode(c)); //100555887
    System.out.println(System.identityHashCode(d)); //100555887 
    System.out.println(System.identityHashCode(e)); //1769597131
```



