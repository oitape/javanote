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
        - 默认优先使用log4j
        - 然后jul log
        
        - 可以使用sl4j，通过绑定器进行动态绑定不同的log实现，参考sl4j官网

- 5.x

```java
public static Log createLog(String name) {
	switch (logApi) { // logApi = JUL
		case LOG4J:
			return Log4jAdapter.createLog(name);
		case SLF4J_LAL:
			return Slf4jAdapter.createLocationAwareLog(name);
		case SLF4J:
			return Slf4jAdapter.createLog(name);
		default:
			return JavaUtilAdapter.createLog(name);
	}
}

```
        