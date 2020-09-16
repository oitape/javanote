- spring4.x和spring5.x不一样

- 4.x,按照以下顺序查找实例化
```java
    private static final String[] classesToDiscover = {
            "org.apache.commons.logging.impl.Log4JLogger",
            "org.apache.commons.logging.impl.Jdk14Logger",
            "org.apache.commons.logging.impl.Jdk13LumberjackLogger",
            "org.apache.commons.logging.impl.SimpleLog"
    }
```